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

1. Structered Models: Data into Buckets (AWN literally uses S3 buckets to separate data)
	- Some examples: Kill Chain, Diamond Model, MITRE ATT&CK, VERIS
	- Diamond Model ([TTP]) ([Diamond Model White Paper](https://www.activeresponse.org/wp-content/uploads/2013/07/diamond.pdf))
		- ![[TheDiamondModel.png]]
	- [MITRE ATT&CK](https://attack.mitre.org/matrices/enterprise/)
		- documentation of tactics and techniques (organized like OWASP top 10)

## Analysis and Production

### Bias.
- Everyone has it. Even you. 
- Examples (confirmation bias)
	- Selectively support one hypothesis
		- May trigger: 
			- Evidence inclusion bias, seeking supporting evidence only
			- Significance Bias, greater significance to supporting data only
### Leveraging Different Types of Analysis
1. Know yourself
	1. What is my favorite type of analysis for given situations?
	2. What analysis types facilitate my process of analysis?
2. Know the team
	1. What are my team members' analysis types?
3. Inject new approaches
	1. Try new types of analysis, especially on critical cases
	2. Do NOT leverage only a single type of analysis

## Dissemination 
- the action or fact of spreading something, especially information, widely

1. Know your audience
	1. Different audiences have different intel needs.
		 - non-technical? maybe tactic is enough
		 - semi? techinique?
		 - Industry professionals? full TTP?
	2. Can be summarized:
		- Strategic - least technical
		- Operational
		- Tactical - most technical
	3. Tips on Effective Report Writing
		1. Tell an Honest Story
		2. BLUF - Bottom Line Up Front (begin a messge with key)
		3. Request Action
		4. Metrics That Matter
		5. Bring It to Life
		6. Give Credit
		7. Technical Appendixes