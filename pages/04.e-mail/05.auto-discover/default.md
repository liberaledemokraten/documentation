---
title: 'Auto Discover'
taxonomy:
    category:
        - email
    tag:
        - email
        - configuration
---

!!! If you are not interested in the technical details, but just want to use an e-mail client, check out [Autokonfiguration - LD Mail](https://mail.liberale-demokraten.de/setup/index.html) (in German).

Some email clients support a method called "automatic discovery" (or similar). That enables a quick setup of your e-mail accounts in the client. That however also requires us to provide information the client can retrieve. There are multiple methods for this depending on the used client.

## Table of Contents <a id="toc"></a>
  * [DNS SRV Records (RFC 6186)](#rfc6186)
  * [Thunderbird Autoconfiguration](#autoconfig)
  * [iOS mobileconfig](#mobileconfig)
  * [Outlook Autodiscover](#autodiscover) (not supported by us)

## DNS SRV Records (RFC 6186) <a id="rfc6186"></a> [↑Top](#toc)
!!! Please consider the hostname, protocols and ports described in [Webmail](https://doc.ld-online.net/e-mail/webmail). The below DNS records are just an example, you may have to modify it according to the details of the mailserver to be used. The documentation on this method can be found under [RFC 6186](https://tools.ietf.org/html/rfc6186).

This is the recommended standard method to go for. It requires DNS records to be set. In our case, the DNS records may look as follows:

```bind

_submission._tcp  IN    SRV  0   1  465  mail.example.com.
_imap._tcp        IN    SRV  0   0  0    .
_imaps._tcp       IN    SRV  0   1  993  mail.example.com.
_pop3._tcp        IN    SRV  0   0  0    .
_pop3s._tcp       IN    SRV  10  1  995  mail.example.com.
```

The above DNS entries mean that only IMAPS on port 993 and POP3S on port 995 (with encryption) are supported for the domain, but not plain (unencrypted) IMAP and POP3. It also means that IMAPS should be preferred if both IMAPS and POP3S can be used, as the priority field has a lower value for IMAPS (0) than for POP3S (10). The e-mail submission is done via SMTP on port 465 while the entry covers connections with and without TLS (in our case TLS-only).

## Thunderbird Autoconfiguration <a id="autoconfig"></a> [↑Top](#toc)
This method is supported by Mozilla Thunderbird and a few other clients that have implemented the same type of automatic detection. The details on this can be found under [Thunderbird:Autoconfiguration](https://wiki.mozilla.org/Thunderbird:Autoconfiguration).

### Defining Autoconfig Location [↑Top](#toc)
! Thunderbird by default tries to fetch the configuration file from `http://autoconfig.example.com/mail/config-v1.1.xml?emailaddress=address@example.com` and `http://example.com/.well-known/autoconfig/mail/config-v1.1.xml`. Checking the TXT record may not yet be implemented by Thunderbird. Thunderbird is also not RFC 6186 compatible as per their wiki article.

As described in [Thunderbird:Autoconfiguration:DNSBasedLookup](https://wiki.mozilla.org/Thunderbird:Autoconfiguration:DNSBasedLookup), it is possible to tell the client the location of the XML configuration file via a DNS TXT record. The record is supposed to look as follows:

```bind

@    IN    MX  10      mail.example.com.
@    IN    TXT         "mailconf=https://example.com/autoconfig.xml"
```
Note that the `mailconf` URL can be anything, but following requirements must be met:

* The URL must use the HTTPS protocol
* The webserver must return a valid and trusted SSL/TLS certificate
* The certificate must include the domain name of the e-mail address as it's common name (CN) or as a subject alternative name (SAN)
* The URL must return a valid XML configuration

### Webserver Configuration
Unfortunately, Thunderbird and many other clients do not check the DNS TXT record, so the configuration is fetched from the standard URLs. So, there is an easy tweak we can use in our nginx configuration:

```conf
rewrite ^(/.well-known/autoconfig/mail/config-v1.1.xml) /autodiscover/autoconfig.xml last;
```

Furthermore, we may want to save the configuration file under the mail provider's domain instead of with each domain. So we can fool Thunderbird with the following nginx configuration, assuming the file can be found in `https://mail.example.com/autodiscover/autoconfig.xml`:

```conf
location /autodiscover {
        proxy_set_header Host mail.example.com;
        proxy_pass      https://mail.example.com;
 }
```

### XML Configuration File [↑Top](#toc)
This is the XML file that should be fetched calling the above set `mailconf` URL. Details on this is specified in [Thunderbird:Autoconfiguration:ConfigFileFormat](https://wiki.mozilla.org/Thunderbird:Autoconfiguration:ConfigFileFormat).

```xml
<?xml version="1.0"?>
<clientConfig version="1.1">
    <emailProvider id="example.com">
      <domain>example.com</domain>
      <domain>example.org</domain>

      <displayName>Example E-Mail</displayName>
      <displayShortName>ExMail</displayShortName>

      <incomingServer type="imap">
         <hostname>mail.example.com</hostname>
         <port>993</port>
         <socketType>SSL</socketType>
         <username>%EMAILADDRESS%</username>
         <authentication>password-encrypted</authentication>
      </incomingServer>
        
      <incomingServer type="pop3">
         <hostname>mail.example.com</hostname>
         <port>995</port>
         <socketType>SSL</socketType>
         <username>%EMAILADDRESS%</username>
         <authentication>password-encrypted</authentication>
          <pop3>
            <!-- remove the following and leave to client/user? -->
            <leaveMessagesOnServer>true</leaveMessagesOnServer>
            <downloadOnBiff>true</downloadOnBiff>
         </pop3>
      </incomingServer>

      <outgoingServer type="smtp">
         <hostname>mail.example.com</hostname>
         <port>465</port>
         <socketType>SSL</socketType>
         <username>%EMAILADDRESS%</username>
         <authentication>password-encryptedt</authentication>
      </outgoingServer>


      <!-- A page where the ISP describes the configuration.
           This is purely informational.
           The text content should contains a description in the native
           language of the ISP (customers), and a short English description,
           mostly for us.
      -->
      <documentation url="https://doc.example.com/e-mail/">
         <descr lang="en">Configure ExMail via IMAP</descr>
         <descr lang="de">ExMail über IMAP konfigurieren</descr>
      </documentation>

    </emailProvider>


    <webMail>
      <loginPage url="https://mail.example.com/" />
      <loginPageInfo url="https://mail.example.com/">
        <username>%EMAILADDRESS%</username>
        <usernameField id="rcmloginuser" name="_user" />
        <passwordField id="rcmloginpwd" name="_pass" />
        <loginButton id="rcmloginsubmit" />
      </loginPageInfo>
    </webMail>

    <clientConfigUpdate url="https://www.example.com/config/mozilla.xml" />

</clientConfig>
```

Note that synchronizing the address book via CardDAV, calendar via CalDAV and sharing files via WebDAV (e.g. with Thunderbird FileLink) is also possible to be configured with the XML configuration. However, we haven't included those in the above file to keep the configuration easy to read.


## iOS mobileconfig <a id="mobileconfig"></a> [↑Top](#toc)
The `.mobileconfig` file used by iOS devices too is simply an XML file. However, it must be downloaded and installed on the device first for it to take effect. So, we will have to create a form where each e-mail user can download a customized profile for themselves. Meanwhile a basic configuration file will be stored on the server and only the parts to be customized will be replaced for the user.

### Request Form [↑Top](#toc)

```php
<?php if (isset($_REQUEST['email']) && (strpos($_REQUEST['email'], "@") !== false)): ?>
<?php
    header('Content-Type: text/plain');
    header("Content-Disposition: attachment; filename=\"exMail.mobileconfig\"");
    $conf = file_get_contents('ios.mobileconfig');
    $conf = str_replace('MAILADDR', $_REQUEST['email'], $conf);

    // assuming the email address is formatted as `firstname.lastname@example.com`
    $atPosition = stripos($_REQUEST['email'], "@", 0);
    $userPart = substr($_REQUEST['email'], 0, $atPosition);
    $dotPosition = stripos($userPart, ".", 0);
    $displayName = "";
    if ($dotPosition == false) {
      $displayName = ucfirst($userPart); // uppercase first character
    }
    else {
        $firstName = ucfirst(substr($userPart, 0, $dotPosition));
        $lastName = ucfirst(substr($userPart, $dotPosition+1));
        $displayName = $firstName . " " . $lastName;
    }
    $conf = str_replace('DISPLAYNAME', $displayName, $conf);
    print $conf;
?>
	  
<?php else: ?>
<html>
    <head>
        <title>iPhone Email Auto Configuration</title>
        <meta name="viewport" content="width=device-width; initial-scale=1; user-scalable=no" />
    </head>
    <body style="font: 100% Verdana;">
        <form method="post" action="iphone.php">
          <p style="text-align: center">
            Enter your email address to obtain an automatic configuration profile for your Email Account.<br/>
            <br/><br/>
            <input type="text" name="email"/><br/>
            <br/>
            <button type="submit">Submit</button>
          </p>
        </form>
    </body>
</html>
<?php endif ?>
```

### mobileconfig File [↑Top](#toc)
The mobileconfig file is saved as `ios.mobileconfig` in the server and looks as follows:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>ConsentText</key>
	<dict>
		<key>default</key>
		<string>This is a removable profile for iOS devices intended to ease up the email configuration.</string>
	</dict>
	<key>PayloadContent</key>
	<array>
		<dict>
			<key>EmailAccountDescription</key>
			<string>Liberale Demokraten</string>
			<key>EmailAccountName</key>
			<string>DISPLAYNAME</string>
			<key>EmailAccountType</key>
			<string>EmailTypeIMAP</string>
			<key>EmailAddress</key>
			<string>MAILADDR</string>
			<key>IncomingMailServerAuthentication</key>
			<string>EmailAuthPassword</string>
			<key>IncomingMailServerHostName</key>
			<string>mail.example.com</string>
			<key>IncomingMailServerPortNumber</key>
			<integer>993</integer>
			<key>IncomingMailServerUseSSL</key>
			<true/>
			<key>IncomingMailServerUsername</key>
			<string>MAILADDR</string>
			<key>OutgoingMailServerAuthentication</key>
			<string>EmailAuthPassword</string>
			<key>OutgoingMailServerHostName</key>
			<string>mail.example.com</string>
			<key>OutgoingMailServerPortNumber</key>
			<integer>465</integer>
			<key>OutgoingMailServerUseSSL</key>
			<true/>
			<key>OutgoingMailServerUsername</key>
			<string>MAILADDR</string>
			<key>OutgoingPasswordSameAsIncomingPassword</key>
			<true/>
			<key>PayloadDescription</key>
			<string>Configures email account.</string>
			<key>PayloadDisplayName</key>
			<string>Example</string>
			<key>PayloadIdentifier</key>
			<string>com.example.mail.profile.email1</string>
			<key>PayloadOrganization</key>
			<string>Example Organization</string>
			<key>PayloadType</key>
			<string>com.apple.mail.managed</string>
			<key>PayloadUUID</key>
			<string>08E88671-C2CB-45CA-99B3-CE20C9D08A66</string>
			<key>PayloadVersion</key>
			<integer>1</integer>
			<key>PreventAppSheet</key>
			<false/>
			<key>PreventMove</key>
			<false/>
			<key>SMIMEEnabled</key>
			<false/>
			<key>disableMailRecentsSyncing</key>
			<false/>
		</dict>
	</array>
	<key>PayloadDescription</key>
	<string>This profile was enabled by Example Organization to ease up the email configuration for iOS devices (e.g. iPhone, iPad, etc.)</string>
	<key>PayloadDisplayName</key>
	<string>Example Mail</string>
	<key>PayloadIdentifier</key>
	<string>com.example.mail.profile</string>
	<key>PayloadOrganization</key>
	<string>Example Organization</string>
	<key>PayloadRemovalDisallowed</key>
	<false/>
	<key>PayloadType</key>
	<string>Configuration</string>
	<key>PayloadUUID</key>
	<string>EA5F1000-208D-4138-8ADE-ABC12DE3F456</string>
	<key>PayloadVersion</key>
	<integer>1</integer>
</dict>
</plist>
```


## Outlook Autodiscover <a id="autodiscover"></a> [↑Top](#toc)
! Some linked Microsoft Docs articles are already dead and the feature is barely well documented. The below method is tested to **not** function with Outlook as part of Office 365.

The Outlook Client (as of Office 2007) follows the Autodiscover as outlined in [Plan to automatically configure user accounts in Outlook 2010 - Microsoft Docs](https://docs.microsoft.com/en-us/previous-versions/office/office-2010/cc511507(v=office.14)?redirectedfrom=MSDN#imap-settings). More in-depth information can be found in the above documentation, as well as in [E-Mail Autoconfig & AutoDiscover - Matthias P. Würfl](https://matthias.wuerfl.com/email-autoconfig-autodiscover/) and [Providing Email client autoconfiguration information - moens.ch (archived)](http://web.archive.org/web/20150102224744/http://moens.ch/2012/05/31/providing-email-client-autoconfiguration-information/).

Outlook fetches the autocondiscover file in XML format in the following order:

1. POST https://example.com/autodiscover/autodiscover.xml
2. POST https://autodiscover.example.com/autodiscover/autodiscover.xml
3. GET http://autodiscover.example.com/autodiscover/autodiscover.xml (accepts an HTTP 302 to the correct URL)
4. From the domain the DNS SRV `_autodiscover._tcp.example.com` record points to following the above order

### Sample POST Request
Here's an example for a POST request made by the Outlook client:

```xml
<?xml version="1.0" encoding="utf-8"?>
    <Autodiscover xmlns="http://schemas.microsoft.com/exchange/autodiscover/outlook/requestschema/2006">
      <Request>
        <EMailAddress>max.mustermann@example.com</EMailAddress>
        <AcceptableResponseSchema>http://schemas.microsoft.com/exchange/autodiscover/outlook/responseschema/2006a</AcceptableResponseSchema>
      </Request>
    </Autodiscover>
```

### Webserver Configuration [↑Top](#toc)
!!! Note that the request URI may vary depending on Outlook's version. Modern applications seem to query `/Autodiscover/Autodiscover.xml` while some older versions go for `/autodiscover/autodiscover.xml`. So, both URLs should return a response.

As we are running nginx in the front, rewriting the XML into a PHP script makes more sense and is less troublesome.

```conf
rewrite ^(/autodiscover/autodiscover.xml) /autodiscover/autodiscover.php last;
rewrite ^(/Autodiscover/Autodiscover.xml) /autodiscover/autodiscover.php last;
```

The `location /autodiscover` should be set up as done above in the [Thunderbird autoconfig](#autoconfig). 

### XML Configuration File [↑Top](#toc)
The following is an example configuration file with a few advanced features. Upload it as a PHP file instead of as an XML file if the above webserver configuration was used.

```php
<?php
$expiration = date('Ymd', strtotime("+1 week"));
$ttl = 168; // 7 days
$raw = file_get_contents('php://input');
$loginName = array();
preg_match('/<EMailAddress>(.*)<\/EMailAddress>/', $raw, $loginName);
header('Content-Type: application/xml');

// assuming the email address is formatted as `firstname.lastname@example.com`
$atPosition = stripos($loginName[1], "@", 0);
$userPart = substr($loginName[1], 0, $atPosition);
$dotPosition = stripos($userPart, ".", 0);
$displayName = "";
if ($dotPosition == false) {
  $displayName = ucfirst($userPart); // uppercase first character
}
else {
	$firstName = ucfirst(substr($userPart, 0, $dotPosition));
	$lastName = ucfirst(substr($userPart, $dotPosition));
	$displayName = $firstName . " " . $lastName;
}
?>

<? echo '<? xml version="1.0" encoding="utf-8" ?>' ?>
<Autodiscover xmlns="http://schemas.microsoft.com/exchange/autodiscover/responseschema/2006">
<Response xmlns="http://schemas.microsoft.com/exchange/autodiscover/outlook/responseschema/2006a">
<!-- User: Optional
This tag gives user-specific information.  Autodiscover must be UTF-8 encoded.
-->
<User>
<!-- DisplayName: Optional
The server may have a good formal display name.  The client can decide to accept it or change it.  This will save the user time in the default case.
-->
<DisplayName><?php echo $displayName ?></DisplayName>
</User>


<Account>
<AccountType>email</AccountType>

<!-- Action: Required
This value indicates if the goal of this account results is to provide the settings or redirect to another web server that can provide results.
VALUES:
redirectUrl: If this value is specified, then the URL tag will specify the http: or https: URL containing the Autodiscover results to be used.  In order to prevent the server from being able to send the client into an infinite loop, the client should stop redirecting after 10 redirects.
redirectAddr: If this value is specified, then the XML tag will specify the e-mail address that Outlook should use to execute Autodiscover again.  In other words, the server is telling the client that the e-mail address the client should really be using for Autodiscover is not the one that was posted, but the one specified in this tag. 
settings: If this value is specified, then the XML will contain the settings needed to configure the account.  The settings will primarily be under the PROTOCOL tag.
-->
<Action>settings</Action>

<!-- Image: Optional
This is a JPG picture to brand the ISP configuration experience with. The client can choose whether or not they download this picture to display. (not used by Outlook 2007)
-->
<Image>https://mail.example.com/icon.png</Image>

<!-- ServiceHome: Optional
This is a link to the ISP's Home Page. The client can choose whether or not they expose this link to the user. (not used by Outlook 2007)
-->
<ServiceHome>https://mail.example.com</ServiceHome>

<Protocol>
<!-- TYPE: Required.
The value here specifies what kind of mail account is being configured.
POP3: The protocol to connect to this server is POP3. Only applicable for AccountType=email.
SMTP: The protocol to connect to this server is SMTP. Only applicable for AccountType=email.
IMAP: The protocol to connect to this server is IMAP. Only applicable for AccountType=email.
DAV: The protocol to connect to this server is DAV. Only applicable for AccountType=email.
WEB: Email is accessed from a web browser using an URL from the SERVER tag. Only applicable for AccountType=email. (not used by Outlook 2007)
NNTP: The protocol to connect to this server is NNTP. Only applicable for AccountType=nntp. (not used by Outlook 2007)
-->
	<Type>IMAP</Type>
	
	<!-- ExpirationDate: Optional.
	The value here specifies the last date which these settings should be used. After that date, the settings should be rediscovered via Autodiscover again. If no value is specified, the default will be no expiration.
	-->
	<ExpirationDate><?php echo $expiration ?></ExpirationDate>
	
	<!-- TTL: Optional.
	The value here specifies the time to live in hours that these settings are valid for. After that time has elapsed (from the time the settings were retrieved), the settings should be rediscovered via Autodiscovery again. A value of 0 indicates that no rediscovery will be required. If no value is specified, the default will be a TTL of 1 hour.
	-->
	<TTL><?php echo $ttl ?></TTL>

	<!-- Server: Required.
	The value here specifies the name of the mail server corresponding to the server type specified above.
	For protocols such as POP3, SMTP, IMAP, or NNTP, this value will be either a hostname or an IP address.
	For protocols such as DAV or WEB, this will be an URL.
	-->
	<Server>mail.example.com</Server>
	<Port>993</Port>

	<!-- LoginName: Optional.
	This value specifies the user's login.  If no value is specified, the default will be set to the string preceding the '@' in the email address.  If the Login name contains a domain, the format should be <Username>@<Domain>.  Such as JoeUser@SalesDomain.
	-->
	<LoginName><?php echo $loginName[1]; ?></LoginName>

	<!-- DomainRequired: Optional.  Default is off.
	If this value is true, then a domain is required during authentication.  If the domain is not specified in the LOGINNAME tag, or the LOGINNAME tag was not specified, the user will need to enter the domain before authentication will succeed.
	-->
	<DomainRequired>on</DomainRequired>

	<!-- DomainName: Optional.
	This value specifies the user's domain. If no value is specified, the default authentication will be to use the e-mail address as a UPN format <Username>@<Domain>. Such as JoeUser@SalesDomain.
	-->
	<DomainName>example.com</DomainName>

	<!-- SPA: (Secure Password Authentication) Optional.
	This value specifies whether or not secure password authentication is needed.
	If unspecified, the default is set to on.
	-->
	<SPA>on</SPA>

	<!-- SSL: Optional.
	This value specifies whether secure login is needed.
	If unspecified, the default is set to on.
	-->
	<SSL>on</SSL>

	<!-- AuthRequired: Optional.
	This value specifies whether authentication is needed (password).
	If unspecified, the default is set to on.
	-->
	<AuthRequired>on</AuthRequired>

</Protocol>

<Protocol>
	<Type>POP3</Type>
	<ExpirationDate><?php echo $expiration ?></ExpirationDate>
	<TTL><?php echo $ttl ?></TTL>
	<Server>mail.example.com</Server>
	<Port>995</Port>
	<LoginName><?php echo $loginName[1]; ?></LoginName>
	<DomainRequired>on</DomainRequired>
	<DomainName>example.com</DomainName>
	<SPA>on</SPA>
	<SSL>on</SSL>
	<AuthRequired>on</AuthRequired>

	<!-- UsePOPAuth: Optional.
	This value can only be used for SMTP types.
	If specified, then the authentication information provided for the POP3 type account will also be used for SMTP.
	-->
	<UsePOPAuth>on</UsePOPAuth>
</Protocol>

<Protocol>
	<Type>SMTP</Type>
	<ExpirationDate><?php echo $expiration ?></ExpirationDate>
	<TTL><?php echo $ttl ?></TTL>
	<Server>mail.example.com</Server>
	<Port>465</Port>
	<LoginName><?php echo $loginName[1]; ?></LoginName>
	<DomainRequired>on</DomainRequired>
	<DomainName>example.com</DomainName>
	<SPA>on</SPA>
	<SSL>on</SSL>
	<Encryption>TLS</Encryption>
	<AuthRequired>on</AuthRequired>
</Protocol>

</Account>
</Response>
</Autodiscover>
```