
`#maps2graphs` is a social project/factory for creating `#roadmap` graphs based on `#GoogleMapsAPI` -- example: "Tokyo Family Marts" Graph


# Summary, Disclaimers, FAQ

1. See **screenshots.pdf** to get the overall idea about the application
2. Read **Section: Example Run** for a quick walk through the procedures.
3. Read **Section: Implementation** for details about the software.
4. **Section: Terminology** is useful to get aquiainted with terms -- I use lots of acronyms and abbreviations.
5. Remember: you **do not need me** to start building your own dataset.  The webapp is a **standalone** HTML file which can be run anywhere -- that includes desktop computers and notebooks under any OS, tablet PCs, smartphones, etc.  So, as long as you can gather your own crowd, you can build your own dataset.  I would be nice if you share, though. 
6. Remember **the math**: `#GoogleMapsAPI` daily quota per IP is 2500 requests.  This app takes it easy and sends **one request per 45 seconds**.  Now, with 100 locations, it takes `0.5 * n * ( n - 1) ~= 5200` request to built the graph from pairwise requests -- that assuming that A-B and B-A routes are the same (not completely true but close enough).  If you do not know `n` but have access to the total request count `c`, then `n = 0.5 + sqrt( 0.25 + 2 * c)`.  I am sure that there is a mathematical way to identify the pair of locations from the sequence number of a request, but being unaware of it I use the brutal method -- simply go over the two loops `for ( i = 0; i < n - 1; i++) { for ( ii = i + 1; ii < n; ii++) { keep.track.of.pos(); }}` and catch your `seqno`.
7. Crowd or no crowd.  The **worst case** is that you have no crowd and you have to do it yourself.  However, like I do, you can run the client/user side from multiple places (home, work, smartphone, etc.) and thus decrease the time you spend aggregating the dataset.  Works for me. 


# Terminology 

1. **Dataset**: in the spirit of the `#maps2graphs` idea, based on the number of locations, using map APIs to create a graph (data structure in graph theory) connecting the locations.  That very **graph is the dataset**.

2. **Location**: GoogleMapsAPI is robust to find locations other other addresses.  Addresses are preferred, but are not required.  For examle, for the test run I used city names and everything worked smoothly. 

3. **Manager**: the primary contact for a given dataset.  Manager provides the list of locations.  

4. **Crowd**: a number of people which collaborate with the manager to create a specific dataset.  Each crowd member should have access to PUBLIC url.

5. **PURL**: public URL, created by the manager for the use by crowd members.

6. **PRURL**: private URL, used only by the manager.  When manager gets a piece of data from his/her crowd, the app provides aggregation routines which result in updated content for PURL and PRURL. 



# Overview and Justification

Say, we want to create a (road) graph connecting all Family Mart convinience stores (conbini) in Tokyo Area.  We can get addresses of all Family Marts at Family Mart website.  Me... I normally crawl the website automatically ... pace the application for slow (read:human-like) extraction, and get the list of addresses within an hour or so.  Now, how does one find out how these locations are connected by roads?   If you do not see the necessity for such datasets, do not bother to read further. 


We have **#GoogleMapsAPI**.  In **GIS** -- Geographic Information Systems -- we have 2d-mesh and 3d-mesh data. There is a third alternative -- to get data in whatever the format car navigators use, but a small inquiry into that opportunity came back with nothing.  Apparently, all car navigators use proprietary formats which are impossible to extract and convert into something meaningful ... like GPS coordinates akin to GoogleMapAPI coords.  If we could get that data, the objective of this entire project would be fulfilled completely.  This does not appear to be the case, through.  If someone knows otherwise, prove me wrong. 

2d- and 3d-mesh are really bad formats.  2d-mesh resolution is 10kms.  3d-mesh has 1km resolution and ... that's pretty much as far as GIS goes.   How many intersections and roads does one get within 1km-by-1km square in a moderately big city?  Very many.  

I actually walked down the GIS path.  I officially requested data from Tokyo Association of Roads, got their CD, went through its data and discovered that data's max resolution is that of 2d-mesh.  When I plotted a portion of Tokyo on a graph ... yes, good guess ... the entire plot was a single dot ... simply because all the roads and intersections ended up having the same 2d-mesh coordinates.    You can have +1 -1 variety within one city, but it is not enough to make a pretty plot.  You still get the correct graph, though, because each intersection has unique ID apart from its 2d-mesh coordinates, ... you just can't draw the graph. 

That leaves **GoogleMapsAPI as the only alternative**. The only problem is that access to GoogleMapsAPI is restricted by quotas.  I did the math and arrived at roughly "1 request per 36 seconds" limitation.   There are two quotas -- **(1)** short-term and **(2)** daily.  Since this project needs to run continuously, the access to API is paced by design, so we only care about the daily quota, which is 2500 requests per IP per day.  

That is in itself a justification for this project.  Say, we need a graph connecting 50 locations.  Assuming (often wrongly or by a big stretch) that all end-to-end routes are the same in both directions (A-to-B = B-to-A), we roughly need `0.5 * ( 50 ^ 2) = 1250` requests.  With 3-request-per-2min pacing, we can be done in 1250 / 3 = 400 minutes = roughly 7 hours. 

That's all the justification.  If I had 7 people working on that graph from 7 separate locations *(read: 7 independent quota counters)*, I could be done in 1 hour.   This is why I am undertaking this whole thing. 


# Implementation

I have coded a standalone webapp in **standalone.html**.  Everything you need is in that one HTML file, specifically:

1. user side and manager side, the former needs `public URL` **PURL** to work, the latter is should have access to both **PURL** and **PRURL**, as well as occassional access to pieces provided by the crowd for each specific dataset, which are also provided as URLs (shared by other people). 

2. User side goes through the routine of feeding the **PURL** to the app and letting it run its queries.  When done (or have to go, or worried that browser is about to quit on you), just click **ABORT** and use whatever the current data you have.  Data unit is JSON for the route for one pair of locations.  The order of requests is randomized to avoid too much overlap across crowd members.  Each chunk of data is shared via each user's GoogleDrive, where the output generated by the app is saved as plain text and shared via the URL to a publically shared GoogleDrive file. 

3. Manager should aggregate each chunk of data into its private data structure.  Frequent aggregation also helps by moving the finished requests out of scope so that users do not have to run queires on them again.  For now, the app does the aggregation (in the browser), but I will provide a PHP script to do it automatically later on. 

4. Data sturcture. Private file is array where the first element is a string representing the status of each unique combination of locations. `0` means data is yet to be collected, `1` means that the data at that position is present in the private file.  The rest of the public file is the list of locations, each location stored as base64( string) to make it pleasing to the eye.  Private file is also array with length equal to the number of combinations of requests.  Initial state is `?` in each item.  As data is aggregated into the file, more and more entries will have meaninful data.  Note that private file will keep growing in side, while public file has a constant size. 


# Social Collaboration 

From the **justification** above, the necessity for **crowdsourcing** should be clear.  However, there are good side effects/benefits.  Here is the list of direct and indirect benefits, with some features mixed in.

1. Direct benefit.  One gets to save time extracting the graph from GoogleMapsAPI.  The more people, the shorting the waiting time. 

2. Feature.  One does not have to sweat too much to participate.  Simply click on a small URL from the **CFH** (call for help).  And keep the tab running in your browser for a while.   When you need to close the browser or cannot help for some other reason, click **DONE** in the webapp, copy-paste the base64 gibberish that comes out into a plaintext file, make it publically available on your GoogleDrive and post the report on Twitter with the hashtag of your current **#maps2graphs4????** where ???? is announced in the original CFH. 

3. Direct benefit.  All changes are announced on Twitter, so, everyone gets to share the data.  The more social participation, the faster we all get the data.  Once the graph is there, it can stay there.  Or you can download and store it at your place.  I might build a repository in the future... say at GitHub.  

4. Indirect benefit.   We can learn of new uses for #maps2graphs datasets.  I plan to use it for ITS research. Specifically, I am working on electric cards, car battery replacement, etc.  You can find your own uses for the same data.  When you do, please, annouce with #maps2graphs hashtag on Twitter.


# Example Run/Cycle/Map/Graph/Project

This is simply walking you through the one hypothetical run. 

* **manager** selects `maker/new`, pastes a list of locations and generates public and private files
* **manager** puts the files into some location in his/her GoogleDrive and shares them, the urls are converted at [gdriveurl][http://www.gdriveurl.com/].  I could figure out how to negotiate with GoogleDrive directly myself, but it seems unnecessary when there is a service.  Maybe in the future.  *WARNING: you cannot feed raw GoogleDrive share links to the app, they do not return the files by some field html structures containing them. gdriveurl fixes that very problem.*
* **manager** shares the PURL with the crowd.  Preferrably on Twitter.  Otherwise, whatever mode of communication you use. 
* **user** goes to `user` in the app, picks time budget as `10^x requests`, and feeds PURL to the app, the app then starts sending the requests.  You can see the status in the panel.  You can leave the application to run in the background (not in Android, though), and do something else. 
* when **user** needs to go or use the browser for something else, he/she clicks **ABORT**, where the app generates JSON for current data and makes it available for copy/paste.  
* **User** should copy the contents into a plaintext file, put it into somewhere in his/her GoogleDrive and share the file. Again, the URL should be wrapped by  [gdriveurl][http://www.gdriveurl.com/].  Post the link on Twitter. The app actually generates the text for you in which you only have to insert/replace URL and the edit the `#maps2graphs4???` with the one for your dataset
* **manager** goes to `maker/existing/add results` in the app, feeds it with PRURL, PURL and the URL for the provided chunk of data, and lets the application merge the data.  The output is the new private and public files which **manager** should use to overwrite earier versions. PURL and PRURL do not have to be updated as long as you use the same file names and do not move the files to another folder. 
* when `maker/status` shows that 100% of the dataset is ready, you can start using it. The structure is simple enough and is described in **Section: Implementation**.  Do not forget that in each item of the private file is base64( JSON string).  Just extract one to see the internal structure -- it contains all the legs of the route as provided by the GoogleMapsAPI.  All location and places are base64()-ed as well, again, to make them pretty. 


# Coding and Activity Log

* *20130508* completed the application, running the testrun, everything seems to work smoothly





