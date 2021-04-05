---
title: 'Reporting & Monitoring'
published: true
---

There are multiple tools for receiving reports for various reasons. All of them are set through HTTP headers with each response. The following three are the recommended ones and are achieved through adding them as headers in HTTP reponses.

## Reporting
The following HTTP headers can be set to enable reporting of errors and security risks.

### Report-To
The Report-To header defines where reports shall be sent. It is set as follows:

```
add_header Report-To "{\"group\":\"groupname\",\"max_age\":31536000,\"endpoints\":[{\"url\":\"https://report.example.com/a/d/g\"}],\"include_subdomains\":true}";
```

### CSP Reporting
We have already dealt about CSP. Thanks to CSP reporting however, you can be informed of any violations to the policies you set. Here an example on how to achieve that:

```
add_header Content-Security-Policy "default-src 'self'; report-to groupname";
```

### Network Error Logging
Network Error Logging (NEL) reports if the client failed to load the site due to some network errors, e.g. if the server is down. It can be set as follows:

```
add_header NEL "{\"report_to\":\"groupname\",\"max_age\":31536000,\"include_subdomains\":true}";
```

### Expect-CT
This enables reporting and enforcement of Certificate Transparency requirements. It aims to prevent the use of misissued certificates for the website from being unnoticed. If the `enforce` keyword is missing, the client may report without enforcing.

```
add_header Expect-CT 'max-age=3600, report-uri="https://report.example.com/r/d/ct/enforce", enforce';
```


## Monitoring
Of course, you may also want to monitor the reported errors. To do so, it is advised to use the [report-uri](https://report-uri.com) tool of Scott Helme. Otherwise, you might have to set up your very own infrastructure that is being able to both receive and process the reports in a way you can see them.