---
marp: true
title: Php cURL Security Tips
---
<style>
section{
    background: #C8C7CB;
}
h1{
    font-size:55px;
    color: #3F5891;
    text-align: center;
    border-bottom: 1px solid;
}
h5{
    font-size: 25px;
    text-align: center;
}
</style>

![bg left](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*F5tboLGE_oO_9xRx)

# Securing Php-cURL Server-to-Server Communications

##### Presented By
<style scoped>
img {
    border-radius: 50%;
    height: 50px;
    width:50px;
    vertical-align: middle;
    display: inline;
}
h6{
    color: #3F5891;
    text-align: center;
}
thead{
    font-weight: bold;
    font-size: 24px;
}
th{
  background:  #C8C7CB;
}
table {
    border-collapse: collapse !important;
    margin: 6px auto 0;
}
table, tr, td, th {
  border: 0 !important;
  background: transparent !important;
}
td {
  text-align: center;
  vertical-align: middle;
}
</style>
|![](https://miro.medium.com/v2/resize:fill:176:176/1*rmTbkWfJPJkuHaL9p0xFyw.png) | Parthipan Natkunam |
| ------------- | ------------- |

---

## cURL Post
```php
<?php
// Data to be sent
$postData = ['key' => 'value'];
// Initialize cURL
$curl = curl_init('http://service1.example.com/api');
// Set options to POST
curl_setopt($curl, CURLOPT_POST, true);
curl_setopt($curl, CURLOPT_POSTFIELDS, $postData);
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
// Execute and close session
$response = curl_exec($curl);
curl_close($curl);
?>
```

`CURLOPT_RETURNTRANSFER` returns the data rather than echoing it.

---
## Prevent Infinite Redirects
- By default, cURL doesn’t follow redirects.
- But in practice, we might run into situations where we may have to follow a redirect or two.
- __if you had enabled this redirect option then limit the maximum number of redirects to follow__
```php
curl_setopt($curl, CURLOPT_MAXREDIRS, 3);
```
- `CURLOPT_FOLLOWLOCATION` option can be used to configure this.
<style scoped>
table{
    display: table;
    width: 100%;
    text-align: center;
}
</style>

| MAXREDIRS Value  | Implication |
| ------------- | ------------- |
| -1 | Allow infinite redirections  |
|  0 | Disable all redirections  |

⚠️ **Never use -1 as a value for this setting.**

---
## Restrict Protocols
- Supports multiple protocols : FTP, LDAP, etc.
- Limit it to HTTPS and if required HTTP.
```php
curl_setopt($curl, CURLOPT_PROTOCOLS, CURLPROTO_HTTP | CURLPROTO_HTTPS);
```
- Always restrict the the redirect protocols:
```php
curl_setopt($curl, CURLOPT_REDIR_PROTOCOLS, CURLPROTO_HTTPS);
```

---
## Set a Timeout
- To prevent servers from keeping the cURL session alive for longer periods than necessary,always set a reasonable time-out value to the session.
```php
curl_setopt($curl, CURLOPT_TIMEOUT, 15);
```
- The value is in **seconds**.
- The above configuration will make cURL wait for 15 seconds per request before timing out.

⚠️ If a **server redirects, the timeout won’t be reset**. So in total, a server would have 15 seconds to respond with the final response, including responses from redirects if any.

---
## Verify Certificate Domains
```php
curl_setopt($curl, CURLOPT_SSL_VERIFYHOST, 2);
```
- verify that a Common Name field or a Subject Alternate Name field in the SSL peer certificate matches the provided hostname
- Always use value `2` in Prod.

⚠️ Never use the value `1` in production (this is due to certain vulnerabilities in the older versions of TLS, that were patched in later versions). **Support for value 1 removed in cURL 7.28.1.**

---
## Verify Issuer
```php
curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, true);
```
- verifies if the certificate was issued by a trusted Certificate Authority (CA).
-  If enabled cURL will start rejecting self-signed certificates.

---
## Ensure the Certificate Hasn’t Been Revoked
```php
curl_setopt($curl, CURLOPT_SSL_VERIFYSTATUS, true);
```
- Verify the SSL certificate status.
-  If this option is set then, cURL will verify that the server attaches the [Online Certificate Status Protocol (OCSP)](https://www.techtarget.com/searchsecurity/definition/OCSP) response during the TLS handshake.

---
# Thank You