2022-07-07 21:44
source: [HackTricks](https://book.hacktricks.xyz/cloud-security/jenkins#code-execution)
tags: #pages #cybersec #HackTricks #exploitation 


# Code Execution on [[Jenkins]]


## Overview
This Page give basic scripts or scans that can be ran to easily detect vulnerabilities on Jenkins.

## Details
### Unauthenticated Enumeration 
#### check what I can run without creds
- This uses [[Metasploit]] > thus the msf flag in the code.
    - [Page search](Jenkins-CI%20Enumeration) `msf> use auxiliary/scanner/http/jenkins_enum`
    - Command search `msf> use auxiliary/scanner/http/jenkins_command`


## Summary
