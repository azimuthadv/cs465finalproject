CVE Visualizer
===============
A final project for the Spring 2014 CS465 (Information Visualization) class at Middlebury College -- a visualizer for CVE vulnerabilities.


Who worked on the project
--------------------------
[Daniel Trauner ('14)](www.danieltrauner.com) and [Arturo Vidal ('15)](www.arturovidal.org)


How to deploy the project
-------------------------
In order to deploy this project, you will need both the HTML/Javascript (D3) frontend file "index.html" as well as all of the backend files (based on wimremes' [cve-search](https://github.com/wimremes/cve-search) project) within the "backend_*.zip" archive.  While we used the smallest ($5/month) instance on DigitalOcean running Ubuntu 14.04 x64 and set up the backend to serve API requests publicly to a different machine hosting the frontend page, you could theoretically run the API and frontend off of the same machine (whether remote or local).

First, in order to set up the backend:
	1. Install Python 3
	2. Install MongoDB
	3. Install the following Python 3 libraries
		..* pymongo
		..* argparse
		..* flask
		..* flask-restful
		..* flask-pymongo
	4. Extract the "backend_*.zip" file to your home directory
	5. Populate the cvedb instance with CVE and CPE data by running ```python3 ~/cve-search/db_mgmt.py -p``` and then ```python3 ~/cve-search/db_mgmt_cpe_dictionnary.py``` (this may take a while, but only has to be done once)
	6. Set up automatic updating of the populated cvedb instance via ```cron``` (create a cron event that executes ```python3 ~/cve-search/db_updater.py -v``` from the cve-search folder on regular intervals)
	7. Run the "rest-server.py" script in the cve-search folder to start the API server

Next, in order to set up the frontend:
	1. Replace all occurrences of "162.243.5.105" with the address of the server you installed the backend on (or 127.0.0.1 if the backend is running on the same machine that is serving the frontend index.html file)
	2. Use your favorite webserver application to host the index.html file

Once the backend is properly set up and the references to the backend are correct within the frontend index.htm file, the visualization should load.  Note that it takes a few seconds to load the (thousands) of unique vendors within the drop down selector.  Be patient :)


Where the data came from
------------------------
The main CVE and CPE data feeds are from the National Vulnerabilities Database (NVD), which provides this information in the XML format.  But our backend is based heavily on wimremes' [cve-search](https://github.com/wimremes/cve-search) project, designed to dump this XML data (and similar/related data from a number of other sources) into a MongoDB instance for easier management.  In particular, given the large amount of data (and the importance of keeping the database up-to-date), the ability to perform incremental updates is essential.


What you did to transform the data to a usable state
----------------------------------------------------
We used the data as it was stored by the original cve-search project, but added the additional functionality of a flask-based read-only JSON API for easier querying of small portions of the entire data set for use with D3.  The API itself has a number of endpoints (not all of which are currently being used):

	* ```/cves/<string:cpe_str>``` -- returns all CVEs affecting the given CPE string (partial CPEs such as product or vendors names may work here, but it is advisable to be as specific as possible)
	* ```/cve/<string:cve_id>``` -- returns the CVE corresponding to the given CVE ID
	* ```/cpes``` -- returns all unique CPE strings
	* ```/vendors``` -- returns all unique vendors


A description of who the visualization is for and what questions it is designed to answer
-----------------------------------------------------------------------------------------
This visualization is meant to serve two main audiences:
	* A system administrator (especially at the enterprise scale) who wishes to install a particular type of product and has a number of candidates in mind after taking into account external factors such as budget and features.  By exploring how many vulnerabilities each candidate product has had historically (as well as the severity and descriptions of each of these vulnerabilities), he or she can make an informed decision about whether or not a given candidate product may be appropriate for a particular application.
	* A curious individual or researcher wanting to gain insight into the vulnerability history of particular vendors and products (especially within a particular time interval).

A description of the visualization including rational for the encodings and interactions you chose. A description of any approaches you contemplated or tried and then rejects would be appropriate as well
---------------------------------------------------------------------------------------------------------------------------------------------------------


An assessment of the success of your final product. What works, what doesn’t work, what changes would you make if you were to do this again.
--------------------------------------------------------------------------------------------------------------------------------------------
