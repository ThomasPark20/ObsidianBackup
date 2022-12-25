# Diamond Model
tags: #books
source : [Whitepaper](https://www.activeresponse.org/wp-content/uploads/2013/07/diamond.pdf)
authors: Sergio Caltagirone, Andrew Pendergast, Christopher Betz (all Center for Cyber Intelligence Analysis and Threat Research)

---

 ### Simple model
 
![[SimpleDiamond.png]]

##### Axiom 1
- For every intrusion **event** there exists an ==adversary== taking a step towards an intended goal by using a ==capability== over ==infrastructure== against a ==victim== to produce a result.

##### ==Event==
- discrete time-bound activity restricted to a specific phase where...
	- an *adversary*, requiring external *resources*, uses a *capability* and *methodology* over some *infrastructure* against a *victim* with a given *result*
- ==Core Features==
	- adversary
		- ==operator== -> hacker
		- ==customer== -> entity that benefits from activity
	- capability
		- ==capacity== -> all the vulns and exposures that can be utlized regardless of victim
		- ==adversary arsenal== -> adversary's complete set of capabilities
	- infrastructure
		- physical and/or logical comm structures the adversary uses to deliver and maintain the effects of capabilities
		- ==Type 1== -> fully controlled or owned by adversary
		- ==Type 2== -> "middle" infrastructure (typically what victims see)
			- Zombies, hosting servers, hacked emails etc.
		- ==Service Providers== -> ISP, domain registrars, mail providers etc
	- victim
		- ==Persona== -> people and orgs targetted (names, jobs, industries, interests, etc)
		- ==Asset== -> attack surface, set of tech under victim control.
- ==Meta-Features== 
	- used to order events with an *activity thread*
	- timestamp (start and end)
	- phase
		- ==Axiom 4== 
			- Every malicious activity contains ==two or more== phases which must be successfully executed in succession to achieve the desired result.
	- result
	- direction
		- from who to who? (adversary to victim?) (victim to adversary?)
	- methodology
		- syn flood? phishing campaign?
	- resources
		- Software (metasploit, etc)
		- Knowledge (where to get exploits, etc)
		- Information (username/password, etc)
		- Hardware
		- Funds
		- Facilities
		- Access (network path?)
- ==Confidence Value==
	- applies to all features
	- likeliness suite
		- confidence of analytic conclusion
		- accuracy of data source
![[event.png]]
Example Event n-tuple.
![[victim.png]]
Example detailed Victim n-tuple.

