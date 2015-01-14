This is an application to retrospectively visualize the flow of patients through the hospital campus, portrayed in a network of edges and nodes using a D3.js force directed graph 

  
![alt tag](https://github.com/ajkou/Ward-Path-Application/blob/master/D3Image.png)

	A majority of but not all paths lead back to the community via normal hospital inpatient discharge. 
	Other trails quickly leadback to hospitalization, remain in long term care facilities, or 
	conclude with an inpatient record of death.

The disposition of transfers was determined by the entire data catalog of the VAPAHCS Patient Transfer File (PTF), which is the official hospital record of all patient admissions, inter-ward transfers, and inpatient discharges from the Palo Alto campus. This data represents continuous documentation of a patient's location from the time a patient enters the hospital to the hour at which he/she goes home. The PTF also contains similar recordings marked for administrative census, records about fee-basis encounters, and documentation of mortality.

Information about length of stay was derived by querying tabular records in VistA table PTF_45 and table P_45$535_535, which contain every patient’s ward ward discharge recordings and the timestamp. Time series calculations yeilded the length of stay at any given ward by looking at preceding discharges and doing a comparison with the last available timestamp.

This data was stored in relational format within a MS Access 2010 database and re-queried locally using standard SQL. MS Access SQL (probably a simplified version of SQL-92) does not include many common set operations. Therefore, to deal with contingencies like conflicting records or patients in limbo (never discharged), a series of subqueries were created in this project. These views contained the logic that either repaired or eliminated confounding records. 

A number of decisions were made in subqueries regarding the interpretation of transfer records available in the PTF. 
Conflicts within the records and confounding facets of the data were resolved as follows,

	•	Transfer records in P_45$535_535 were deduplicated
	•	Census records were not included in dataset
	•	Self-transfers were removed from the dataset (transferred to and from the same ward)
	•	When multiple transfer records disagree, the transfer conferring the shorter interval in ward X was taken as fact
	•	Transfer records were prioritized over discharge records when they had identical timestamps
	•	Documentation of death was treated as the final discharge record. Any transfers found after a date of death were ignored
	•	The first known ward for any patient was assumed to come from the community (although a patient could have come from another hospital system, the ER, or abroad)
	•	Discharges to fee-basis consultations were treated and calculated like a transfer

To transform this data into a web format, a one-way conversion of tabular data to JSON was required. After aggregating tuples of transfer records to ward-to-ward counts, a data converter was built on the RtoJSON function (Package 'rjson'). RtoJSON uses recurive implementation that constructs the heirarchical structure of JSON data via series of string concatenations. The problem with this function was that it also converts all numerics to strings and litters the JSON with double quotation marks, so an integer handler had to written into the function. 

Partway through development, I was temporarily convinced that a web enabled graphical network contained too much technical overhead for a simple operations research problem. I attempted a more immediate visualization technique using the NodeXL Excel Template (Social Media Research Foundation), a packaged MS Office tool to visualize a network in the social sciences. 

Paths passing through the Medical ICU
![alt tag](https://github.com/ajkou/Ward-Path-Application/blob/master/GraphImage%20MSICU.png)

Paths passing through the Surgical ICU
![alt tag](https://github.com/ajkou/Ward-Path-Application/blob/master/GraphImage%20SMICU.png)

However, I found this visualization to be inadequeate for a problem of this scope with >60 nodes and high edge density/connectivity, although excessively complex networks were a consistent problem in both visualization attempts. In both situations, techniques to deal with the high complexity included:
 
	Filters and drills to subset the JSON data.
	
	Visual tricks to spread out nodes or selectively render relevant nodes by relevancy.

Additional developement includes:

	Directed graph interactivity for drill down and highlights
	
	Joining the data management process into a unified workflow
