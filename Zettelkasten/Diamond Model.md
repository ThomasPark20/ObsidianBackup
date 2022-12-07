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
	- capability
	- infrastructure
	- victim
- ==Meta-Features== 
	- used to order events with an *activity thread*
	- timestamp (start and end)
	- phase
	- result
	- direction
	- methodology
	- resources
- ==Confidence Value==
	- applies to all features
	- likeliness suite
		- confidence of analytic conclusion
		- accuracy of data source
![[event.png]]
Example Event n-tuple.
![[victim.png]]
Example detailed Victim n-tuple.

