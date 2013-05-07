
`#maps2graphs` is a social project/factory for creating `#roadmap` graphs based on `#GoogleMapsAPI` -- example: "Tokyo Family Marts" Graph


# Overview and Justification

Say, we want to create a (road) graph connecting all Family Mart convinience stores (conbini) in Tokyo Area.  We can get addresses of all Family Marts at Family Mart website.  Me... I normally crawl the website automatically ... pace the application for slow (read:human-like) extraction, and get the list of addresses within an hour or so.  Now, how does one find out how these locations are connected by roads?   If you do not see the necessity for such datasets, do not bother to read further. 


We have **#GoogleMapsAPI**.  In **GIS** -- Geographic Information Systems -- we have 2d-mesh and 3d-mesh data. There is a third alternative -- to get data in whatever the format car navigators use, but a small inquiry into that opportunity came back with nothing.  Apparently, all car navigators use proprietary formats which are impossible to extract and convert into something meaningful ... like GPS coordinates akin to GoogleMapAPI coords.  If we could get that data, the objective of this entire project would be fulfilled completely.  This does not appear to be the case, through.  If someone knows otherwise, prove me wrong. 

2d- and 3d-mesh are really bad formats.  2d-mesh resolution is 10kms.  3d-mesh has 1km resolution and ... that's pretty much as far as GIS goes.   How many intersections and roads does one get within 1km-by-1km square in a moderately big city?  Very many.  

I actually walked down the GIS path.  I officially requested data from Tokyo Association of Roads, got their CD, went through its data and discovered that data's max resolution is that of 2d-mesh.  When I plotted a portion of Tokyo on a graph ... yes, good guess ... the entire plot was a single dot ... simply because all the roads and intersections ended up having the same 2d-mesh coordinates.    You can have +1 -1 variety within one city, but it is not enough to make a pretty plot.  You still get the correct graph, though, because each intersection has unique ID apart from its 2d-mesh coordinates, ... you just can't draw the graph. 

That leaves **GoogleMapsAPI as the only alternative**. The only problem is that access to GoogleMapsAPI is restricted by quotas.  I did the math and arrived at roughly "1 request per 36 seconds" limitation.   There are two quotas -- **(1)** short-term and **(2)** daily.  Since this project needs to run continuously, the access to API is paced by design, so we only care about the daily quota, which is 250 requests per day.  

That is in itself a justification for this project.  Say, we need a graph connecting 50 locations.  Assuming (often wrongly or by a big stretch) that all end-to-end routes are the same in both directions (A-to-B = B-to-A), we roughly need `0.5 * ( 50 ^ 2) = 1250` requests.  With 3-request-per-2min pacing, we can be done in 1250 / 3 = 400 minutes = roughly 7 hours. 

That's all the justification.  If I had 7 people working on that graph from 7 separate locations *(read: 7 independent quota counters)*, I could be done in 1 hour.   This is why I am undertaking this whole thing. 


# Implementation

I have coded a standalone webapp in **standalone.html**.  

More details later.


# Social Collaboration 

From the **justification** above, the necessity for **crowdsourcing** should be clear.  However, there are good side effects/benefits.  Here is the list of direct and indirect benefits, with some features mixed in.

1. Direct benefit.  One gets to save time extracting the graph from GoogleMapsAPI.  The more people, the shorting the waiting time. 

2. Feature.  One does not have to sweat too much to participate.  Simply click on a small URL from the **CFH** (call for help).  And keep the tab running in your browser for a while.   When you need to close the browser or cannot help for some other reason, click **DONE** in the webapp, copy-paste the base64 gibberish that comes out into a plaintext file, make it publically available on your GoogleDrive and post the report on Twitter with the hashtag of your current **#maps2graphs4????** where ???? is announced in the original CFH. 

3. Direct benefit.  All changes are announced on Twitter, so, everyone gets to share the data.  The more social participation, the faster we all get the data.  Once the graph is there, it can stay there.  Or you can download and store it at your place.  I might build a repository in the future... say at GitHub.  

4. Indirect benefit.   We can learn of new uses for #maps2graphs datasets.  I plan to use it for ITS research. Specifically, I am working on electric cards, car battery replacement, etc.  You can find your own uses for the same data.  When you do, please, annouce with #maps2graphs hashtag on Twitter.


# Example Run/Cycle/Map/Graph/Project

More details later.



