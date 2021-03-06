---
layout: post
title: Craigslist Indexer Addendum
---

I should add something to the Craigslist Indexer [post below][1], because I just pushed some changes today that were glossed over in my previous post, and I think it's something that I (and perhaps others?) seem to do regularly.

[1]: /2013/01/07/cl_indexer/ "Craigslist Indexer"

The setup:  I have a function that will be passed in a string, and I want it to be able to return a list of the neighborhood names that appear in that string, whether it be a singleton list or a three item list.

I originally wrote the scraper with the neighborhood names that I was searching for in a list.  That is, I had a list of ['DUMBO', 'Fort Greene'...].  This worked fine, and with the help of Hacker School, I had an awesomely short list comprehension that did ran through the list, checked the string for both uppercase and lowercase versions of the string, and returned a list of the names that appear.  This was the whole function:

```
    def find_nabe(item):
        return [x for x in NEIGHBORHOOD_LIST if any(item.find(y) != -1 for y in [x, x.lower(), x.upper()])]
```

But the problem, which I mentioned below, is that there are a lot of misspellings on Craigslist and a lot of different ways to refer to a neighborhood (i.e. Bed-Stuy, BedStuy, Bedford-Stuyvesant).  So I made a dictionary that has one key spelling for each neighborhood mapped to all of the possible spellings I want to look for.  I only want to store the key in my database, so I need the function to return the key when it finds one of the values.  I still need the function to return a list, even if it is a singleton list, to keep the ability to have a one Craigslist listing be associated with multiple neighborhoods in my database.

It's not that difficult of a problem, but I struggled with it for a day or so because I was trying to replicate something slick like the list comprehension above.  But a friend of mine pointed out that the length of the list comprehension was getting long, and it was actually much easier to understand simple nested loops.  So, I just broke part of the looping out to it's own function, and came up with this (keep in mind NEIGHBORHOOD_LIST is now a dictionary, and it was a list above):

```
    def find_nabe(item):
        nabes = []
        for k, v in NEIGHBORHOOD_LIST.items():
            newnabe = (check_spellings(item, k, v))
            if newnabe: nabes.append(newnabe)
        return nabes


    def check_spellings(item, k, v):
            nabes = []
            for value in v:
                if any(item.find(y) != -1 for y in [value, value.lower(), value.upper()]):
                    return k
```

It's eleven lines of code where I used to have two, but it's probably a lot easier to understand and debug than trying to pack this all into a list comprehension.  And speed isn't really an issue here...This is just a script to populate the database.

I think I was falling into a trap of trying to replicate the one liner for the simple reason of writing slick code.  I've found myself doing this a few times, especially as a I try to make my programs more functional in style.  But I'm gradually coming to the realization that 1) I can always go back and refactor this into a more functional style if I want, and 2) functional style isn't always the best or easiest to understand.
