# Open Source

## User Flag

### Enumeration

As always, let's open with a nice big nmap scan and see what results we get after adding to our /etc/hosts.

```
nmap -A -sV 10.10.11.164 -p-
Nmap scan report for opensource.htb (10.10.11.164)
Host is up (0.076s latency).
Not shown: 65532 closed ports
PORT     STATE    SERVICE VERSION
22/tcp   open     ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|_  256 02:1f:97:9e:3c:8e:7a:1c:7c:af:9d:5a:25:4b:b8:c8 (ED25519)
80/tcp   open     http    Werkzeug/2.1.2 Python/3.10.3
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.1.2 Python/3.10.3
|     Date: Thu, 11 Aug 2022 22:50:12 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 5316
|     Connection: close
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>upcloud - Upload files for Free!</title>
|     <script src="/static/vendor/jquery/jquery-3.4.1.min.js"></script>
|     <script src="/static/vendor/popper/popper.min.js"></script>
|     <script src="/static/vendor/bootstrap/js/bootstrap.min.js"></script>
|     <script src="/static/js/ie10-viewport-bug-workaround.js"></script>
|     <link rel="stylesheet" href="/static/vendor/bootstrap/css/bootstrap.css"/>
|     <link rel="stylesheet" href=" /static/vendor/bootstrap/css/bootstrap-grid.css"/>
|     <link rel="stylesheet" href=" /static/vendor/bootstrap/css/bootstrap-reboot.css"/>
|     <link rel=
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.1.2 Python/3.10.3
|     Date: Thu, 11 Aug 2022 22:50:12 GMT
|     Content-Type: text/html; charset=utf-8
|     Allow: HEAD, GET, OPTIONS
|     Content-Length: 0
|     Connection: close
|   RTSPRequest: 
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
|     "http://www.w3.org/TR/html4/strict.dtd">
|     <html>
|     <head>
|     <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: HTTPStatus.BAD_REQUEST - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
|_http-server-header: Werkzeug/2.1.2 Python/3.10.3
3000/tcp filtered ppp
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port80-TCP:V=7.80%I=7%D=8/11%Time=62F58745%P=x86_64-pc-linux-gnu%r(GetR
SF:equest,1573,"HTTP/1\.1\x20200\x20OK\r\nServer:\x20Werkzeug/2\.1\.2\x20P
SF:ython/3\.10\.3\r\nDate:\x20Thu,\x2011\x20Aug\x202022\x2022:50:12\x20GMT
SF:\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x20
SF:5316\r\nConnection:\x20close\r\n\r\n<html\x20lang=\"en\">\n<head>\n\x20
SF:\x20\x20\x20<meta\x20charset=\"UTF-8\">\n\x20\x20\x20\x20<meta\x20name=
SF:\"viewport\"\x20content=\"width=device-width,\x20initial-scale=1\.0\">\
SF:n\x20\x20\x20\x20<title>upcloud\x20-\x20Upload\x20files\x20for\x20Free!
SF:</title>\n\n\x20\x20\x20\x20<script\x20src=\"/static/vendor/jquery/jque
SF:ry-3\.4\.1\.min\.js\"></script>\n\x20\x20\x20\x20<script\x20src=\"/stat
SF:ic/vendor/popper/popper\.min\.js\"></script>\n\n\x20\x20\x20\x20<script
SF:\x20src=\"/static/vendor/bootstrap/js/bootstrap\.min\.js\"></script>\n\
SF:x20\x20\x20\x20<script\x20src=\"/static/js/ie10-viewport-bug-workaround
SF:\.js\"></script>\n\n\x20\x20\x20\x20<link\x20rel=\"stylesheet\"\x20href
SF:=\"/static/vendor/bootstrap/css/bootstrap\.css\"/>\n\x20\x20\x20\x20<li
SF:nk\x20rel=\"stylesheet\"\x20href=\"\x20/static/vendor/bootstrap/css/boo
SF:tstrap-grid\.css\"/>\n\x20\x20\x20\x20<link\x20rel=\"stylesheet\"\x20hr
SF:ef=\"\x20/static/vendor/bootstrap/css/bootstrap-reboot\.css\"/>\n\n\x20
SF:\x20\x20\x20<link\x20rel=")%r(HTTPOptions,C7,"HTTP/1\.1\x20200\x20OK\r\
SF:nServer:\x20Werkzeug/2\.1\.2\x20Python/3\.10\.3\r\nDate:\x20Thu,\x2011\
SF:x20Aug\x202022\x2022:50:12\x20GMT\r\nContent-Type:\x20text/html;\x20cha
SF:rset=utf-8\r\nAllow:\x20HEAD,\x20GET,\x20OPTIONS\r\nContent-Length:\x20
SF:0\r\nConnection:\x20close\r\n\r\n")%r(RTSPRequest,1F4,"<!DOCTYPE\x20HTM
SF:L\x20PUBLIC\x20\"-//W3C//DTD\x20HTML\x204\.01//EN\"\n\x20\x20\x20\x20\x
SF:20\x20\x20\x20\"http://www\.w3\.org/TR/html4/strict\.dtd\">\n<html>\n\x
SF:20\x20\x20\x20<head>\n\x20\x20\x20\x20\x20\x20\x20\x20<meta\x20http-equ
SF:iv=\"Content-Type\"\x20content=\"text/html;charset=utf-8\">\n\x20\x20\x
SF:20\x20\x20\x20\x20\x20<title>Error\x20response</title>\n\x20\x20\x20\x2
SF:0</head>\n\x20\x20\x20\x20<body>\n\x20\x20\x20\x20\x20\x20\x20\x20<h1>E
SF:rror\x20response</h1>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Error\x20code
SF::\x20400</p>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Message:\x20Bad\x20req
SF:uest\x20version\x20\('RTSP/1\.0'\)\.</p>\n\x20\x20\x20\x20\x20\x20\x20\
SF:x20<p>Error\x20code\x20explanation:\x20HTTPStatus\.BAD_REQUEST\x20-\x20
SF:Bad\x20request\x20syntax\x20or\x20unsupported\x20method\.</p>\n\x20\x20
SF:\x20\x20</body>\n</html>\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 239.62 seconds
```

Looks like we've got a port 22 and 80 open but one on 3000 filtered. It's listed as ppp which on https://www.speedguide.net/port.php?port=3000 is a "User-level ppp daemon.

Ppp also known as pppd stands for Point-To-Point Protcol daemon and is meant to manage network connections between two nodes on Unix-like operating systems.

Okay let's check out the website. After accessing port 80, it looks like it's called upcloud and it's for file transfers.

I immediately found a download button and immediately pulled it down. It contains a DockerFile and a bash script to run the DockerFile.

*DockerFile*

```
FROM python:3-alpine

# Install packages
RUN apk add --update --no-cache supervisor

# Upgrade pip
RUN python -m pip install --upgrade pip

# Install dependencies
RUN pip install Flask

# Setup app
RUN mkdir -p /app

# Switch working environment
WORKDIR /app

# Add application
COPY app .

# Setup supervisor
COPY config/supervisord.conf /etc/supervisord.conf

# Expose port the server is reachable on
EXPOSE 80

# Disable pycache
ENV PYTHONDONTWRITEBYTECODE=1

# Set mode
ENV MODE="PRODUCTION"

# Run supervisord
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
```

*build-docker.sh*

```
#!/bin/bash
docker rm -f upcloud
docker build --tag=upcloud .
docker run -p 80:80 --rm --name=upcloud upcloud
```

Back to the webpage, there is also an upload page. This is another potential vector we could exploit.

I would also like to point out that I also found a /console directory as well that requires a pin to use.

So inside the source.zip we downloaded on the main page, there is a hidden .git directory.

I next checked the logs and found that the Dockerfile had been modified.

```
git log --raw
commit 2c67a52253c6fe1f206ad82ba747e43208e8cfd9 (HEAD -> public)
Author: gituser <gituser@local>
Date:   Thu Apr 28 13:55:55 2022 +0200

    clean up dockerfile for production use

:100644 100644 76c7768 5b0553c M        Dockerfile

commit ee9d9f1ef9156c787d53074493e39ae364cd1e05
Author: gituser <gituser@local>
Date:   Thu Apr 28 13:45:17 2022 +0200

    initial

:000000 100644 0000000 76c7768 A        Dockerfile
:000000 100644 0000000 e69de29 A        app/INSTALL.md
:000000 100644 0000000 5c2ecc0 A        app/app/__init__.py
:000000 100644 0000000 877f291 A        app/app/configuration.py
:000000 100644 0000000 91e52af A        app/app/static/css/style.css
:000000 100644 0000000 b335ef9 A        app/app/static/js/ie10-viewport-bug-workaround.js
:000000 100644 0000000 401c744 A        app/app/static/js/script.js
:000000 100644 0000000 259a9e2 A        app/app/static/vendor/bootstrap/css/bootstrap-grid.css
:000000 100644 0000000 8661e3e A        app/app/static/vendor/bootstrap/css/bootstrap-grid.css.map
:000000 100644 0000000 6533f31 A        app/app/static/vendor/bootstrap/css/bootstrap-grid.min.css
:000000 100644 0000000 1b393db A        app/app/static/vendor/bootstrap/css/bootstrap-grid.min.css.map
:000000 100644 0000000 91b0fc4 A        app/app/static/vendor/bootstrap/css/bootstrap-reboot.css
:000000 100644 0000000 701f671 A        app/app/static/vendor/bootstrap/css/bootstrap-reboot.css.map
:000000 100644 0000000 5308df6 A        app/app/static/vendor/bootstrap/css/bootstrap-reboot.min.css
:000000 100644 0000000 b8551f7 A        app/app/static/vendor/bootstrap/css/bootstrap-reboot.min.css.map
:000000 100644 0000000 8eac957 A        app/app/static/vendor/bootstrap/css/bootstrap.css
:000000 100644 0000000 521afc5 A        app/app/static/vendor/bootstrap/css/bootstrap.css.map
:000000 100644 0000000 86b6845 A        app/app/static/vendor/bootstrap/css/bootstrap.min.css
:000000 100644 0000000 b939eb6 A        app/app/static/vendor/bootstrap/css/bootstrap.min.css.map
:000000 100644 0000000 5344522 A        app/app/static/vendor/bootstrap/js/bootstrap.bundle.js
:000000 100644 0000000 93cf73e A        app/app/static/vendor/bootstrap/js/bootstrap.bundle.js.map
:000000 100644 0000000 78c533b A        app/app/static/vendor/bootstrap/js/bootstrap.bundle.min.js
:000000 100644 0000000 54d2495 A        app/app/static/vendor/bootstrap/js/bootstrap.bundle.min.js.map
:000000 100644 0000000 f1e68d3 A        app/app/static/vendor/bootstrap/js/bootstrap.js
:000000 100644 0000000 e5f6ce9 A        app/app/static/vendor/bootstrap/js/bootstrap.js.map
:000000 100644 0000000 e5a2429 A        app/app/static/vendor/bootstrap/js/bootstrap.min.js
:000000 100644 0000000 757dbf3 A        app/app/static/vendor/bootstrap/js/bootstrap.min.js.map
:000000 100644 0000000 e1e271c A        app/app/static/vendor/font-awesome/all.min.css
:000000 100644 0000000 773ad95 A        app/app/static/vendor/jquery/jquery-3.4.1.js
:000000 100644 0000000 a1c07fd A        app/app/static/vendor/jquery/jquery-3.4.1.min.js
:000000 100644 0000000 e15edef A        app/app/static/vendor/jquery/jquery-3.4.1.min.map
:000000 100644 0000000 a88adb2 A        app/app/static/vendor/popper/popper-utils.js
:000000 100644 0000000 50e078a A        app/app/static/vendor/popper/popper-utils.js.map
:000000 100644 0000000 f6560a1 A        app/app/static/vendor/popper/popper-utils.min.js
:000000 100644 0000000 7fda3a9 A        app/app/static/vendor/popper/popper-utils.min.js.map
:000000 100644 0000000 7fa913d A        app/app/static/vendor/popper/popper.js
:000000 100644 0000000 d56fdda A        app/app/static/vendor/popper/popper.js.flow
:000000 100644 0000000 9633363 A        app/app/static/vendor/popper/popper.js.map
:000000 100644 0000000 8a17212 A        app/app/static/vendor/popper/popper.min.js
:000000 100644 0000000 7107f61 A        app/app/static/vendor/popper/popper.min.js.map
:000000 100644 0000000 29218c7 A        app/app/templates/index.html
:000000 100644 0000000 316cd1d A        app/app/templates/success.html
:000000 100644 0000000 fd19858 A        app/app/templates/upload.html
:000000 100644 0000000 eebf49d A        app/app/utils.py
:000000 100644 0000000 f2744c6 A        app/app/views.py
:000000 100644 0000000 b64e36b A        app/run.py
:000000 100755 0000000 1b76267 A        build-docker.sh
:000000 100644 0000000 a14dd36 A        config/supervisord.conf
jex@jex-kubuntu:~/HTB/Loot/OpenSource/.git$ git diffee9d9f1ef9156c787d53074493e39ae364cd1e05:Dockerfile HEAD:Dockerfile
git: 'diffee9d9f1ef9156c787d53074493e39ae364cd1e05:Dockerfile' is not a git command. See 'git --help'.
jex@jex-kubuntu:~/HTB/Loot/OpenSource/.git$ git diff ee9d9f1ef9156c787d53074493e39ae364cd1e05:Dockerfile HEAD:Dockerfile
diff --git a/Dockerfile b/Dockerfile
index 76c7768..5b0553c 100644
--- a/Dockerfile
+++ b/Dockerfile
@@ -29,7 +29,6 @@ ENV PYTHONDONTWRITEBYTECODE=1
 
 # Set mode
 ENV MODE="PRODUCTION"
-# ENV FLASK_DEBUG=1
 
 # Run supervisord
 CMD ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
```

Now after doing some digging, we can create a PIN based on an md5 hash using this script: https://github.com/devdg/htb-opensource/blob/main/keygen.py

The pin is: 866-968-626



