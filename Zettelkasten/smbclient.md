2022-07-20 23:48
source : [manpage]()
tags: #tools #SMB #cybersec #microsoft #samba

# smbclient

## Usage

This is a client to access SMB servers. 

## Hacking 

As seen in [[Tactics]] , can be used to access SMB servers to further enumerate.

```
smbclient {servicename} [password] [-b <buffer size>] [-d debuglevel] [-e] [-D Directory]
        [-U username] [-W workgroup] [-M <netbios name>] [-m maxprotocol] [-A authfile] [-N] [-C]
        [-g] [-l log-basename] [-I destinationIP] [-E] [-c <command string>] [-i scope]
        [-O <socket options>] [-p port] [-R <name resolve order>] [-s <smb config file>]
        [-t <per-operation timeout in seconds>] [-T<c|x>IXFvgbNan] [-k]
```

{servicename} is inform of `//server/service` `server` is not necessarily the ip address, but the name #netBIOS gives. So may have to access netBIOS first to gain the server name.

Another: If `server == ip`, `smbclient -L host` should give all available shares. (password may be there) If `server != ip`, `smbclient -I host` may help

^ Definitely helps. It helped for [[Archetype#Thought Process]]
