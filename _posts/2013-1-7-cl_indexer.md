---
layout: post
title: Craigslist Indexer
---

Craigslist Indexer
=====

My first larger project at Hacker School was [Craigslist Indexer][1], which is an app to scrape the Craigslist housing listings.  With a lot of other websites, this wouldn't be a 2-3 week project, but Craigslist has no API and a poorly formatted RSS feed, so aggregation isn't easy.  Here is a sample listing from the Craigslist feed:

[1]: http://github.com/swoopej/cl_indexer "Craigslist Indexer"

```
    <item rdf:about="http://newyork.craigslist.org/brk/abo/3396723906.html">
    <title><![CDATA[* HOT 3 BR * NEW* CLOSE TO J M Z TRAINS* GREAT BLOCK * E. WILLIAMSBURG (E. WilliamsBurg / Bushwick JMZ ) $2200 3bd]]></title>
    <link>http://newyork.craigslist.org/brk/abo/3396723906.html</link>
    <description><![CDATA[Brand new apartment ! Will be finished by DEC 1st 
    Close by trains J M Z 
    Beautiful tree line block. As well it is quiet! 
    WELL MAINTAINED building ... Kept CLEAN and KEPT at all times !!!! 
    The APARTMENT has nice high ceilings *** 
    Large Closets ..  [...]]]></description>
    <dc:date>2012-11-26T12:22:02-05:00</dc:date>
    <dc:language>en-us</dc:language>
    <dc:rights>Copyright &#x26;copy; 2012 craigslist, inc.</dc:rights>
    <dc:source>http://newyork.craigslist.org/brk/abo/3396723906.html</dc:source>
    <dc:title><![CDATA[* HOT 3 BR * NEW* CLOSE TO J M Z TRAINS* GREAT BLOCK * E. WILLIAMSBURG (E. WilliamsBurg / Bushwick JMZ ) $2200 3bd]]></dc:title>
    <dc:type>text</dc:type>
    <dcterms:issued>2012-11-26T12:22:02-05:00</dcterms:issued>
    </item>
```

Most of the information that I wanted, i.e. neighborhood, price, and bedrooms, are actually contained in the same title tag, with other delineation to separate them.  The only consistent structures are 1) parentheses around the neighborhood, 2) a dollar sign before the price, and 3) the bedrooms are a digit followed by "bd".

Fortunately, the neighborhood, price, and bedrooms appear in that order, so I did a couple of things.  Before looking for any of the key data, I got the string from within the title tag and found the last occurrence of an open parentheses.  Craigslist allows you to put all the parentheses you want in the user created title, but the last open paren will be the one containing the neighborhood.

The neighborhood description can contain dollar signs, however, so I again had to use Python's rfind method to find the last dollar sign in the already shortened string.I used this index to find both the price and the bedroom, and toss the listing if the correct information isn't found.

The other issue with Craigslist is that almost all of the data is user created, so there is no uniform way of delineating neighborhoods or even spelling certain neighborhoods.  A lot of landlords will fudge the boundaries of neighborhoods by saying that a listing is "Prospect Heights/Crown Heights" when the actual location is the other edge of Crown Heights near Brownsville.  

I can't help the fact that some landlords will outright lie about location, because those people often won't include exact addresses or map links, but I did account for hybrid neighborhood listings that include two neighborhoods.  I did this by separating out neighborhoods into its own database table and having a listing-to-neighborhood table that has relations between listings and neighborhoods.  If more than one neighborhood is found in the title listing, multiple relations are inserted into the table.

To solve the problem of spelling different neighborhoods, I just used a dictionary with correct spellings as the key and incorrect spellings as the values.  My parser looks for all the values in the Craigslist data and if a match is found, the key for that value is used for the database entry.  

After some refactoring with Zach, one of our facilitators at Hacker School, I was able to get a decently robust parser for the messy data.  I built a fairly simple front end using Flask that just returns JSON, but I'm hoping to add some visualization using Javascript soon.
