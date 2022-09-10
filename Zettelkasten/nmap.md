2022-07-07 20:18
source : [Manpage](https://linux.die.net/man/1/nmap)
tags: #tools #network #cybersec

# nmap
network scanner
## Usage
`-sC` This is equivalent to `--script=default` Default script in nmap is intrusive. Easily detected so don't use on other's servers.

`-sV` Probes open ports, checks service and their versions.

`-O` OS detection

`-Pn` Skips ping. so skips host discovery. This catches the cases where the server denies a ping, making it look like it is down but actually online. 

