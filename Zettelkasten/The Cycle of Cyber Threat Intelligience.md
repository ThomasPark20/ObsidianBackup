2022-11-21 20:37
source: [Katie Nickels](https://www.youtube.com/watch?v=J7e74QLVxCk)
tags: #research #cti #cybersec


# The Cycle of Cyber Threat Intelligience

![[TheIntelligienceCycle.png]]

## Planning and Direction

1. Intelligience Requirements
	- What people from the organzation need from the CTI team.
		1. Gaps or Questions that need to be addressed.
	- Examples
		1. Strategic
			 - What business units are at most risk to cyber crime?
				 - (Maybe Sales, customer facing departments with customer data)
		 2. Operational
			 - What activity groups are currently active in our industry?
				 - (Revil, Lazarus Group, many other thru IABs)
		 3. Tactical
			 - What adversary behaviors should security focus on to identify threats that are the most likely to breach our organization?
				 - (Phishing Campaigns, Adversary History, OSINT regarding adversary)
2. Threat Modeling
 ![[ThreatModeling.png]]
- Identify what threats or adversaries may target which department or what data of our organization.

3. Collection Management Framework
	 - Where are we getting the data? How is it processed? How is it delivered? What questions are we asking of the data?
	 - I.E. What requirements can we fulfill using these data?
	 - ==What is the ingestion and analysis pipeline of our threat intel operation?==
	 ![[SampleExternalCMFonMalwareData.png]]

## Collection
### Some Key Collection Sources
1. Intrusion Analysis
	1. Perform analysis on previous intrusions.
		![[Intrusion_Kill_Chain_-_v2 2.png]]
2. Malware
	1. Malware Collections / Samples
		- Analyze previous malware from certain adversaries etc
		- VirusTotal, Hybrid-Analysis, Joe Sandbox etc are malware sandboxes.
3. Domains
	1. Start
		 1. Find initial indicator thru WHOIS etc to find email etc
	2. Pivot
		 1. Find new info with info found by initial indicator.
		 2. Who used this email? etc
	3. Validate
		 1. Ensure these data points and investigations are meaningful
		 2. Microsoft address is not meaningful
4. TLS Certificates
	1. Censys.io, scans.io, Circl.lu, PassiveTotal
	2. Can be used to find C2 infrastructure

## Processing and Exploitation

