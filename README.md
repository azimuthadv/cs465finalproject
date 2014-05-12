CVE Visualizer
===============
A final project for the Spring 2014 CS465 (Information Visualization) class at Middlebury College.


Authors
-------
[Daniel Trauner ('14)](http://www.danieltrauner.com) and [Arturo Vidal ('15)](http://www.arturovidal.org)  


Requirements
------------
In order to deploy this project, you will need both the HTML/Javascript (D3) frontend file "index.html" as well as all of the backend files (based on wimremes' [cve-search](https://github.com/wimremes/cve-search) project) within the "backend_*.zip" archive.  While we used the smallest ($5/month) instance on DigitalOcean running Ubuntu 14.04 x64 and set up the backend to serve API requests publicly to a different machine hosting the frontend page, you could theoretically run the API and frontend off of the same machine (whether remote or local).  

**First, in order to set up the backend:**  
1. Install Python 3  
2. Install MongoDB  
3. Install the following Python 3 libraries: ```pymongo```, ```argparse```, ```flask```, ```flask-restful```, and ```flask-pymongo```  
4. Extract the "backend_*.zip" file to your home directory  
5. Populate the cvedb instance with CVE and CPE data by running ```python3 ~/cve-search/db_mgmt.py -p``` and then ```python3 ~/cve-search/db_mgmt_cpe_dictionnary.py``` (this may take a while, but only has to be done once)  
6. Set up automatic updating of the populated cvedb instance via ```cron``` (create a cron event that executes ```python3 ~/cve-search/db_updater.py -v``` from the cve-search folder on regular intervals)  
7. Run the "rest-server.py" script in the cve-search folder to start the API server  

**Next, in order to set up the frontend:**  
1. Replace all occurrences of "162.243.5.105" in the "index.html" file with the address of the server you installed the backend on (or 127.0.0.1 if the backend is running on the same machine that is serving the "index.html" file)  
2. Use your favorite webserver application to host the index.html file

Once the backend is properly set up and the references to the backend are correct within the frontend index.htm file, the visualization should load.  Note that it takes a few seconds to load the (thousands) of unique vendors within the drop down selector.  Be patient :)  


Data Sources
------------
The main CVE and CPE data feeds are from the National Vulnerabilities Database (NVD), which provides this information in the XML format.  But our backend is based heavily on wimremes' [cve-search](https://github.com/wimremes/cve-search) project, designed to dump this XML data (and similar/related data from a number of other sources) into a MongoDB instance for easier management.  In particular, given the large amount of data (and the importance of keeping the database up-to-date), the ability to perform incremental updates is essential.


Data Transformation
-------------------
We used the data as it was stored by the original cve-search project, but added the additional functionality of a flask-based read-only JSON API for easier querying of small portions of the entire data set for use with D3.  The API itself has a number of endpoints (not all of which are currently being used):

* ```/cves/<string:cpe_str>```: returns all CVEs affecting the given CPE string (partial CPEs such as product or vendors names may work here, but it is advisable to be as specific as possible)
* ```/cve/<string:cve_id>```: returns the CVE corresponding to the given CVE ID
* ```/cpes```: returns all unique CPE strings
* ```/vendors```: returns all unique vendors


Who is it for?  What questions does it answer?
----------------------------------------------
**This visualization is meant to serve two main audiences:**  

* A system administrator (especially at the enterprise scale) who wishes to install a particular type of product and has a number of candidates in mind after taking into account external factors such as budget and features.
* A curious individual or researcher wanting to gain insight into the vulnerability history of particular vendors and products (especially within a particular time interval).

**This visualization is meant to answer the following:**  

* For a given vendor, how many unique vulnerabilities affected all of their products monthly (across all-time or a specified window)?
* For a given product, how many unique vulnerabilities affected all of its versions monthly (across all-time or a specified window)?
* Which products for a particular vendor seem to be targeted the most in terms of the number of vulnerabilities affecting those products?
* For all of the vulnerabilities depicted in each of the above scenarios, how severe were they overall (CVSS base score) and what specifically do they affect (description)?

Description
-----------
**Initial Idea**  
Our original paper sketches (I didn't make a copy so I may be a little off here) were meant to illustrate a visualization that provides a hierarchical view of CVE data through distinct scatterplots of the same type (brushable time x-axis, vendors, products, or versions on y-axis, and scatterplot points for each vulnerability).  But right off of the bat after some initial exploration of the data, we realized that a "true" overview of all vendors would take too long to load given the massive amount of data (and would physically result in a need to scroll far down the page to see the entire plot).  In thinking about the audience for our visualization, we realized that restricting the user to one vendor at a time does make some sense despite the sacrifice of extra context.  This is especially true given that at the enterprise level many products from different vendors don't integrate well with each other (i.e. Microsoft and Apple products) so a system administrator would be more likely to care about the vulnerability statistics of individual vendors as part of their effort to mix-and-match as little as possible.

Furthermore, we had illustrated a "CVE" page for each individual vulnerability.  In practice this was too distracting and involved more navigation that we wanted to include.  Given that the benefit of including separate CVE pages would have not been worth the cost, we decided to use a hover-over tooltip for the CVE ID, "Published" date, description, and CVSS score.  This more rapidly conveys the same information.

Lastly, instead of a search page, we realized it would be easier (and cause fewer problems) to simply provide the user with a dropdown menu of all of the current vendors.

**Initial Versions**  
Other than leaving off the true "vendor-level" overview and search page as well as the separate CVE page in favor of a static tooltip, we didn't make too many changes to the initial idea.  Most notably, we initially tried using a tooltip that followed the mouse on hover over any scatterplot point; however, attempting to calculate all of the different variants on where the tooltip needed to be drawn proved difficult given the large variation in length of the "description" field for each vulnerability.

**Final Version**  
Upon loading the current version of the "index.html" page, the user is presented with a dropdown menu in the top left corner as well as instruction text.  Selecting an individual vendor brings up a loading spinner as all vulnerabilities for all products of the chosen vendor are loaded; once this process is complete the user is presented with a scatterplot.

The plot itself is based on the general idea behind Mike Bostock's D3 [Focus+Context via Brushing](http://bl.ocks.org/mbostock/1667367) example.  In our case, the (context) histogram above the scatterplot shows the number of unique vulnerabilities per month across the entire time span of all data points plotted in the actual scatterplot (focus).  The histogram also includes brushing functionality that allows the user to select a particular time interval - an important feature given the large number of partially overlapping vulnerability points for certain popular products that can look too crowded when viewed all at once.  While the top histogram's axis' scale remains fixed, on brushing, the axis at the bottom of the screen has its scale modified accordingly to reflect the span of time selected via the brush.

Once a user has selected a particular time interval, the focus region will only contain scatterplot points for those vulnerabilities that were published within that time interval.  Each point has a fixed radius and is color-coded with a (reversed) Color Brewer Red-Yellow-Blue scale based on its CVSS base score representing overall severity (from 0.0 to 10.0), and represents an individual vulnerability's affecting a particular product.  On hover over an individual point, all points representing the same vulnerability (same CVE ID) are made 100% opaque and outlined in black and have guide lines drawn from their left side towards the y-axis.  Additionally, hovering over an individual point replaces the instructions and vendor selector above the entire chart with a fixed tooltip that displays CVE ID, "Published" date, a description, and the exact CVSS score (with a background color identical to that of the hovered-over point).

Finally, clicking on the name of a product on the y-axis transitions the scatterplot from a view of all products by one vendor to that of all versions of one product.  While not everyone uses the same versioning system, the versions view gives the user a rough idea of how many vulnerabilities affecting the given product were version-specific.  One can return to the "all products by vendor" view by clicking on any of the y-axis labels.

Overall, we tried to keep in mind a number of Gestalt principles and other "cultural associations" that might give the average user a more rapid understanding of the data.  Aside from the use of position and continuity (through guides), the use of a Red-Yellow-Blue scale for CVSS score evokes a "hot-warm-cold" or "critical-medium-low" association that makes it easy to see not only the frequency of vulnerabilities overall, but also a rough breakdown of how many low, medium, and critical risk vulnerabilities affect a particular product.  The use of focus and context via brushing (brushing on histogram to affect scatterplot) as well as details on demand (tooltip) also serve to provide the user with lots of context information that is certainly absent from what the National Vulnerability (NVD) provides.

Final Assessment
----------------
**What (hopefully) works**  
* Focus+context via brushing (overall layout)
* Histogram frequency of vulnerabilities by time
* Bottom axis modified on brushing while top axis remains fixed
* Points near edges are not clipped by default
* Scatterplot view with color-coded dots bringing up guides, highlighting, and tooltip information on hover
* Redrawing of chart based on a single product's versions on click of the product's label on the y-axis (and a return to the products by vendor view on click of any version label)

**What doesn't work**  
* Display on certain size screens (there are a number of hard-coded values we slaved over trying to replace with more reasonable solutions but didn't succeed in doing in the end for many elements)
* Issue with using arrow keys to navigate the dropdown (chart is not removed)
* Many vendors have no products (currently warns you with an alert)
* Clicking on the y-axis labels in product view does transition to the version view, but if the product has no versioning for some reason, there are no labels to click (and so the user is stuck unable to return to the product view)

**What we'd like to do better**  
* Y-axis expansion on click to leave context of other products when bringing up one product's versions (we tried, but it was too difficult given the time contstraints)
* Clean up the data more on the backend to remove vendors with no products, for example
* Add a few extra bits of information contained in the XML files cve-search reads from but doesn't store
* Find a way to integrate the "refernece" links included with each vulnerability
* Better overall formatting and placement of information
