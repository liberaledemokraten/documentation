---
title: 'Content Delivery Network'
---

It's not hard to notice that Nextcloud requires alot of not user-related static resources such as JavaScript and CSS files. That creates a non-negligible load on the server if multiple users are using the LD Cloud. To both lower the load on our server as well as to deliver static content faster, we are using Cloudflare's CDN Network to CDNize such content.

## Set Up and Run the Cloud Storage

### Get Static Files
To deliver such static contents through the CDN, we first have to determine which files should be served through the CDN. To that end, we will copy such files to a seperate directory using the following script

_getfilestosync.sh_
```bash
#!/bin/bash

filetypes=( "css" "js" "woff" "woff2" "ttf" "png" "svg" "ico" )

root={{path_to_domain_root}}
wwwroot="$root/public_html"
appdir="$wwwroot/apps"
coredir="$wwwroot/core"
cpdir="$root/sync/files"

for filetype in ${filetypes[*]}
do
    cd $appdir
    cp -r -u --parent `find -name \*.$filetype` $cpdir/apps 2> /dev/null
    
    cd $coredir
    cp -r -u --parent `find -name \*.$filetype` $cpdir/core 2> /dev/null
done
```

### Set Up a Bucket
To reduce the load on our server, we'll have to save those now copied files to a Cloud bucket like Amazon S3, Backblaze B2 or similar. We are using Backblaze's EU-Central (Amsterdam, NL) B2 servers to deal with that due to multiple reasons, among them:

* Backblaze is more cost-efficient
* Backblaze has a better reputation than Amazon
* Backblaze has a Bandwidth Alliance with Cloudflare

First, go through the following steps:

1. Register for Backblaze B2
2. Create a new bucket
3. Add a new Application Key with read and write permissions
4. Install the [B2 CLI client](https://www.backblaze.com/b2/docs/quick_command_line.html)
5. Configure the client appropriately

Alternatively, follow the analogous steps for whatever cloud storage you have chosen to use.

NOTE: We are using a private bucket, meaning the content can be read only if authorized to do so. This makes the configuration of the CDN a little more complicated, but access control and the use of bandwidth can be managed easier.

### Synchronize the Files
Now that we have both retrieved the files to synchronize and also set up our cloud storage, we can upload our files. The keyword here is to synchronize instead of just uploading such that we can avoid uploading if the file was not modified and exists in our cloud storage already. To that end, we run the client we had set up before, or the following script:

_syncfiles.sh_
```sh
#! bin/sh
syncdir={{path_to_domain_root}}"/sync"

cd $syncdir
bash getfilestosync.sh # get files to synchronize
cd $syncdir

python /home/{{username}}/.local/lib/python2.7/site-packages/b2/console_tool.py sync files/core/ b2://{{bucket_name}}/core/ > /dev/null
python /home/{{username}}/.local/lib/python2.7/site-packages/b2/console_tool.py sync files/apps/ b2://{{bucket_name}}/apps/ > /dev/null
```

## Set Up Your CDN
We are using Cloudflare's CDN as it is a zero-cost solution with alot of benefits including the aforementioned Bandwidth Alliance and Workers. You may go with a different provider if you wish so, but keep in mind that the following steps will be tailored down to the use of Cloudflare.

First, go through the following steps:

1. Create a Cloudflare Account
2. Add the domain that you want to use Cloudflare with and activate Cloudflare DNS
3. Add a CNAME record for your CDN (sub)domain. It should point to the domain that you see when you browse through your files in the bucket and see in the "friendly URL" section when you click on a file (e.g. f003.backblazeb2.com)
4. Create a Cloudflare Worker and get your APIs
5. Add "{{cdn_domain}}/file/{{bucket_name}}" to the Worker's route

You may want to configure the caching settings in Cloudflare while at it, and also activate the option to minify scripts. Also, you may want to redirect requests to {{cdn_domain}}/core/ and {{cdn_domain}}/apps/ to {{cdn_domain}}/file/{{bucket_name}}/core/ and {{cdn_domain}}/file/{{bucket_name}}/apps/. To do so, utilize either page rules or another Worker.

NOTE: If your B2 bucket is public, then skip using a worker. You only need it if you are using a private bucket such as we do.

### Setting Up the Worker
If your B2 bucket is public, then skip this part. But otherwise,go through the following steps:

1. Create a new Application Key explicitly for the bucket in question. You don't need to give any write permissions. Copy the Key and the Key ID.
2. Copy the bucket ID
3. Copy your Cloudflare Account ID, your API Key to edit workers and the name of your worker

Paste the above information in the following script

_update\_cfworker.py_
```python
import requests
import base64
import json

flagDebug = False ' set to 'True' for debugging information

bucketSourceId = '{{bucket_id}}' # insert your Bucket ID
bucketFilenamePrefix = ''
# for b64 encoding.
b2AppKey = b'{{application_key}}' # insert your Application Key. Keep the letter b there
b2AppKeyId = b'{{application_key_id}}' # insert the App Key ID. Keep the letter b there

# Cloudflare settings
cfAccountId = '{{cloudflare_account_id}}' # insert our Cloudflare Account ID
cfWorkerApi = '{{cloudflare_edit_worker_api_key}}' # insert the API key to edit workers
cfWorkerName = '{{cloudflare_worker_name}}' # insert the worker's name


# An authorization token is valid for not more than 1 week
# This sets it to the maximum time value
maxSecondsAuthValid = 7*24*60*60 # one week in seconds

# DO NOT CHANGE ANYTHING BELOW THIS LINE ##

baseAuthorizationUrl = 'https://api.backblazeb2.com/b2api/v2/b2_authorize_account'
b2GetDownloadAuthApi = '/b2api/v2/b2_get_download_authorization'

# Get fundamental authorization code

idAndKey = b2AppKeyId + b':' + b2AppKey
b2AuthKeyAndId = base64.b64encode(idAndKey)
basicAuthString = 'Basic ' + b2AuthKeyAndId.decode('UTF-8')
authorizationHeaders = {'Authorization' : basicAuthString}
resp = requests.get(baseAuthorizationUrl, headers=authorizationHeaders)


if (resp.status_code >= 400):
    sys.exit("Status " + resp.status_code + ": Could not retrieve b2 authorization token from Backblaze.")

if flagDebug:
    print (resp.status_code)
    print (resp.headers)
    print (resp.content)

respData = json.loads(resp.content.decode("UTF-8"))

bAuToken = respData["authorizationToken"]
bFileDownloadUrl = respData["downloadUrl"]
bPartSize = respData["recommendedPartSize"]
bApiUrl = respData["apiUrl"]

# Get specific download authorization

getDownloadAuthorizationUrl = bApiUrl + b2GetDownloadAuthApi
downloadAuthorizationHeaders = { 'Authorization' : bAuToken}

resp2 = requests.post(getDownloadAuthorizationUrl,
                      json = {'bucketId' : bucketSourceId,
                              'fileNamePrefix' : "",
                              'validDurationInSeconds' : maxSecondsAuthValid },
                      headers=downloadAuthorizationHeaders )

if (resp2.status_code >= 400):
    sys.exit("Status " + resp.status_code + ": Could not retrieve b2 download authorization from Backblaze.")

resp2Content = resp2.content.decode("UTF-8")
resp2Data = json.loads(resp2Content)

bDownAuToken = resp2Data["authorizationToken"]

if flagDebug:
    print("authorizationToken: " + bDownAuToken)
    print("downloadUrl: " + bFileDownloadUrl)
    print("recommendedPartSize: " + str(bPartSize))
    print("apiUrl: " + bApiUrl)

workerTemplate = """addEventListener('fetch', event => {
    event.respondWith(handleRequest(event.request))
})
async function handleRequest(request) {
let authToken='<B2_DOWNLOAD_TOKEN>'
let b2Headers = new Headers(request.headers)
b2Headers.append("Authorization", authToken)
var requestFrom = request.url.replace("{{cnd_domain}}", "{{b2_bucket_domain}}"); # insert
modRequest = new Request(requestFrom, {
    method: request.method,
    headers: b2Headers
})
var response = await fetch(modRequest)

if(response.status >= 400) {
    var targetUrl = request.url.replace("https://{{cdn_domain}}/file/{{bucket_name}}/", "https://{{cloud_domain}}/"); # insert
	
    let requestHeaders = new Headers(request.headers)
    requestHeaders.append("Cookie", "consent=true")
	modRequest = new Request(targetUrl, {
		method: request.method,
		headers: {
            'Cookie': 'consent=true',
            'Accept': 'text/javascript,text/css,image/webp,*/*',
            'Accept-Encoding': 'br, zstd, gzip, deflate',
            'Host': 'cloud.liberale-demokraten.org',
            'Connection': 'keep-alive'
        }
	})
	
	response = await fetch(modRequest)
}

return response
}"""

workerCode = workerTemplate.replace('<B2_DOWNLOAD_TOKEN>', bDownAuToken)

cfHeaders = { 'Authorization' : "Bearer " + cfWorkerApi,
              'Content-Type' : 'application/javascript' }

cfUrl = 'https://api.cloudflare.com/client/v4/accounts/' + cfAccountId + "/workers/scripts/" + cfWorkerName

resp = requests.put(cfUrl, headers=cfHeaders, data=workerCode)

if (resp.status_code >= 400):
    sys.exit("Status " + resp.status_code + ": Could not update worker " + cfWorkerName)

if flagDebug:
    print(resp)
    print(resp.headers)
    print(resp.content)
```

NOTE: replace content in double curly brackets with the corresponding data. All such lines can be found by searching for lines including the term `# insert`.

## Set Up the Web Server
Now, you should be able to get any files within the B2 bucket through `https://{{cdn_domain}}/files/{{bucket_name}}/{{file_path}}` but that of course is not sufficient for us to be able to utilize our CDN. We also need to tell our visitors to fetch the resources through the CDN instead of through our own server. We also need to tell browsers, that the CDN domain is authorized to deliver resources for the LD Cloud. The latter, we do by adding CSP Headers.

NOTE: The following sections will deal with it by configuring nginx that way. If you use a different web server, the configuration may be different, but analogous.

### Content Security Policy (CSP)
Add the following line to your nginx configuration or create a new configuration with the following line and include it into your nginx configuration:

```nginx
add_header Content-Security-Policy "default-src 'none'; connect-src 'self' {{cdn_domain}}; prefetch-src  'self' {{cdn_domain}}; font-src 'self' {{cdn_domain}} data:; img-src 'self' {{cdn_domain}} data: https://usercontent.apps.nextcloud.com; media-src 'self'; script-src 'self' 'unsafe-inline' {{cdn_domain}}; style-src 'self' 'unsafe-inline' {{cdn_domain}}; frame-ancestors 'self'; report-to groupname";
```

!!! See the [CSP](../../web/content-security-policy) article in the Web section for more on setting the CSP header.

### Load Content from CDN
We now need to tell clients to fetch the resources not from our server directly but from our set-uo and running CDN. TO do so, we add the following to our nginx configuration:

```nginx
location / {
    sub_filter_once off;
    sub_filter '<link rel="stylesheet" href="/apps/accessibility/css/user-' '<link rel="stylesheet" href="/apps/accessibility/css/user-';
    sub_filter 'defer src="/core/' 'defer src="https://{{cdn_domain}}/file/{{bucket_name}}/core/';
    sub_filter 'defer src="/apps/' 'defer src="https://{{cdn_domain}}/file/{{bucket_name}}/apps/';
    sub_filter '<link rel="stylesheet" href="/apps/' '<link rel="stylesheet" href="https://{{cdn_domain}}/file/{{bucket_name}}/apps/';
    sub_filter '<link rel="stylesheet" href="/core/' '<link rel="stylesheet" href="https://{{cdn_domain}}/file/{{bucket_name}}/core/';
    sub_filter '<img alt="" src="/core/img/' '<img alt="" src="https://{{cdn_domain}}/file/{{bucket_name}}/core/img/';
    sub_filter '<img alt="" src="/apps/settings/' '<img alt="" src="https://{{cdn_domain}}/file/{{bucket_name}}/apps/settings/';
}
```

NOTE: After each update of the LD Cloud's core components (e.g. Nextcloud), you should re-check whether resources are still fetched through the CDN or the links have been modified. You might have to modify the above `sub_filter` directives accordingly. Also note that the above configuration will result in most of the defined type of components to be loaded through the CDN, but it won't load 100% of all desired contents through the CDN most likely. However, the above solution is sufficient for our purposes.

## Server Management Settings
We are using a server management platform based on VestaCP. It eases up multi-user multi-site configuration and enables means to set up the appropriate web and proxy configurations through templates. It is adviced to use the templates in that case as the management platform may override your manually edited configuration otherwise.

Read [Templates](../../server-setup/templates) for more details.