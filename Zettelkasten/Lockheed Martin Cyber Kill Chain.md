2022-12-25 16:30
source: [LM](https://www.lockheedmartin.com/content/dam/lockheed-martin/rms/documents/cyber/LM-White-Paper-Intel-Driven-Defense.pdf)
tags: #pages #cti #cybersec 


# Lockheed Martin Cyber Kill Chain


## Overview
Structural map of adversary intrusion.
- Structure in "chains" or "stages" where disruption or mitigation of one stage disrupts the structure and the adversary's campaign.

## Details

#### Indicators and Indicator Life Cycle
- **Atomic**
==cannot be broken down into smaller==
	-  IP addresses
	- email addresses
- **Computed**
==derived from data involved in an incident==
	- hash values
	- regex
- **Behavioral**
==collections of computed and atomic indicators==
	- "intruder would initially use a backdoor which generated network traffic matching \[regex\] at the rate of \[some frequency\] to \[some IP address\], and then replace it with one matching the MD5 hash \[value\] once access was established"
![[IndicatorLifeCycle.png]]

Revealed -> (leveraged by using the indicator) -> Mature -> (matching activity discovered) -> Utilized -> (Analyze) 

