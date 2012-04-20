---
layout: post
category : Programming
tags : [ASCII art, PHP, asynchronous, cURL]
title: ASCII Art Signatures In The Wild
---
{% include JB/setup %}

<a href="https://github.com/geon/asciiart"><img style="position: absolute; top: 0; right: 0; border: 0;" src="https://a248.e.akamai.net/assets.github.com/img/30f550e0d38ceb6ef5b81500c64d970b7fb0f028/687474703a2f2f73332e616d617a6f6e6177732e636f6d2f6769746875622f726962626f6e732f666f726b6d655f72696768745f6f72616e67655f6666373630302e706e67" alt="Fork me on GitHub"></a>

**or: How I Downloaded The Internet And Learned How Much One Million Is**

As a web developer/designer (amongst other things), I take pride in my craftsmanship and like to sign my work with an [ASCII art](http://en.wikipedia.org/wiki/ASCII_art) signature in an HTML comment. I know I'm not the only one, since I was inspired by seeing others do it.

So, I had this idea: Wouldn't it be awesome to scan through a chunk of the Internet and see who else is doing the same thing, and how their signatures look? 

Just getting a large list of websites turns out to be tricky. I first intended to use one of the large website catalogs, but I couldn't find on in an accessible format. I have only found one good resource, the [Alexa one million top domains list](http://s3.amazonaws.com/alexa-static/top-1m.csv.zip). 

Downloading In Parallel
-----------------------

Downloading the files one by one was painfully slow. Many of the servers would take 1-3 seconds to respond, and some of them wouldn't answer t all, timing out after a 60 second limit. Not cool. Luckily, it turns out the cURL PHP extension has an API for downloading files in parallel, asynchronously.

While the documentation was somewhat scarce, I found a fair amount of examples of how to do it. The only problem is, they are all written with the assumption that you need 3-10 files, not a million, so they will just block until *all* files are downloaded.

With some experimentation I found out that I could add more URLs as the old ones completed, maintaining a download "pool" of constant size. Ta-Da! Parallel, asynchronous IO in PHP. Amazing.

...Except for the fact that cURL in PHP doesn't come with asynchronous DNS resolution by default, so it still blocks until the connection is actually opened. Asynchronous DNS *should* work if PHP and libcurl is recompiled with [c-ares](http://c-ares.haxx.se/) enabled, but I don't want to go that far.

The code is [available on Github](https://github.com/geon/asciiart/blob/master/download.php) if you are interested in the details.

To count the files, you can run the command

	$ find cache | wc -l

It should list all the files in the cache dir and count the number of lines. Pro tip: It's much faster on a local disk and actual hardware than under VirtualBox and a mounted Samba share.

Detecting ASCII Art
-------------------

Coming up with a good heuristic for detecting ASCII art is surprisingly difficult. I found a few [earlier attempts](http://www.w3.org/WAI/ER/IG/ert/AsciiArt.htm), but they were mostly useless to me.

I quickly found really nice signatures that wouldn't fit the definitions I had in mind, so I didn't want to have too much prejudice of what ASCII art is. In the end It seemed like filtering out stuff I knew would *not* be ASCII art worked better.

After this filtering you are left with a short enough list to process manually.

I have set up some rules for this:

* Only the first HTML comment is considered. If you bothered to make a signature, you'd put it at the top where it's visible.

* ASCII art is at least 3 lines long. Otherwise it isn't very 2-dimensional. I want 2D.

* It is less than 40 lines long. The reasoning is that you wouldn't make a signature longer than a page of text. If you *actually* have a piece of ASCII art that big as your signature, it is most likely generated, which is kind of cheating.

* There shouldn't be HTML-fragments in it, nor javascript, IE conditional comments etc.

* Detect and discard various CMS debug info.

I used the command 

	$ sort ascii_art.txt | uniq -dc | sort -n

to find commonly occurring lines. The command sorts all lines, then filters out duplicates and counts them, then sorts them again by their frequency. Most of the high scoring ones were debug info and copyright notices that I could add to the heuristics.

Results
-------

Besides possibly getting me onto a few governments' watchlists, this experiment yielded some nice ASCII art.

### My Favorites

I picked out a few examples I think shows off the skill of the artist.

Gotta love the [Aperture Science Sentry Turret](http://half-life.wikia.com/wiki/Aperture_Science_Sentry_Turret).

Aparently, their designer is a Redditor.

### The Rest, In No Particular Order

####fightbeat.com

