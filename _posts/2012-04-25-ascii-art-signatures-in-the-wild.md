---
category : Programming
tags : [ASCII art, PHP, asynchronous, cURL]
title: ASCII Art Signatures In The Wild
---


<a href="https://github.com/geon/asciiart"><img style="position: absolute; top: 0; right: 0; border: 0;" src="https://a248.e.akamai.net/assets.github.com/img/30f550e0d38ceb6ef5b81500c64d970b7fb0f028/687474703a2f2f73332e616d617a6f6e6177732e636f6d2f6769746875622f726962626f6e732f666f726b6d655f72696768745f6f72616e67655f6666373630302e706e67" alt="Fork me on GitHub"></a>
<style type="text/css">
pre{
background-color: #222;
color: #eee;
line-height: 1em;
}
div.wide pre{
overflow-x: auto;
overflow-y: hidden;
padding-bottom: 20px;
}
div.wide pre code{
display: block;
width: 1000px;
}
</style>

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

To count the downloaded files, you can run the command

	$ find cache | wc -l

It should list all the files in the cache dir and count the number of lines. Pro tip: It's much faster on a local disk and actual hardware than under VirtualBox and a mounted Samba share.

The code is [available on Github](https://github.com/geon/asciiart/blob/master/download.php) if you are interested in the details.

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

Besides possibly getting me onto a few governments' watchlists, this experiment yielded some nice ASCII art. There is *way* too much to show in a blog post fitting the intended audience's attention span. Fortunately most of it is [auto generated crap](http://www.squidoo.com/online-ascii-art-generator), so you won't miss it. I've picked out a few of the more interesting examples.

### My Favorites

####onetruefan.com

Apparently, their designer is a Redditor.


	
	    :                                                         
	      7?               NARWHALS. OH YEAH.                    
	         =O?=                                                 
	             ~7=          88888,                              
	                =OZ?=   DDD8Z$7ZOOZ                           
	                    :O?8O8D8ZZI+==IZ8OZ                       
	                      =IZZ$Z7$ZOZZ7I=IZO8Z$                   
	                       O7=ZIZIZZZZO7ZI=IIZOOZO                
	                         ZI7I==?IIIZ7Z777+?$OOO7              
	                           Z$I+?=:=?II$77$ZII7Z$OZ            
	                            ?$Z77I===??II7$$III=I$O           
	                              7$$I+=====++++$I?+II$OZ         
	                               OI+=I+=:~====+I+7+7+?Z8        
	                                O$==7=::~=====+IIIII$ZO       
	                                 ?7=?ZO7+====+??+7IIIZO       
	                                 =I78Z  II$7I=??I77?I$ZO      
	                                 $ZZ8$     +7$$II7I7$7$8      
	                                 :88I         $Z7I7I77Z8      
	                                                IZ77$$ZO      
	                                                 7Z$$$$I      
	                                                  Z7$ZO       
	                                                  OZZZ7       
	                                                  Z$$8        
	                                                 :ZZ8         
	                                             88OOO8O          
	                                              8DD88Z          
	                                                 $8O8         
	                                                   Z8Z        

####aapeli.com

Gotta love the [Aperture Science Sentry Turret](http://half-life.wikia.com/wiki/Aperture_Science_Sentry_Turret).

	.__
	/ |\ Hello!
	| o| Can I help you?
	\_|/
	/ |\


####rooftopsolutions.nl

All glory to the HTML-toad!

	

                   ,'``.._   ,'``.
                  :,––._:)\,:,.–,.:       
                  :`,––''   :`...';\      
                   `,'       `––'  `.
                   /                 :
                  /                   \
                ,'                     :\.___,-.
               `...,––––'``````-..._    |:       \
                 (                 )   ;:    )   \  _,-.
                  `.              (   //          `'    \
                   :               `.//  )      )     , ;
                 ,-|`.            _,'/       )    ) ,' ,'
                (  :`.`-..____..=:.-':     .     _,' ,'
                 `,'\ ``––....-)='    `._,  \  ,') _ '``._
              _.-/ _ `.       (_)      /     )' ; / \ \`-.'
             `––(   `-:`.     `' ___..'  _,-'   |/   `.)
                 `-. `.`.``––––``––,  .'
                   |/`.\`'        ,','); 
                       `         (/  (/

                   =*=*=*= HET BIJSTERE SPOOR =*=*=*=

    Thanks a million for checking out my sourcode. Truly, I am honored.

    Unfortunately, you're just looking at a stock Habari template. One
                of these days I'll write my own again.

                                 

                                                     Evert, 2010-05-27 

####hachbe.be

<div class="wide"><pre><code>
	                                        _________________________________________________________
	                                       |                                                         |
	                                       |    Bienvenu Ã  toi p'tit curieu du code source!          |
	                                       |                                                         |
	                                       |    Je te souhaite une bonne navigation parmis ce code   |
	                                       |_  ______________________________________________________|
	                                         \/
	                                                                       
	                 `.:/oossssso/-``                 
	          `-:+osyysso++++++++ossyyo+-.            
	    `:+osyysoo+++++++++++++++++++++osyys+:.       
	   `yyo++++++++++++++++++++++++++++++++++osy:     
	   .hoo++++++++++++++++++++++++++////////+/oy`    
	   -hoooooo+++++++++++++++//////////++++++++h`    
	   -hoooooooooo++++++++/////ss++++++++++++++h`    
	   -hooooooooooooo+++++//++ysosso+++++oo+++oh`    
	   -hsooooooooooooooo+++++yo+//oso+++oysys+oy`    
	   .hsooooooooooooooo+++++o/.  `+o+++so/ooosy     
	   .hsssoooooooooooooo+++++.    `oo+++. .s+sy     
	    yysssooooooooooooo++++/     `/s++/  `ooss     
	    yyssssoooooooooooo+++o-     `-y+o/`-:osys     
	    yyssssosoooooooooo+++o:  .:/.-y+o+:ddsoyo     
	    syssssssooooooooooo++o/ :yhms:y++s::+yoho     
	    /hsssssssssoooooooo+++o``+so:os+ooossooh/     
	     /yysssssssssoooooo+++o+```-+y+oooooosyo`     
	       ./syysssssooooooo+++os++ssoooosyyo/.       
	          `:+yyyssooooooo+oo+ooossyyo:.           
	              .+syysoooooooosyys+:.               
	                 `:oyyyyyss+/.`                   
</code></pre></div>

####gearjunkie.com

	        HELLO: Welcome to GearJunkie.com's source code 
	
	                                    
	                                   7~8$:.                           
	                              .   ZMM.M7                           
	                        ..7~III?7I7DO=ID?                           
	                      .:77$7777$7$ZND$+ .                          
	                      .I+$Z88888DOO$$~.                             
	                     OI777$ZZODNNMOOZ=                              
	                     $N77I+++?O.  +=II~                             
	                    ..DNZZ+=7?=~=.I?=.+=                            
	                       $+DI8I+?+++I?=.7~=$$N$                      
	                        ?NM7?777I+I=?$=I+,,$NO.                     
	                        .7,III~=?7???8Z7~8N=?..                     
	                          =D:~+?.     ,,8M?$ .                      
	               . +ODOD$:$8I.~+.. ...=.Z8?ZZ7$? .                    
	              .88 .   .+8ZINN   .:~.,.D?Z=.  . ?I.                  
	             O7      .,O +,:~ .?=8.?.I.. $.    ..D                  
	             8.     .?8    =~= .8.. $    7Z      .8.                
	             I     ?DD+..  7N~. ,. ~D     88      I:                
	             $.    $N:=,ZOZZDON$   .8      Z      7.                
	             =?    D$. .  .OOND+I. .8+            8                 
	              $,.       ..7. .ON~    Z..         I                  
	               .7:,. ..:7O            O?..   ..:8.                  
	                  ..II.                  77???.


####webvariants.de

	                     ?II77$
	                   7II7Z$777Z
	                  ,III     77
	                  7II7     $7+
	                   7I7      7
	                    II     ,7
	                     I7    7
	                      7I  7
	                        II
	                      +'  +II
	                  ,+?=      ,I7       Solchen Code darfst Du auch schreiben,
	                ?++?          7I7     wenn Du bei uns arbeitest - oder sogar
	              ?+++?           ,7I7    noch besseren, wenn Du kannst und willst.
	             ?+++?             7$7
	             ?++?              $7$    Wir freuen uns immer Ã¼ber neue Kontakte
	            ,?+?               77Z    und Bewerbungen von fÃ¤higen Entwicklern.
	             ?+?              $77Z
	             ?+I             $7777    Guck doch mal hier nach:
	             ?+'            7777$     webvariants.de/unternehmen/jobs-praktika/
	      IIIIII7,+           77777$      oder schreib uns einfach eine mail an
	   ,II'       ,777I..777777777$       kuehle {Ã¤t} webvariants {dÃ¶t} de
	   II          I  $777777777+
	  $II         I7     'I7'
	   7II.    .:II7
	   ~7IIIIIIIIII
	     77IIIIIII




### The Rest, In No Particular Order



####rafflecopter.com

	    RAFL:RAFL:LOL:RAFL:RAFL
	               |
	      L   /=========
	     LOL===       []\
	      L    \          \
	            \__________\
	              |     | 
	           ==============/

####snotr.com

	     _
	  _\( )/_
	   /(O)\


####webcreatorbox.com
<div class="wide"><pre><code>

	            _______
	            |      |
	   _________|____  |
	   | _______|__  | |
	   | |      |______|
	   | |         | |     Design is not just what it looks like and feels like. Design is how it works.
	   | |         | |     Steve Jobs
	   | |         | |
	   | |  w e b  | |     デザインとは、単にどのように見えるか、どのように感じるかということではない。どう機能するかだ。
	   | |_________| |     スティーブ・ジョブズ
	   |_____________|
	 

</code></pre></div>

####nfb.ca

	   _____
	  / ___ \
	 / / _ \ \
	 \ \(_)/ /
	  \_   _/
	   |   |   Equipe web ONF /
	   |_|_|   NFB web team  / 2008-2011
	   =====


####vimeo.com

	         _
	  __   _|_|_ __ ___   ___  ___
	  \ \ / / | '_ ' _ \ / _ \/ _ \
	   \ V /| | | | | | |  __/ |_| |
	    \_/ |_|_| |_| |_|\___|\___/
	               you know, for videos

####surveygizmo.com

	   ____                       ______
	  / __/_ _______  _____ __ __/ ___(_)__ __ _  ___    ___  ___   __ _
	 _\ \/ // / __/ |/ / -_) // / (_ / /_ //  ' \/ _ \  / __)/ _ \ /  ' \
	/___/\_,_/_/  |___/\__/\_, /\___/_//__/_/_/_/\___/{}\___)\___//_/_/_/
	                      /___/  Making the world safe for surveys!


####teehanlax.com

	 _____            _                             _                 
	|_   _|___   ___ | |__    __ _  _ __      _    | |     __ _ __  __
	  | | / _ \ / _ \| '_ \  / _` || '_ \   _| |_  | |    / _` |\ \/ /
	  | ||  __/|  __/| | | || (_| || | | | |_   _| | |___| (_| | >  < 
	  |_| \___| \___||_| |_| \__,_||_| |_|   |_|   |_____|\__,_|/_/\_\
	
	/**************************************
	*
	*       Why you go look here?
	*
	***************************************/

####codeweavers.com

	               _
	  ___ ___   __| | _____      _____  __ ___   _____ _ __ ___ 
	 / __/ _ \ / _` |/ _ \ \ /\ / / _ \/ _` \ \ / / _ \ '__/ __|
	| (_| (_) | (_| |  __/\ V  V /  __/ (_| |\ V /  __/ |  \__ \
	 \___\___/ \__,_|\___| \_/\_/ \___|\__,_| \_/ \___|_|  |___/
 
####tunegenie.com

	  __                              _    
	 / /___ _____  ___ ___ ____ ___  (_)__ 
	/ __/ // / _ \/ -_) _ `/ -_) _ \/ / -_)
	\__/\_,_/_//_/\__/\_, /\__/_//_/_/\__/ 
	                 /___/                 


####netdirekt.com.tr
<div class="wide"><pre><code>

	
	             1000000000000000000000001   10  100000000000000000000000000000000000000000000000000001         
	           1000000000000000000000000000  00 000000000000000000000000000000000000000000000000000000001       
	        100011                          10           111  111                     11         101100000      
	       001                             100           100  000                    1001        001   00001    
	    1100        111           11      1101      111  000  00  11 11     1111     000         0001    1000   
	   1100      000000000    100000001 00000001 00000000001 000  000001 000000000   000  000000000001   0000   
	  1010       00001 1000  0001   1001 1000  10001   0000  000 10001  0001   1000 100110001  1000      1000   
	  1000      1001    000 000010000000  001  000      000 1001 000   000000000000 0000001    1001       0001  
	   000      000    1001 000100000001 000  1001     1000 000  000   000000000001 000000     000       1000   
	   0000     000    000  000          000   000   10000  000 1001   000          0001000    000       0000   
	    0000   1001    000   000000001  1000    0000000000 1000 000     000000001  000   0001 1001     10000    
	     00001  11      1      1001      11       1001  1   11   11       1001      11    10   11     00000     
	      000001                                                    11                            10000000      
	       100000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000        
	          1110000000000000000000000000000000000000000000000000000000000000000000000000000000000011  

</code></pre></div>

####n-c-team.com
<div class="wide"><pre><code>

	                                                                                           #
	                    ###             ##                                          ######       #
	                   #####            ##                                      #############     ##
	                   ######          ###                                     ######    #####    ##
	                  ########         ###                               ##   ####        #####   ###
	                  #########        ###   ########  ##### ##    ###  ##### ####   ##   ####    ###
	                 ###########       ### ##########  ##### ##    ###  ##### ####  #########     ###
	                 #####  #####     #### ##     ###  ##    ##    ###   ##   #####  ######     ####
	                #####    ######  ####  ###    ###  ##    ##    ###   ##    #####          #####
	                ####       #########    #########  ##     #######    ##      #################
	                             #####        #### ##  ##      #####     ##         ###########
	
	                  ######                   ##               ##      ####
	                 ##        ###  ##### #### ### ##### #####  ###      ##  #####  ##### #### ####
	                 ##       #   # #   #   ## ##  #   # ##  ## ##       ##  # # #  #   # ##  ##  ##
	                  ######   ###  #   # #### ##  ### # ##  ## ##       ##  #####  ### # ##  ##  ##
</code></pre></div>

####hakim.se

	         _______ _______ __  __ _______ _______    _______ _______ 
	        |   |   |   _   |  |/  |_     _|   |   |  |     __|    ___|
	        |       |       |     < _|   |_|       |__|__     |    ___|
	        |___|___|___|___|__|\__|_______|__|_|__|__|_______|_______|
	        
	        DAMN RIGHT, ASCII ART BABY.


####22tracks.com

	    .222222222222222222222222222.
	    22222+     :22222~     +22222
	    222         ..2           222
	    22    ~==, .2,    .=+~     22
	    2    .222222+    ,22222.   :2
	    2.   :2222222    222222~    2
	    2~    22222222222222222    ~2
	    22.    222222222222222     22
	    222:.    ~2222222222     .222
	    222222.     22222:     222222
	    22222222      2~     22222222
	    2222222222~.      ,2222222222
	    2222222222222.    22222222222
	    2                           2
	    2                           2
	    ~222222222222222222222222222~


####speedycash.com

	   ______               __    ______           __   
	  /   __/___ ___ ___ __/ / __/   __/___  ____ / /__ 
	 /__   / . / -_| -_|/ . /\/ /   /_ / .'|/_ -// _  /
	/_____/ __/___|___|/___/\  /\_____\\__,|/___/_/ _/
	     /_/              |___/                  


####sustainlane.com

	   ______
	  /      \
	 |        |
	 |        |
	  \_  ___/
	    |/
	        Live Responsibly.


####1doost.com

	    .dDDD     8DDDDDDDDDD.                                             .DD      
	    DDDDD     ODDDDDDDDDDDN                                           DDDD      
	 DDDDDDDD     ODDDD ..+DDDD.   . DDDD8      . dDDDD       .DDDDD+   O8DDDDOO .  
	 DDD.8DDD     ODDDD   .DDDDD  .DDDDDDDDD   .DDDDDDDDDd  .DDDDDDDDD..DDDDDDDD    
	     8DDD     ODDDD    DDDDD  DDDD ..DDDD. DDDD~  DDDD+.DDDD..    .   DDDD  .   
	     8DDD     ODDDD    DDDDD DDDD,   DDDD. DDDD  . DDDD  DDDDDDDDD.   DDDD      
	     8DDD     ODDDD    DDDD=.DDDDI  .DDDD  DDDD.   DDDD    .dDDDDDD.  DDDD      
	     8DDD     ODDDDDDDDDDDD  .DDDDD=DDDDD. 7DDDD+NDDDD  DDDD,. NDDD   DDDDDD    
	     8DDD     ODDDDDDDDDD.   ..NDDDDDDDI   . DDDDDDDD   .DDDDDDDDD..  DDDDDD..  
	    .   .     ..                 ... .        .. . .       . ...        .  ...  
	                                 ..    . .      . ..  ..     ...      ...  .    
	                               ..DDDDDDDDD .  ZDDDDDDDD,.. DDD8DDDDDD,DDDDDD    
	                                DDDD+. DDDD. 8DDDD. DDDD.. DDDDDDDDDDDDDDDDDd   
	                               .DDD?         DDDD    DDDD  DDDD...DDDD..=DDDD.  
	                          ... .,DDD~.  ... ..DDDD.   DDDD  DDDD ..DDDD .=DDDD.  
	                         DDDDI. DDDD . DDDD  DDDDD .DDDD.. DDDD ..DDDD .=DDDD.  
	                         DDDDI   DDDDDDDDD.   dDDDDDDDD    DDDD.  DDDD..=DDDD.  
	                             .    . 7OO...    .  =OO. ..         .    . .    .  
	                                                                                
	                                                                                
	                                                                     1Doost.com  

####arquinauta.com

	                       _     __  __  _____  __    __  ___  __        
	  __ _ _ __ __ _ _   _(_) /\ \ \/__\/__   \/ / /\ \ \/___\/__\  /\ /\
	 / _` | '__/ _` | | | | |/  \/ /_\    / /\/\ \/  \/ //  // \// / //_/
	| (_| | | | (_| | |_| | / /\  //__   / /    \  /\  / \_// _  \/ __ \ 
	 \__,_|_|  \__, |\__,_|_\_\ \/\__/   \/      \/  \/\___/\/ \_/\/  \/ 
	              |_|                                                    
	 arquiNETWORK.net

####cubiq.org

	                 _   _     
	         ___ _ _| |_|_|___ 
	        |  _| | | . | | . |
	        |___|___|___|_|_  |.org
	                        |_|
	
	        Welcome to the source, Neo. Make yourself home.
	        The code is to the bottom, everything's under MIT license.
	        http://cubiq.org/license

####photosecrets.com

	        :::::::::::::::::::::: Welcome to the source code of ::::::::::::::::::::::     
	                 _____ _          _        ______                   _ 
	                |  __ \ |___  ___| |_  ___/ ____/ ___  ___ _ __  __| |_ ___
	                | |__) | _  \/ _ \ __|/ _ \___  \/ _ \/ __| '__/ _ \ __|___|
	                |  ___/ | | | (_) ||_| (_) |__)  | __/ (__| | |  __/ |_\__ \
	                |_|   |_| |_|\___/\__|\___/_____/\___|\___|_|  \___|\__|___/
	
	        ::::::::: PhotoSecrets.com coding & design by Andrew Hudson, 2012 :::::::::

####boatnerd.com
	                                                           ______ 
	                                                           |O 0 |/_/
	  ______                                                   | 0  0  |_
	 |=======\_________________________-/\/\/\/\/\/\/\/\/\/\___|   0   0 |_
	 |                                                                     |
	 |              (C) By Boatnerd.com all rights reserved                |
	 |                       www.boatnerd.com                         _____|
	 |_______________________________________________________________/-+ b


####kitsune.fr

	                               __
	                             .d$$b
	                           .' TO$;\
	                          /  : TP._;
	                         / _.;  :Tb|
	                        /   /   ;j$j
	                    _.-"       d$$$$
	                  .' ..       d$$$$;
	                 /  /P'      d$$$$P. |\
	                /   "      .d$$$P' |\^"l
	              .'           `T$P^"""""  :
	          ._.'      _.'                ;
	       `-.-".-'-' ._.       _.-"    .-"
	     `.-" _____  ._              .-"
	    -(.g$$$$$$$b.              .'
	      ""^^T$$$P^)            .(:
	        _/  -"  /.'         /:/;
	     ._.'-'`-'  ")/         /;/;
	  `-.-"..-""    " /         /  ;
	 .-" ..-""         -'          :
	 ..-""-.-"          (\      .-(\
	   ..-""               `-\(\/;`
	     _.                      :
	                             ;`-
	                            :\
	                            ;  


####idiotcomputer.jp

	I'm not here, This isn't happening.
	 __     _____     __     ______     ______  
	/\ \   /\  __-.  /\ \   /\  __ \   /\__  _\ 
	\ \ \  \ \ \/\ \ \ \ \  \ \ \/\ \  \/_/\ \/ 
	 \ \_\  \ \____-  \ \_\  \ \_____\    \ \_\ 
	  \/_/   \/____/   \/_/   \/_____/     \/_/                                         
	 ______     ______     __    __     ______   __  __     ______   ______     ______    
	/\  ___\   /\  __ \   /\ "-./  \   /\  == \ /\ \/\ \   /\__  _\ /\  ___\   /\  == \   
	\ \ \____  \ \ \/\ \  \ \ \-./\ \  \ \  _-/ \ \ \_\ \  \/_/\ \/ \ \  __\   \ \  __<   
	 \ \_____\  \ \_____\  \ \_\ \ \_\  \ \_\    \ \_____\    \ \_\  \ \_____\  \ \_\ \_\ 
	  \/_____/   \/_____/   \/_/  \/_/   \/_/     \/_____/     \/_/   \/_____/   \/_/ /_/

####cheapmonday.com

	         __  __ ______ _______ _______ _______ _______    _______ _______
	        |  |/  |   __ \   _   |   |   |     __|       |  |     __|    ___|
	        |     <|      <       |       |    |  |   -   |__|__     |    ___|
	        |__|\__|___|__|___|___|__|_|__|_______|_______|__|_______|_______|


####unblogged.net

	                ...................................
	                ........................................
	                ..............MMMMMMMMMMI...............
	                ..........MMMMMMMMMMMMMMMMM.............
	                ........ MMMMMMMMMMMMMMMMMMMMM..........
	                .......MMMMMMMM..  ....?MMMMMMM ........
	                ..... MMMMMMM.. .  .......MMMMMM........
	                .... $MMMMM....    ..... ..MMMMMM  .....
	                .....MMMMM... .. .MM.... ...MMMMMM......
	                ....DMMMM,   MM   MM+..MM ...MMMMM......
	                ....MMMMM    MM   MMM .MMM...MMMMM .....
	                ....MMMMM .. MM:..MMM..MMM...MMMMM......
	                ....MMMMM    MM ..MMM. MMM...MMMMM......
	                ....IMMMMM...MM...MMM .MM?...MMMMM. ....
	                .....MMMMM ...M...MM....M....MMMMM......
	                ......MMMMM,.................MMMMM......
	                ......MMMMMMM.... ... . .....MMMMM......
	                ....... MMMMMMMM    . . .....MMMMM......
	                .........MMMMMMMMMMMMMMMMMMMMMMMMM......
	                .......... MMMMMMMMMMMMMMMMMMMMMMM......
	                ..............+MMMMMMMMMMMMMMMMMMM......
	                ........................................ 
	                ....................... Unblogged V4


####chobbit.com
<div class="wide"><pre><code>

                                            00          0                                                       
                                             0003     000                                                       
                                              00003   003                                                       
                                                0003  03                                                        
           00000000     0003      0003      0000000000     00000000003     00000000003     00003  00000000000003
         000000000003   0003      0003    0000000000000    0000000000003   000000000003    00003  00000000000003
        00003     003   0003      0003   000003    00000   0003    00003   0003    0003    00003       0003     
        0003            00000000000003   00003      00003  00000000003     00000000003     00003       0003     
        0003            00033333330003   00003      00003  0000333300003   0000333300003   00003       0003     
        00003    0003   0003      0003    00003    00003   0003     00003  0003     00003  00003       0003     
         00000000003    0003      0003     000000000003    0000000000003   0000000000003   00003       0003     
            00003       0003      0003        000003       00000000003     00000000003     00003       0003     
                                                                                                                
                                                                                         E-mail:info@chobbit.com
</code></pre></div>

####pioneer.com.br
<div class="wide"><pre><code>

                                    ..       ..                                             
  x=~                         . uW8"       dF                                               
 88x.   .e.   .e.             `t888       '88bu.         .u    .          u.    .d``        
'8888X.x888:.x888       .u     8888   .   '*88888bu    .d88B :@8c   ...ue888b   @8Ne.   .u  
 `8888  888X '888k   ud8888.   9888.z88N    ^"*8888N  ="8888f8888r  888R Y888r  %8888:u@88N 
  X888  888X  888X :888'8888.  9888  888E  beWE "888L   4888>'88"   888R I888>   `888I  888.
  X888  888X  888X d888 '88%"  9888  888E  888E  888E   4888> '     888R I888>    888I  888I
  X888  888X  888X 8888.+"     9888  888E  888E  888E   4888>       888R I888>    888I  888I
 .X888  888X. 888~ 8888L       9888  888E  888E  888F  .d888L .+   u8888cJ888   uW888L  888'
 `%88%``"*888Y"    '8888c. .+ .8888  888" .888N..888   ^"8888*"     "*888*P"   '*88888Nu88P 
   `~     `"        "88888%    `%888*%"    `"888*""       "Y"         'Y"      ~ '88888F`   
                      "YP'        "`          ""                                  888 ^     
                                                                                  *8E       
                                                                                  '8>       
                                                                                   "        
	~~~~~~~~~~~~~~~~~~~~~
	www.webdrop.net.br
</code></pre></div>

####skittles.com

	 i;;;;;;;,,,,,,,,,,,,,::::::::::::::::::::..:::::::::::::::::::::.:
	 i;;;,,,,,,,,,,,,,,::::::::::::::.................................:
	 ;;;,,,,,,,,,,,,,::::::::::........:::;;,;,:::....................:
	 ;,,,,,,,,,,,:::::::::::.......:,:,i;;ijjifjit;,:.................:
	 ;,,,,,,,,,:::::::::::........,fGLffLKKEKKWWELji:.................:
	 ;,,,,,,,,::::::::::.........;GDDEKWi.,;;;iLK#EGi;................:
	 ;,,,,,::::::::::...........:DEDGL. ,;;i,,,;i;jWEEt,..............:
	 ,,,,,:::::::::::........... ifjtt;ti;Gi:,;;iiiGKfi:..............:
	 ,,::::::::::::............. tff ,fffEDi:;fGEfijKDG,. ............:
	 ,:::::::::::.:..............:f. :iffff:GLtGfitDKKG: .............:
	 ,:::::::.:..................:,:,iiti:,,fLttjDDKWDt ..............:
	 ,::::::.....................t:ittft;,fLj,,,;tEKKj ...............:
	 ,::::........................iitL#WWGitji;itGj; ... .............:
	 :::::.......................:jjtffjtfEEtiiiG; ...................:
	 ::..........................:jfiijjjffjiift  ....................:
	 :........................::,;jGLi;;iitfjL, ......................:
	 :............ ...:,:t,ji.,i;i;LEfLGftfjji .....................:.:
	 :.......... :,,;tjtLtL;:,iijtjtjttttjjtji,: ...................:::
	 :.... .. . .fffftffjLt;ttjjjLLfjtttjfLLffj;f: ..............:::::,
	 :......  .;ttLLfffffffjffffffLLLLfffffjjjjfffti:.........::::::::,
	 :...     ,ifjLffLffffffffLfLfffffffffffffjffftt;;...:.:::::::::::,
	 ....    :ijjLLLGLfffffffLLLLLfLLLLLfffffffffffjfji,::::::::::::::,
	 :...   .ijLGfGLGLfffffLLLLLLLLLLffLLfffLffffLfjjfjjt:::::::::::,,,
	 :.... .;tfLLGLLGGLLLLLLLLLLLLLLLfLLLffLffffffjjfffft,::::::::::,,,
	 :.... ,;tLGLLGLDGLLLLLLLLLLLLLGLLLLLffLLfffLfLffjjfj,::::::::,,,,;
	 :....;ttjtLLLLLGDGGLLLLLLGGGGGGLLLLLLLLLfLLLffjffffj,::,:,,,,,,,,;

####tangogameworks.com

	      ,e,.    ,eg,                                      
	    gM"__"e ,@"","*,                                    
	    MF gg#Gf$E 4g@j#                                    
	    *#,"_GA '@g__Cg'                                    
	     ""$MF    $M""                                      
	        M1    Mf                                        
	       ,MM,_,$Mf       gggggggge_                       
	       MMMMMMMMM      ^"""""""MMMM#g.                   
	  ____JMMMMMMMMM____           "$MMMM#                  
	 ^""""MMMMMMMMMM@"""'     _____,gMMMMM@.                
	       MMMMMMMMf     _gg@MMMMMMMMMMMMMM@                
	       MMMMMMMMf   ,#MMMMM"     "MMMMMMM,               
	       @MMMMMMMf  jMMMMMM  ,gMg  $MMMMMM1               
	       $MMMMMMMf  @MMMMMM  &MMM  !MMMMMM                
	       ^@MMMMMMg  'MMMMMM,  '"   $MMMMM" _,   _,    _   
	        ^*@MMMMMg_ ^"@MMMMgg,,,g@MMMMF' =MM@ JMMM  MMM, 
	           """@M@MM@ggg""""""""""""'    ^Yf"  "f"  "@F  
	                   ''"'                                 

####dula.tv

	* /     \             \            /    \       
	* |       |             \          |      |      
	* |       `.             |         |       :     
	* `        |             |        \|       |     
	*  \       | /       / ___  ____  \\        :    
	*   \      \/   ___~~          ~____| \     |    
	*    \      \_-~                    ~-_\    |    
	*     \_     \        _.::::::::.______\|   |    
	*       \     \______// _ ___ _ (_(__>  \   |    
	*        \   .  C ___)  ______ (_(____>  |  / 
	*        /\ |   C ____)/      \ (_____>  |_/   
	*       / /\|   C_____)       |  (___>   /  \  
	*      |   (   _C_____)\______/  // _/ /     \  
	*      |    \  |__   \\_________// (__/       |  
	*     | \    \____)   `::::   ::'             | 
	*     |  \_          ___\       /_          _/ |
	*    |              /    |     |  \            | 
	*    |             |    /       \  \           | 
	*    |          / /    |         |  \           |
	*    |         / /      \__/\___/    |          |
	*   |           /        |    |       |         |
	*   |          |         |    |       |         |
	* NOT USELESS CODE!


####pokelondon.com

	|_   __ \  .'   `.|_  ||_  _| |_   __  |
	  | |__) |/  .-.  \ | |_/ /     | |_ \_|
	  |  ___/ | |   | | |  __'.     |  _| _
	 _| |_    \  `-'  /_| |  \ \_  _| |__/ |
	|_____|    `.___.'|____||____||________|

####smakprov.se

	  ____                  _
	 / ___| _ __ ___   __ _| | ___ __  _ __ _____   __
	 \___ \| '_ ` _ \ / _` | |/ / '_ \| '__/ _ \ \ / /
	  ___) | | | | | | (_| |   <| |_) | | | (_) \ V /
	 |____/|_| |_| |_|\__,_|_|\_\ .__/|_|  \___/ \_/
	                            |_|
	                Läs först, köp sen.
	                Copyright 2010, Smakprov Media AB



####encycmet.com

	            .-/ \                                  / -\-.
	        _.-~ /   \___  ______ __  _    _     _   /___| ~-._
	        \ /  -~||  __||_  __//  || |  | |  /| | / __/| .\ /
	         / . . ||  __| | |\ / ' || |__| |_/ | || (_/ |   \
	        / / ~| ||____| |_| /_/|_||____|____||_| \___\| |\ \
	       / /   |-~\    \ \ \ | || ||    |    // / /   /~-| \ \
	      / /__ / \  \____\|\_\|_/|_||____|___//_/\/___/  / __\ \
	     /  .-~\   \-~                                 ~-/\/~-.  \
	    /.-~    \                                         /    ~-.\
	   /~      .-         ENCYCLOPEDIA METALLICA          -.      ~\
	   \    .-~           Please   Don't    take           ~-.    /
	    \.-~              anything from the site             ~-./
	                      without  asking first.
	                      Thanks!    -Sem

####twindots.com

	                                      ,.   '\'\    ,===.
	     Quiet, Pinky; I'm pondering.    | \\  l\\l_ //    |   Err ... right,
	            _              _         |  \\/ `/  `.|    |   Brain!  Narf!
	          /~\\   \        //~\       | Y |   |   ||  Y |
	          |  \\   \      //  |       |  \|   |   |\ /  |   /
	          [   ||        ||   ]       \   |  o|o  | /  /   /
	         ] Y  ||        ||  Y [       \___\_==_ /_/__/
	         |  \_|l,======.l|_/  |       /.-\(____) /==.\
	         |   ('          `<   |       `==(______)===='
	         \  (/~`==____=='~\)  /           U// U / \
	          `-_)-__________-<_-'            / \  / /|
	              /(_#(__)#_)\               ( .) / / ]
	              \___/__\___/                `.`' /   [
	               /__`=='__\                  |`-'    |
	            /\(__,==~~ __)                 |       |__
	         /\//\\(  `==~~ )                 _l       |==:.
	         '\/  \^\      /^)               |  `   (  <   \\
	              _\ \-__-/ /_             ,-\  ,-~~-. \   `:.___,/
	             (___\    /___)           (____/    (____)    `==='


####accessmaincomputerfile.net

	aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
	88888888888888888888888888888888888888888888888888888
	8888"""""""""""""""8888888888888888888888888888888888
	8888    Access     8888888888888888888888888888888888
	8888     Main      8888888888888888888888888888888888
	8888    Computer   888888888888888888888888888888888"
	8888     File      888888888888888888888888888888888a
	8888aaaaaaaaaaaaaaa888888888888888888888888888888888a
	8888aaaaaaaaaaaaaaa888888888888888888888888888888888a
	88888888888888888888888888888888888888888888888888888
	88888888888888888888888888888888888888888888888888888
	88888888888888888888888888888888888888888888888888888
	88888888888888888888888":::::"88888888888888888888888
	888888888888888888888::;gPPRg;::888888888888888888888
	88888888888888888888::dP'   `Yb::88888888888888888888
	88888888888888888888::8)     (8::88888888888888888888
	88888888888888888888;:Yb     dP:;88( )888888888888888
	888888888888888888888;:"8ggg8":;888888888888888888888
	88888888888888888888888aa:::aa88888888888888888888888
	88888888888888888888888888888888888888888888888888888
	88888888888888888888888888888888888888888888888888888
	88888888888888888888888888"88888888888888888888888888
	8888888888888888888888888:::8888888888888888888888888
	8888888888888888888888888:::8888888888888888888888888
	8888888888888888888888888:::8888888888888888888888888
	8888888888888888888888888:::8888888888888888888888888
	8888888888888888888888888:::8888888888888888888888888
	88888888888888888888888888a88888888888888888888888888
	"""""""""""""""""""' `"""""""""' `"""""""""""""""""""



####popshuffle.com

	  __   __
	 (  \,/  )
	  \_ | _/  IN THE FUTURE EVERY URL WILL BE POPULAR FOR 1.5 SECONDS
	  (_/ \_)                          - thomas and the wise butterfly

####fatosemdia.com.br

	   FWSFWSFWSFWSFWSFWSFWSFWSFWSFWS
	  FWSFWSFWSFWSFWS        FWSFWSFW  SFWSFW
	 FWSFWSFWSFWSF             WSFWS   FWSFWS
	FWSFWSFWSFWSF              WSFWS  FWSFWSF
	FWSFWSFWSFWS      FWS       FWSF  WSFWSFW
	FWSFWSFWSFWS      FWS       FWS  FWSFWSF
	FWSFWSFWSFWS      FWS       FWS  FWSFWSF              2010_FAÇA! WEBSITES @ Grupo Faça!
	FWSFWSFWSFWS      FWSFWSFWSFWSF  WSFWSF               www.grupofaca.com.br
	FWSFWSFWS                 FWSF   WSFWSF               (66) 3531-4645
	FWSFWSFWS                 FWSF  WSFWSF
	FWSFWSFWS                 FWSF  WSFWSF                Designer:
	FWSFWSFWS                 FWS  FWSFWS                 Kimberlly Del Santoro
	FWSFWSFWSFW         SFWSFWSFW  SFWSF                  
	FWSFWSFWSFW         SFWSFWSFW SFWSFW
	FWSFWSFWSFW         SFWSFWSF  WSFWSF                  Programador / Estruturação:
	FWSFWSFWSFW         SFWSFWSF FWSFWSF                  -
	FWSFWSFWSFW         SFWSFWSF                          
	 FWSFWSFWSF         WSFWSF   WSFWSF                   
	  FWSFWSFWS         FWSFWS  FWSFWSF
	   FWSFWSFW         FWSFWS  FWSFWSF
	       FWSF         WSFWSFW  SFWSF


####blueline.ca

	             ,
	        _.-"` `'-.
	       '._ __{}_(
	         |'==.__\
	        (   ^_\^
	         |   _ |
	         )\___/
	     .=='`:._]
	    /  \      '-.
	  Blue Line Magazine


####letsgift.it
	   __       _   _         ___ _  __ _      _____ _
	  / /   ___| |_( )___    / _ (_)/ _| |_    \_   \ |_
	 / /   / _ \ __|// __|  / /_\/ | |_| __|    / /\/ __|
	/ /___|  __/ |_  \__ \ / /_\\| |  _| |_  /\/ /_ | |_
	\____/ \___|\__| |___/ \____/|_|_|  \__| \____/  \__|

####wallbot.net

	                                                       ___
	                                              ___      \   \
	                                              \   \     \    \
	                                               \    \___________======___
	LOLOLOLOLOLOLOLOLOLOLOLOLOLOLOLOLOLOLOLOLOLOLOL(/  /_______loljet________/
	                                               /__/        |    /
	                                                          /    /
	                                                         /    /
	                                                        /___ /

####localist.co.nz

	   _                   _
	  | |                 | | o
	  | |  __   __   __,  | |     , _|_
	  |/  /  \_/    /  |  |/  |  / \_|
	  |__/\__/ \___/\_/|_/|__/|_/ \/ |_/

####nihongoup.com

	                                          :@@9             9@B@,      
	                                           ,Bs    2@      M@,,r       
	                                           :@     G9     X@      ,@i  
	                                           @B            @9      B@   
	                      @B                   Bs   9B@   sB@@@M@B@MGB@M@B
	                     XBB                  s@     @@     B@      9@    
	                     @@                   @B    ,B:     @G      @G    
	     ,rr         :r:i@B   ,,,      ,     ,Br    B@     rB,     r@,    
	   s@@BB@B    iB@B@@@Bs  :@B@:    @B     s@     @5     9@      9@     
	  B@s   G@S  MBG    B@     @B    SB9     X,    i2      X       X      
	 S@i   i@@  @Bs    :@@    GBs    B@     :9     2s     rS      rX      
	:@BHXMB@r  :@@     @B:    B@    ,@B     Gs    :@      Br      Bs      
	@B9        9@9     B@    r@G    MBr                                   
	M@B        9B@   ,B@2    B@9  ,XB@                                    
	 i@B9XBBS   @@@B@;2B@Xs  r@B@BH @@SG



####soleilnoir.net

	........................................
	......?O8888888888. .88888888888?.......
	... 8888888888888888888888888888888.....
	..8888888888888888888888888888888888?...
	.O888888888888888888888888888888888888..
	?8888888888888888888888888888888888888..
	8888888O8N8?.8N8N8N888888+ +8888888N88N.
	8NN88N8NN?.....88O8N8N88N     NN8N888888
	NONNONNNM......+NNN8OO88N     NNNONNNN88
	NDDDDDDDNO......:7NDDDDD8     N8D8DDDDD8
	8NDDDDND8DD7==~,....7DD8=.....NDNDNDDNN8
	?NNNNNNNONONNNNNNN. ..8+......N8NNNNNNN8
	 8NNNNNNNNNNNNNNNNNN8.........NNNNNNNNN8
	  ?NNNNNNNNNNNNNNNNNNN........NNNNNNNNN8
	    +NNNNNNNNNNNNNNNNNN.......NNNNNNNNNN
	8N+    NNNNNNNNNNNNNNNNN......NNNNNNNNNN
	NNNNN       .?NNNNNNNNNNN.....NN.....NNN
	88888?8+?  .. .88OONNNNN8.....8O......88
	........................................
	////// Soleil Noir Studio - Wøøt! \\\\\\
	........................................




####thecornernews.com

	_____              _____                            
	  |   |__   ___    |       ___   ___      ___ ___   |\  | ___            __
	  |   |  |  |-     |      |   |  |   |\ | |_  |     | \ | |_  \  /\  /  |__
	  |   |  |  |__    |____  |___|  |   | \| |__ |     |  \| |__  \/  \/   ___|
	
	In the middle of it ALL! -->
	
	
####top7.com
                                                                
	                777777777777                   77777777777  
	                     77                               ,7P'  
	                     77                              d7'    
	                     77 ,adPPYba,   ,dPPYba,       ,7P'     
	                     77a7'     '7ad7P'    '7a     d7'       
	                     777b       d777       d7   ,7P'        
	                     77'7a,   ,a7'77b    ,a7'  d7'          
	                     77 `'YbbdP'' 77`  cdP''  7P'           
	                                  77                      
	                                  77   Home of the Human Choice Project

####bmwcarshowroom.com

	                                                ._.
	                                               `.  \
	                                                 \  \
	                                                  .  \
	                                                  :   .
	                                                  |    .
	                                                  |    :
	                                                  |    |
	  ..._  ___                                       |    |
	 `."".`''''""==..___                              |    |
	 ,-\  \             ""-...__         _____________/    |
	 / ` " '                    `""""""""                  .
	 \                                                      L
	 (>                                                      \
	/                                                         \
	\_    ___..===.                                            L
	  `=='         '.                                           \
	                 .                                           \_
	                _/`.                                           `.._
	             .'     -.                                             `.
	            /     __.-Y     /''''''-...___,...========.._            |
	           /   _."    |    /                ' .      \   '===..._    |
	          /   /      /    /                _,. '    ,/           |   |
	          \_,'     _.'   /              /''     _,-'            _|   |
	                  '     /               `=====''               /     |
	                  `...-'                                       `...-'
	
	                                  I'M IN UR SAUCE!


####merchantandblack.com
<div class="wide"><pre><code>

                                          - - - - - 

                               - - - - - - - - - - - - - - - - 
                                   - - -             - - -     
                                       - - - - - - - - - 

                             /\   - - -            - - -      /\
                            /  \                             /  \
                           /    \                           /    \
                          /      \                         /      \
                         \        \//\_____\//\//\/____/\\/        /
                        \                   - \/ -                 /
                       (            -          -          -          )
                      \\                  \\        //               //     
                                 -   -      -\/\/-     -       
                       /                                              \ 
 - - - - - - - -       \              _____          _____            /    - - - - - - - -        
                                     //===\\        //===\\            /
         - - -          \             \== /          \ ==/             /  - - - 
                        \               ===          ===              \
                         /                    ____                    \
                          \                    \/                    /
                          //   _\\\/__/                 \\_\//_/   \\
 ___________________________//_     _ \ _______()______/ _    _\___________________________                 
                               \//\/\/                   \/\/\/
                                    ************************
                                     KITTY IS WATCHING YOU!
                                     DON'T MAKE KITTY ANGRY.
                                    ************************

</code></pre></div>

####kissfm.com

	                  |))    |))
	    .             |  )) /   ))              .oooooooo.   .ooooooooo.
	    \\   ^ ^      |    /      ))          -sddddddddddy::hdddddmmmmmy-
	     \\(((  )))   |   /        ))        -ddddddhhhhhh    ddddddmmmmmd-
	      / G    )))  |  /        ))         sddddhhhhhhhhhhddddddddmmmmmms
	     |o  _)   ))) | /       )))          odddhhhhhhhhhhhhddddddddmmmmmo
	      - ' |     ))`/      )))            -hdddhhhhhhhh    dddddddmmmmh.
	       ___|              )))              -ydddhhhhhhh    ddddddmmmmy-
	      / __\             ))))`()))          `+hdddddddd    ddddddmmh/`
	     /\@   /             `(())))             ./ydddddd    ddddmdy:`
	     \/   /  /`_______/\   \  ))))             `:sdddd    ddmdo-`
	          | |          \ \  |  )))                .+hm    mh+.
	          | |           | | |   )))                 `+    /`
	         /_/           /_/_/
	   
	   Come work with us in SoHo, NYC! 
	   We're hiring JS ninjas and Python neckbeards (or viceversa. ninja outfit and neckbeard optional)
	   Drop a line and resume to webjobs[at]iheartradio.com


####swisstalk.ch

	                 _         __       ____
	      ____    __(_)__ ___ / /____ _/ / /__
	     (_-< |/|/ / (_-<(_-</ __/ _ `/ /  '_/
	    /___/__,__/_/___/___/\__/\_,_/_/_/\_\Ⅲ


####voleurz.com
<div class="wide"><pre><code>

	                                   00000                                                
	                                   00000                                                 
	                                   00000                                                 
	        00000        ~00000000D    00000   =00000000,   00000        =00000000000000000  
	        00000     ?000000000000000 00000=00000000000000I00000      N0000000000000000000  
	        00000    000000,     000000000000000Z    :00000000000     00000O       000000,   
	        00000  0000000        I000000000000O    0000I  :00000     00000~     000000      
	        000000000000000       00000000000000=         0000000     00000~   000000        
	        000000008 0000000000000000:000008000000000000000000000000000000~:00000000000000  
	        0000008     +00000000000   00000   000000000000   7000000000000~000000000000000
	
	        Site by Bruce Giovando â - brucegiovando.com
	        Copyright Voleurz Design Limited 2011 â - voleurz.com

</code></pre></div>

####jamiedubs.com

	  ____  ___    __ __  ____    _____
	 |    ||   \  |  T  T|    \  / ___/
	 l__  ||    \ |  |  ||  o  )(   \_
	 __j  ||  D  Y|  |  ||     T \__  T
	/  |  ||     ||  :  ||  O  | /  \ |
	\  `  ||     |l     ||     | \    |
	 \____jl_____j \__,_jl_____j  \___j

####diavolo.us
<div class="wide"><pre><code>

	              3.141592653589793238462643383279  
	            5028841971693993751058209749445923  
	           07816406286208998628034825342117067  
	           9821    48086         5132           
	          823      06647        09384           
	         46        09550        58223           
	         17        25359        4081            
	                   2848         1117            
	                   4502         8410            
	                   2701         9385            
	                  21105        55964            
	                  46229        48954            
	                  9303         81964            
	                  4288         10975            
	                 66593         34461            
	                284756         48233            
	                78678          31652        71           _____   _                  _       
	               2019091         456485       66          (____ \ (_)                | |      
	              9234603           48610454326648           _   \ \ _  ____ _   _ ___ | | ___  
	             2133936            0726024914127           | |   | | |/ _  | | | / _ \| |/ _ \ 
	             3724587             00660631558            | |__/ /| ( ( | |\ V / |_| | | |_| |
	             817488               152092096             |_____/ |_|\_||_| \_/ \___/|_|\___/ 
	
	                        diavolo.us                                                

</code></pre></div>

####mutewatch.com

	      __  __          ______
	    /\  /\  /\   /   / ____/\
	    \ \ \/ / /  /    ) ) __\/
	     \ \__/ /    /    \ \ \
	      \__/ /    /     _\ \ \
	      / / /      /   )____) )
	      \/_/      /    \____\/
	
	         YOUNG / SKILLED
	         youngskilled.com 


####u6studio.com

	Copyright by:
	   ___   ___    _________
	 /___ //___ / /_________ /|
	|    ||    | |          | |
	|    ||    | |     _____|/
	|    ||    | |    |/____ /|
	|    ||    | |          | |
	|    ||    | |          | |
	|    ||    | |    ||    | |
	|    ||    | |    ||    | |
	|    ||    | |    ||    | |
	|          | |    ||    | |
	|__________|/|    ||    | |
	 /_________ /|    ||    | |
	|  STUDIO  | |          | |
	|__________|/|__________|/



####wiggintonmail.co.uk
<div class="wide"><pre><code>

	                                        ***=====****
	                                      @@@@@    @@@@!**
	@@@   @ @@@                          @@@!!! @ @  :;!@**        @    @               @@@  @@@       @@    @@     
	@@@   @   @  @@@@    @   @@@@@@     @@@!!!   @   :;;!@**      @@@  @@@       @@@@@@   @  @@@        @    @@     
	@@@@@@@  @@@  @@@@  @  @@@@  @@@    @@!!!       ::;;!@**     @@@@@@@@@@          @@@ @@@ @@@        @    @@  @@ 
	@@@   @  @@@   @@@@@   @@@@@@@@@@    @@@!!     ;;;;!@**     @@ @@@@ @@@@     @@@@@@@ @@@ @@@        @    @@@@@@ 
	@@@   @  @@@    @@@    @@@@            @@!!:    ;;!@**     @   @@   @@@@@  @@@@  @@@ @@@ @@@       @@@@      @@ 
	@@@   @  @@@     @      @@@@@@@@         @!!:   ;@**      @          @@@@@  @@@@@@@@ @@@ @@@       @@@@ @@   @@ 
	                                           !: ;@**
	                                            !@*

</code></pre></div>

####googlezon.jp

	　　　　　　　　　＼　　　∩─ｰ､ 　　　====
	　　　　　　　　　　 ＼／　●　､_ ｀ヽ 　　======
	　　　　　　　　　　　/ ＼(　●　 ● |つ
	　　　　　　　　　　　|　　 X_入__ノ 　 ミ　　　そんな餌で俺様が釣られクマ――
	　　　　　　　　　　　 ､　(＿／　　　ノ　/⌒l
	　　　　　　　　　　　 /＼＿＿＿ノﾞ＿/　 /　　=====
	　　　　　　　　　 　 〈　　　　　　　　 ＿_ノ　　====
	　　　　　　　　　　　 ＼　＼＿　　　　＼
	　　　　　　　　　　　　　＼＿__）　　　　　＼　　　======　　　(´⌒
	　　　　　　　　　　　　　　　　＼　　 ＿＿_ ＼＿＿　　(´⌒;;(´⌒;;
	　　　　　　　　　　　　　　　　　 ＼＿＿＿）＿＿＿）(´;;⌒　 (´⌒;;　　ズザザザ 


####bobbeamanclub.com
<div class="wide"><pre><code>

	#
	# NM                   NM         NM                                                                
	# NMh.-:-.     .:::-`  dMm.-:-.`  MMy.-:-.     .-::-`    .-::-.` ....::.``.::-`   `.-::-` `..`-:-.   
	# NMNmmmNNh: :hNmdmNmo`dMNmmmNNd/`MMNNmmNNh- :hNmddNms`.hNmhhmNh-NMNmmNMmdNmmMmo`/dNmhhNNs+MMmmmNNh. 
	# NMN:``.hMN/NMy. `/MMsmMN/``.sMM/MMm:``-hMN/NMmo++yMMy-oso+ohMMsNMd.`.dMM:``oMM:/os++omMMsMMo``:MMs 
	# NMm.   oMMoMMo   .NMymMN.   +MM+MMd`   yMMoMMmyssyddsyNNysooMMsNMy   hMN`  /MModNmysosMMsMM:   MMy 
	# NMMmysdMNo`+BOBBEAMA.NmMMNyshNNs`MMMmyydMN+`+NNhsymNh-hMNysymMMsNMy   hMN`  /MMoNMmssyNMMsMM:   MMy 
	# +o//oso/.   `:ooo/.  /o/:oso/.  +o:/oso/.   `:+oo+-` `:+sso:+o/+o:   /o+   .oo.`/oso+:+o/oo.   +o:
	# 

</code></pre></div>

####studiosugarfree.com

	          /\
	         /  \
	        /    \
	        \    /
	         \  /
	          \/       
	   
	        studio
	      SUGARFREE


####sennentuntschi.com

	        MMMMMMMMMMMMMMMMMMMMMMM
	        M                     M
	        M                   MMM
	        M                   MMM
	        M                     M
	        M                     M
	        M                     M
	        MMM                   M
	        MMM                   M
	        M                     M
	        MMMMMMMMMMMMMMMMMMMMMMM
	        =======================
	        SQUARE interactive GmbH
	             www.square.ch
	        =======================


####deepend.com.au

	                                          +.-=::/+++o+-    
	                                   .:/+shdmmmmNNNMMNd/    
	                              ./oydmNMMMMMMMMMMMMMNy-     
	                           :oymNNMMMMMMMMMMMMMMMMNy:    
	                         .+hmNMMMMMMMMMMMMMMMMMMMMm/     
	                       odNMMMMMMMMMMMMMMMMMMMMMMMs.   
	                     -hMMMMMMMMMMMMMMMMMMMMMMMMMM/    
	                     .sNMMhs                smMM-     
	                      -yNNms:               .hMM/    
	                       /hNNmy/-             sMMs.    
	                         /ymNNmho:.         -NMm/    
	                            ./shmNNmdys+    yNMh:;\   
	                              -/sydmNNmmmddhhyhNMNy:     
	                                   .-:/osyhddmmmNNNNd|    
	                                           ..-=:::/+/-


####x-equals.com
<div class="wide"><pre><code>

	XXXXXXXXXXXX  XXXXXXXXXXXXX ALL WEBSITE CONTENT COPYRIGHT 2000 - 2009 BY X=PHOTOGRAPHY+CONSULTING
	XXXXXXXXXXXX  XXXXXXXXXXXXX
	   XXXXXX        XXXXXX
	    XXXXXX      XXXXXX
	     XXXXXX    XXXXXX
	      XXXXXX  XXXXXX
	       XXXXXXXXXXXX    XXXXXXXXXXXX
	        XXXXXXXXXX     X-EQUALS.COM 
	       XXXXXXXXXXXX    XXXXXXXXXXXX
	      XXXXXX  XXXXXX
	     XXXXXX    XXXXXX 
	    XXXXXX      XXXXXX
	   XXXXXX        XXXXXX
	XXXXXXXXXXXX  XXXXXXXXXXXXX
	XXXXXXXXXXXX  XXXXXXXXXXXXX ALL WEBSITE CONTENT COPYRIGHT 2000 - 2009 BY X=PHOTOGRAPHY+CONSULTING

</code></pre></div>

####mfsoft.sk

	  MMMMMMM        MMMMMMMMMMMMMMMMM                                    M        
	   MMMMMMMM   MMMMMMMMMMMMMMMMMMMM                                    MM       
	    MMMMMMMMMMMMMMMM   MMM      MM  MMMMMM     MMMMMM   MMMMMMMMMMM   MMM      
	    MMM   MMMMM  MMM   MMM        MMMMMMMMM  MMMMMMMMM   MMMMMMMMMMM  MMM      
	    MMMM    MM   MMM   MMM        MMM   MM  MMM     MMM   MMM    MMMMMMMMMMMMM 
	    MMMM    MM   MMM   MMMMMMMM   MMMM     MMM       MMM  MMM      MMMMMMMMMM  
	    MMMM    MM   MMM   MMMMMMM     MMMMMM  MMM  MMM  MMM  MMMMMMM     MMM    
	    MMMM    MM   MMM   MMM         MMMMMMM MMM  MMM  MMM  MMMMMM      MMM      
	    MMMM    MM   MMM   MMM             MMMMMMM       MMM  MMM         MMM      
	    MMMM    MM   MMM   MMM        MM     MMMMMM     MMM   MMM         MMM      
	   MMMMMM   MM  MMMMMMMMMMMM      MMMMMMMMM  MMMMMMMMM   MMMMM       MMMMM     
	  MMMMMMMM  MM MMMMMMMMMMMMMM       MMMMMM    MMMMMMM   MMMMMMM     MMMMMMM    
	            MM                          © 2010 MFsoft.sk   http://MFsoft.sk                
	           MM 


####geeks.cat

In case you wonder, that string of hex numbers decodes to "Geeks.cat".

	 _____           _                     _   
	|  __ \         | |                   | |  
	| |  \/ ___  ___| | _____    ___  __ _| |_ 
	| | __ / _ \/ _ \ |/ / __|  / __|/ _` | __|
	| |_\ \  __/  __/   <\__ \_| (__| (_| | |_ 
	 \____/\___|\___|_|\_\___(_)\___|\__,_|\__|
	47 65 65 6B 73 2E 63 61 74 



####onebyfourstudio.com

	             .                        
	                  .   :   .           
	              '.   .  :  .   .'       
	           ._   '._.-'''-._.'   _.    
	             '-..'         '..-'      
	          ==._ /.==.     .==.\ _.==   
	              ;/_o__\   /_o__\;       
	         =====|`     ) (     `|=====    ONE BY FOUR STUDIO
	             _: \_) (\_/) (_/ ;_     
	          =='  \  '._.=._.'  /  '==   
	            _.-''.  '._.'  .''-._     
	           '    .''-.(_).-''.    '    
	          jgs .'   '  :  '   '.       
	                 '    :   '           
	                      '                                            


####oocities.org
<div class="wide"><pre><code>

	       X@@@W8S             8MW@00:                     .                .                            
	   MMMMMMMMMMMM0      XMMMMMMMMMMMZ@                  MMMM7   ZMWMS   :MMM:                         
	 MMMM@MZM XMMMMMMr  M8MM@MM0X aMMMMMM                 MMMM7   ZWZM;   :MMM:                         
	 MMMW@M    0SMW@MM  MMMW@MM   X7MMW@M0M   :MMMMMMM            aZ20:                                 
	rMWWWMM    7;MWWMM  MMWW@MM   r MMWWMBM iM@W@M@MMMM.  MMMM.7M@0a2aB@MW MMM.  SBMMMMMMWi    MMMMMMMa 
	XMWWWMM     .MWW@M  MMWW@MM   . MMWWMMM M88@X    ZMM  M2aW XMMBa2Z@MMM Ma@  BM8WMSraMWMW  8M8  2MMMW
	XMWWWMM    .iMWWWM  MMWW@MM   i MMWWWMM M2B       aZ; M2aB    SZ20     Ma0 aWa2B     Za@7 BW        
	SMWWWMM     .MWW@M ,MMWW@MM   . MMWWMMM M2W,          M2aB:   0Z2Wr    MaZ,ZZ22BBMMMWBZBM 8MBMMMMWi 
	,MM@WMM    riMWWMM  MMMW@MM   i MMWWMMM M2W,          M2aB:   0Z2Wr    Maa.aa2aB8MMMW080@  MM@B8ZZW0
	 MMMW@M;M  MMMWMM:  M@MWWMM7r BMMMWMM   M2B:      MMr M2aB:   0Z2B:    Ma8 ZZ22Z             ,88W822
	 7MMMMM@M ZMMMMMM   0.MMMMMMM @MMMMMM   MWBW2    2MM  M2aB:   W82Z8    MaB aMZaM    ZMMMM@MM     822
	   iMMMMMMMMMMS2       MMMMMMMMMMMr8     ;@MMMMMMM:   MZ8Mi   WM8ZB@M@ M0M  ;M@WMMMMM@M0  0MMMMMM@@Z
	         i.                  ;             ii7XX7a    ai;X.    iiX77Si S;X    .:7XXX;.     rrXXXri  

</code></pre></div>

####jest.com

	             `do
	               d8.
	         .d88bo.8P
	        8P     8P                     88
	        `()    88 .od888bo. .od888bo. 88
	               88 88     88 88     88 888888
	               88 888888888 88o.   88 88
	               88 88         `8b.     88
	               88 88          `8b.    88
	        88     88 88     88 88  `8b.  88     88
	        88     88 88     88 88   `8b. 88     88
	        'Y88888P' 'Y88888P' 'Y88888P' 'Y88888P'

####duedil.com

	           ........           
	       .';;;;;;;;;;;;,..      
	     ';;;;;;;;,,,;;;;;;;;.    
	   .;;;;;,.        ..;;;;;,   
	  ';;;;,.             .;;;;;. 
	 .;;;;,     .,;;;'.    .;;;;; 
	 ,;;;;     ;;;;;;;;'    ';;;;.
	 ;;;;;    .;;;;;;;;;    .;;;;.
	 ';;;;.    ';;;;;;;.    ,;;;;.
	  ;;;;,      .....     ';;;;' 
	  .;,.               .,;;;;,  
	         .'...   ..';;;;;;.   
	       .,;;;;;;;;;;;;;;,.     
	        ..',;;;;;;;,'.        
	              ...             
	
	   ____                 _ _ _ 
	  |  _ \ _   _  ___  __| (_) |
	  | | | | | | |/ _ \/ _` | | |
	  | |_| | |_| |  __/ (_| | | |
	  |____/ \__,_|\___|\__,_|_|_|


####wufoo.eu

	Welcome to
	  __       __)              /  
	 (, )  |  /      /)        /   
	    | /| /      // _  _   /    
	    |/ |/  (_(_/(_(_)(_) o     
	    /  |      /)  
	             (/ 



####lyst.com
	
	             ,gggg,
	            d8" "8I                          I8
	            88  ,dP                          I8
	         8888888P"                        88888888
	            88                               I8
	            88        gg     gg    ,g,       I8
	       ,aa,_88        I8     8I   ,8'8,      I8
	      dP" "88P        I8,   ,8I  ,8'  Yb    ,I8,
	      Yb,_,d88b,,_   ,d8b, ,d8I ,8'_   8)  ,d88b,
	       "Y8P"  "Y88888P""Y88P"888P' "YY8P8P88P""Y88
	                           ,d8I'
	                         ,dP'8I
	                        ,8"  8I
	                        I8   8I
	                        `8, ,8I
	                         `Y8P"


####28fun.com

	      OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO
	      OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO
	      OOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOOO
	      OOOOOO                           OOOOOOO
	      OOOOO      OOOOO      OOOOO       OOOOOO
	      OOOOO     OOOOOO      OOOOOOO      OOOOO
	      OOOOO   OOOOOOO         OOOOOOO    OOOOO
	      OOOOO  OOOOOO             OOOOOO   OOOOO
	      88888 888888               888888  88888
	      88888                              88888
	      88888        8888888888888         88888
	      88888     8888888888888888888      88888
	      88888    888888888888888888888     88888
	      88888   8888888         8888888    88888
	      88888  888888             888888   88888
	      88888  88888               88888   88888
	      DDDDD  DDDDD               DDDDD   DDDDD
	      DDDDD  DDDDD               DDDDD   DDDDD
	      DDDDD  DDDDD               DDDDD   DDDDD
	      DDDDD  DDDDDD             DDDDDD  DDDDDD
	      DDDDDD  DDDDDD           DDDDDD  DDDDDDD
	      DDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDD
	      DDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDDD

####llustre.com

	    __.   __.
	  /.   \/.   \
	 /  }   } }   }       ___I_
	   /   / /   /       /\-_-_\       { `v` }        /\
	  /   / /   /    A  /  \_-__\  you  \   /  to ___/  \  /\____ in
	 /   / /   /        |[]| [] |        \ /             \/
	{   { {   {
	 \   \.\   \./
	   ```   ```


####guilty-crown.jp
<div class="wide"><pre><code>

	         WMN"    i                                 yMNy
	        dMMK   ]LAL __ __,     ________           dMME_____  ,______.      qmm     mg,           /
	        MMN    /'  dMF]MN                   gx   uMP""""MMK            mm,;MM[,,    \MNk      _y'
	   hvvvMMMFvvvv   uMNLMMF   ____________  qMN"  uNF    dMNF,,,,,,,qaa nMNF    ,0     P^     _d"
	      _MMN        MMF]M\    """"uMNF"""'_gM'   gN"    dMMF       _MMF gN      d[         _gM"
	      ]MMF       ]MPnMNL_y'   uMMF    ,mNMF   /'     dMMP       _MMF         dMP      _mMP"
	=====qMMN====== uMF dMMMP    uMN'    "  MN         _MMN'       yMP'        uMP'   _mMMME'
	     dMNL      yP'  TNP^   _dP"        ]MF        xMMP       y4P'        ,dP'   MMMMMP"
	    qM[       /"    "    y/"           4        _MMP'      -"          -""      4NP"
	    MN       '                                 gNP'
	   dNF                                       yN"
	                                           y/"

</code></pre></div>

####jingit.com

	             -====               ###  ###                            @@@
	          .::::======            ###  ###                            @@@   .e@
	         ::::     ====           ###                                       @@@    (R)
	        :::         ==e          ###  ###  ## .####.      .####. ##  @@@@@@@@@@@@
	        ::           =@          ###  ###  ##########. .###########  @@@"""@@@"""
	        :*           @@          ###  ###  ###'   '### ###'    '###  @@@   @@@
	        '**         @@@          ###  ###  ###     ### ###      ###  @@@   @@@
	         ****     @@@@           ###  ###  ###     ### ###.    .###  @@@   @@@
	          ******@@@@*     ###...###   ###  ###     ### '###########  @@@   @@@eee
	             *****         "#####'    ###  ###     ###   '#####'###  @@@   '@@@@'
	                                                                ###
	                                                        ###....###        
	                                                         '######'
	                                                         
	        (c)2012 Jingit
	        Site v1.2

####lemmetweetthatforyou.com

	         .-"-.
	        /  ,~a\_
	        \  \__))>
	        ,) ." \
	       /  (    \
	      /   )    ;
	     /   /     /  http://okfoc.us | @okfocus
	   ,/_."`  _.-`
	    /_/`"\\___
	         `~~~`

####tazai.com

	                                                               x
	                                                               xx
	                        xxxxxxxxxxxxxxxxxxxxxxx                 xx
	                     xxxx                     xxxxxx             xx
	                  xxx                               xxxxx         xx          xx
	                 xx                                     xxxx       xx       xxx
	                xx                                          xxx     x      xx
	               xx                                             xxx    x   xx
	              x                                                  xxx xx xx
	             x             xxx                                     xxx  x
	            xx            xxxxx                                      xx
	           xx             xxxxx                                        xx
	           x                                                            xx
	           x                                                             x
	           x xxx                       xxx                              xx
	            xx   xxxx           xxxxxxxx                               xx
	            xx       www.tazai.com                                    xx
	             xx                                                      xx
	              xxx                                                  xxx
	                xx                                               xxx
	                 xxx                                           xx
	                   xxx                                    xxxx
	                      xxxxxx                     xxxxxxxx
	                           xxxxxxxxxxxxxxxxxxxxxx


####chumby.com

	  ___
	 (O o)    chumby says, "I'm now at version 9139."
	 /||||\


####marrybacon.com

	 __       __                                            
	|  \     /  \                                           
	| $$\   /  $$  ______    ______    ______   __    __    
	| $$$\ /  $$$ |      \  /      \  /      \ |  \  |  \   
	| $$$$\  $$$$  \$$$$$$\|  $$$$$$\|  $$$$$$\| $$  | $$   
	| $$\$$ $$ $$ /      $$| $$   \$$| $$   \$$| $$  | $$   
	| $$ \$$$| $$|  $$$$$$$| $$      | $$      | $$__/ $$   
	| $$  \$ | $$ \$$    $$| $$      | $$       \$$    $$   
	 \$$      \$$  \$$$$$$$ \$$       \$$       _\$$$$$$$   
	                                           |  \__| $$   
	                   __      _.._             \$$    $$   
	                .-'__`-._.'.==.'.__.,        \$$$$$$    
	               /=='  '-._.'    '-._./ 
	              /__.==._.==._.'``-.__/  
	  _______     '._.-'-._.-._.-''-..'                   __ 
	 |       \                                           |  \
	 | $$$$$$$\  ______    _______   ______   _______    | $$
	 | $$__/ $$ |      \  /       \ /      \ |       \   | $$
	 | $$    $$  \$$$$$$\|  $$$$$$$|  $$$$$$\| $$$$$$$\  | $$
	 | $$$$$$$\ /      $$| $$      | $$  | $$| $$  | $$   \$$
	 | $$__/ $$|  $$$$$$$| $$_____ | $$__/ $$| $$  | $$   __ 
	 | $$    $$ \$$    $$ \$$     \ \$$    $$| $$  | $$  |  \
	  \$$$$$$$   \$$$$$$$  \$$$$$$$  \$$$$$$  \$$   \$$   \$$


####clover.com

	MMMM?..    .. ..MMMMMMMZ    ... . ~MMMMM
	MM: ,$$$$$$$$$$+. ZMD ..ZOZOOOOOO8= .NMM
	D..~$7$$$$$$$$$$7. I. ?OZZOOOOOOOOO8..ZM
	. ?77777$$$$$$$$$Z.  ZZZZZZZOOOOOOOOZ..Z
	..77777777$$$$$$$$+ :ZZZZZZZZZOOOOOOO:.~
	 .I7777777777$$$$$+ ~ZZZZZZZZZZZOOOOO, =
	= ,77777777777$$$$+ ~Z$$ZZZZZZZZZZZO+..M
	M. .7I7777777777$$+ ~Z$$$$ZZZZZZZZO7..NM
	MMM:..:I77777777$$+ ~Z$$$$ZZZZZZ=.. NMMM
	MMMMM+                           ~NMMMMM
	MMM~ .,?7I77777777+ ~$$$$$$$$$ZZ~ . MMMM
	M,..III+........77= ~$77$$$$$$$$$$$I  NM
	= ,II~.  ..  . .77= :$7777$$$$$$$$$$+ ,M
	 .??,   ... ....77= :7777777$$$$$$$$Z, +
	 .??.          .7I= :777777777$$$$$$Z: ~
	..=?=         .,I7. .77777777777$$$$7..Z
	D  ~??= . . .:II?. ? .+77777777777$$. ZM
	MM, ,??????I??I=  $MD...777777777$~..NMM
	MMMM+ ........ .MMMMMMM$...  .....:NMMMM
	MMMMMMMM$II7NMMMMMMMMMMMMMNZII$8MMMMMMMM


####lavraietimeline.fr

	
	   YGROUND                                                            
	   YGROUND                                                            
	              GRO                                                     
	             YGROU                                                    
	              GROUND       UNDP     OU                                
	                OUN     GROUNDP    ROUNDPL                            
	                         ROU          NDPL      NDPLAYGROUN           
	                                             ROUNDPLAYGROUNDPL        
	                  NDPLA                    YGROUNDPLAYGROUNDPLAY      
	                  NDPL                    AYGROUN         NDPLAYG     
	                                         LAYGRO            DPLAYGR    
	                                         LAYGRO             PLAYGR    
	                                         LAYGRO             PLAYGR    
	                                         LAYGRO             PLAYGR    
	                                          AYGROU           DPLAYG     
	                                          AYGROUND       UNDPLAY      
	                                            GROUNDPLAYGROUNDPLA       
	                                               UNDPLAYGROUND          
	                                                 DPLAYGRO
	
	    facelr 1.2 : facelr.tumblr.com
	    built autumn 2011 by playground : playgroundinc.com
	
	    cheer up, buddy!


####ddfhb.ie

	      DDDDDD
	      DDDDDD
	      DDDDDD
	BBBBBB      FFFFFF
	BBBBBB      FFFFFF
	BBBBBB      FFFFFF
	      HHHHHH
	      HHHHHH
	      HHHHHH

####118milano.it

	###################################################################
	#                                     Centrale Operativa 118 Milano    
	#            #####                                                     
	#    #      ##  ##                                                     
	#   #####       ##   #####                                             
	#  ##########   ###########   #####     #####   ##########             
	#   ##########   ##########  ######    ######  #############           
	#       ######   #####   ####################  #####    #####          
	#        ####   #####     ###################  #####    ######         
	#     #######   ########     ######    ######   ############           
	#  ###########   ##########  #######   ####### ######   ######  
	#  ######## ##  ##  #######  ######    ###### ######    ######         
	#   ####    ##  ##     ###   #######   #######  ##############         
	#           ######           ######    ######     ##########           
	#             
	#  Copyright © 2002-2006 Luca De Grazia
	####################################################################

####alcri.jp

	@@@@@
	@@@@@@@@@@@@@¡¡¡¡¡¡¡
	@@@@@@@@@@@¡¡¡¡¡¡¡¡¡¡¡
	@@@@@¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡
	@@@@@¡@@¡¡¡¡@@¡¡¡¡¡¡¡¡¡¡
	@@@@@@¡@¡¡¡¡@@¡¡¡¡¡¡¡¡¡¡
	@@@@@@@¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡
	@@@@@@@¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡
	@@@@@@@¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡
	@@@@@@@¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡
	@@@@@@@¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡
	@@@@@@@¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡
	@@@@@@@¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡
	@@@@@@@¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡
	@@@@@@@@¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡
	@@@@@@@@¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡
	@@@@@@@@@¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡¡
	@@@@@@@@@@@¡¡¡¡¡¡¡¡@@@¡¡¡¡


####gradx.net

	|                                                     N$$$$$   $$$$$
	|                                                      NIIIIIIIIIII.
	|   +MMMNMMM . DMM,MMMMN        FAY     MMM,MMM7        NIIIIIIII.      
	|  MMMM7?MMMM  DMM,MMMMMM      NMMM.    MMM,ELAINE       .IIIII8                 
	| MMM   .  MMM DMM,.  +MM.    MM MMM    MMM    MMM:       .IIIII8       
	| MMM          NMM,,,MMMM    +MM NMM~   MMM    :MMM      .IIIIIIII      
	| MMM.  NMMMMM.NMM,JACO~     MMM  MMM   MMM    :MMM      $IIN$IIIII.    
	| MMM  .   MMN DMM, MMM   . MMM MMMMMZ  MMM    MMM:     =IIN   IIIII8    
	|  MMMMMNMMMMM DMM,  MMMD  .MMN:::,MMM  MMM:LYALL+     .III=    OIIIII   
	|   SALKINDER  NMM,   MMMM.MMM.    ?MMM MMM:MMMN       $IIZ      LYALL


####hd-subs.com

	................................................................................
	................................................................................
	................................ #..............................................
	........... .... . . ... .......###..   . .. .  ..   .. .... . . ..... .........
	........... .... . ...~#.......... ....  ######=.. . ..#=... ... ...:#:.........
	........... .....  ########. ..... .. .#### .=### .. . ###:. ... .=######.......
	....... ......... ###....###.. .......=##. ....###=    ..###....:##:.. ~##,.....
	.....##......###..##...  .###. .###. ### ...... .##..  ...##=. .###   . =##.....
	.....=##=..~###...###..  .=#=.   #~..  ...   .    ...   . =##.  ###.    ###.....
	.......###=##... ..###..=###.. . ....   ..   .     ..   =##~    .###, :###......
	.... ....###...    .######~.   . ....   ..   .     .. .###.     . #######.......
	.......................................................#............  . ........
	........ .............................................  ........................
	........ .................................................. ....................
	........ .................................................. ....................
	........ ............................... .......................................
	                                                             02/2010 | voindo.eu


####onk.as


	                                     .=============.
	                                    /               \
	                                   / .=====.         \
	I am the Great Cornholio!!         |/ ==`-`-\         \
	                                   |         \        |
	I need TP for my bunghole!!         |   _==   \       |
	                                    _| =-.     |      |
	Come out with your pants down!      o|/o/      |      |
	                                    /  ~       |      |
	ARE YOU THREATENING ME??          (____@)  ___ |      |
	                                      _===~~~.`|      |
	Oh. heh-heh.  Sorry about that.   _______.==~  |      |
	                                   \_______    |      |
	heh-heh.  This is cool.  heh-heh        |  \   |      |
	                                         \_/__/       |
	                                       /            __\
	                                       -| Metallica|| |
	                                       ||          || |
	                                       ||          || |
	                                       /|          / /
	
	
	
	
	
	theme based on pouretrebelle work, â¬ hanks!



####mito.hu

	           ,.od88bo,                         .'; 
	         .'`      *88bn.,. _     ▄█              o   
	       ~              Y8b.▀▀     ██                    .
	    ▀███L▄████▄._▄█████▄.▄██ ██████████  ▄███████▄
	       ███▀   ?██▀    ██Bˆ██     ██    _██▀     ▀██,
	       ██      ██     ██8*██     ██   .██"        ██
	      ¸▀▀      ▀▀     ▀▀B*▀▀     ▀▀ Y* ▀▀         ▀▀
	       .    .          `*Y8bn.    ,dP
	        o$o*               'B8b, .+8   'ascii things
	         '                      ˇ          


####nybox.com
<div class="wide"><pre><code>

	                                                             ``````                            `++` 
	mNNNNNh.   `NNNNN-yNNNNN/   -mNNNNh:mmmmmmmmmmmmmmh/    .ohmMMMMMMMMmho. +mmmmmh-::          .sh-   
	MMMMMMMN+  .MMMMM. sMMMMM/ -NMMMMh`-MMMMMmddddNMMMMM:  oMMMMMNhyyhNMMMMMo .hMMMMMs/dy/`    :hN/     
	MMMMMMMMMh..MMMMM.  /MMMMMoNMMMMo  -MMMMM+////yMMMMN. :MMMMMd`    `dMMMMM:  /NMMMMN+oMNy/+mMs`      
	MMMMMomMMMN+MMMMM.   -mMMMMMMMN:   -MMMMMMMMMMMMMMMh- oMMMMM/      /MMMMMo   -MMMMMMd:hMMMd-        
	NMMMM:`sMMMMMMMMM.    `hMMMMMm.    -MMMMM/====:mMMMMN`:MMMMMh      hMMMMM: `yMMMMMMMMMs/d+          
	NMMMM:  -mMMMMMMM.     .MMMMM:     -MMMMMhhhhhdMMMMMm  sMMMMMmyssymMMMMMs`oNMMMMd/NMMMMN/           
	NNNNN:    oNNNNNN.     .NNNNN:     -MMMMMMMMMMMMNNdo`   -sdMMMMMMMMMMds-:mNNNNNo  `yNNNNNh.         
	                                                 ```..====::/+oosoo+////:::::==..``                 
	                          ``.-::/++ooooossssssssooo++///:::===....``````       ````````             
	          `.-:::://///+++++/::==.``                                                                 
	 ```....==.``  

</code></pre></div>

####cyber.kg

	                       oo                                    o.                   
	   888888oo            oo8                                   8o.                   
	 8888o   oo            o88                                   888                   
	888         88o    88o o88  .ooo.   .o88888.  oooo d8b       888   .88  .88888o88  
	88          888   o8o  8888o 8888  `8oo   888 `o88""8P       8oo .oo8  "oo8ooooo8  
	88           888  88o  888     8o  888    888  888           8888oo    8oo    oo8  
	888           88 8o8   888     8o  8o8oo88o8   888           8o88o8    8oo    oo8  
	 8888o  o8o    o8o8    8888   8o8   888        888     .o.   8o8 8ooo  8ooo   8o8  
	  `8888888o    8oo     `Y8bod8P'    `Yoooo8   d888b    Y8P   888  ooo8  o8oo888o8  
	              8o8                                                            8o8  
	            "oo8                                                         "8oo8   
	          `88"                                                        `8888"

####bubulounge.com.br

	  _____  
	 / ___ \__     ___  ___  _   _ _   _ _____ 
	| |   |   \   /___)/ _ \| | | | | | (___  )
	| | ! |    | |___ | |_| | |_| | |_| |/ __/
	| |___| __/  (___/ \___/ \__  |____/(_____)
	 \_____/                (____/      
	
	Desenvolvido em HTML5 por Soyuz Sistemas
	http://www.soyuz.com.br/


####doonited.com


	     `-:/++oossssssoo++/:-`    
	  -+ssssssssssssssssssssss+-  
	 /sssssssssssssooo++/+ssssss/ 
	.sssssssssssss+.`     sssssss.
	/sssssssssooooo/.     sssssss/
	osssssss+:.````/.     ssssssso
	sssssss/`    ./o.     ssssssss
	ssssss+    .+oss+     ssssssss
	osssss+    osssss`    ssssssso
	/ssssso-   .+oo+:     /osssss/
	.sssssso/.   ``..    `.osssss.
	 :sssssssoo+++ooo+++ooosssss: 
	  -+osssssssssssssssssssso+-  
	    `.://++oooooooo++//:.`


####pauseforlater.com
<div class="wide"><pre><code>

__________                                     _____                  .____              __                   
\______   \_____    __ __  ______  ____      _/ ____\____ _______     |    |    _____  _/  |_   ____ _______  
 |     ___/\__  \  |  |  \/  ___/_/ __ \     \   __\/  _ \\_  __ \    |    |    \__  \ \   __\_/ __ \\_  __ \ 
 |    |     / __ \_|  |  /\___ \ \  ___/      |  | (  &lt;_&gt; )|  | \/    |    |___  / __ \_|  |  \  ___/ |  | \/ 
 |____|    (____  /|____//____  &gt; \___  &gt;     |__|  \____/ |__|       |_______ \(____  /|__|   \___  &gt;|__|    
                \/            \/      \/                                      \/     \/            \/         

</code></pre></div>

####xpdev.com.br

	                \ \  / (_____ (____ \ (_______) |  | |
	                 \ \/ / _____) )   \ \ _____  | |  | |
	                  )  ( |  ____/ |   | |  ___)  \ \/ / 
	                 / /\ \| |    | |__/ /| |_____  \  /  
	                /_/  \_\_|    |_____/ |_______)  \/     


####ladiesland.de

	  ________________________________ __ ___
	 / __  __  __  __  __  __  __  __ \\_\___\__ ___  *  ___
	| |°°||°°||°°||°°||°°||°°||°°||°°| ||___|___|___|_  |___|
	| |°°||°°||°°||°°||°°||°°||°°||°°| ||_|___|___|___|_
	 \ \ `´ / |  `´  ||  `´  ||  `´  | ||_Y_|_U_|_H_|_U_|     |
	  \ |__|   \_____||__||__| \_____| ||_|___|___|___|     ==+==
	   \______________________________//___/___/___/          |


####artchipel.tumblr.com

	theme by pouretrebelle                           themes.pouretrebelle.com
	powered by Tumblr                                tumblr.com
	                      /\   _             _         _  _      
	 ___  ___  _ _  _ _  ___ _| |_ _ _  ___ | |_  ___ | || | ___ 
	| . \/ . \| | || '_>/ ._> | | | '_>/ ._>| . \/ ._>| || |/ ._>
	|  _/\___/`___||_|  \___. |_| |_|  \___.|___/\___.|_||_|\___.
	|_| 

####multirent.nl
<div class="wide"><pre><code>

                        Design &amp; Development door Netexpo Internet Projecten  
        
            `=:+/:.`         .//:    =/=
     `.:/+oooooo+/=`      :hhh:   oho             +y+
     :/+++++++++ooos/     :hysy=  oho   =++++/. =+yhs+=  .====.` `==`  .== `==.=::=`   `=====`      
     ::::::///++oyhho     :hy.yy. oho  /yo.`=yy.`=sho=. :/=``=/:  =/:`=/:` .//=``=//` .//.`.//.     
     ::::===:oooshhho     :hy =ys`oho  yh+:::yh/  sh+  .//...=//`  .///:`  .//   `//. //=   =/:     
     ::::====oooshhho     :hy  /h+oho `yh+:::::.  sh+  .//......   `///=   .//    //= //.   =//     
     ==::====oooshys:     :hy   oyyho  sh/   oo=  sh+  `//`  `::  `//.:/=  .//`  `//. :/=   :/:     
       `...==oo+/:.`      :yy   `syyo  .sy+/oy+`  +yyo: =/:==::. .//.  :/= .//:=://=  `:/:=:/:`     
           `.:.`          `.`    `..`   `.==.`     ...`  `````   ```    `` .//`````     ``.``       
                                                                           .//`                  
                                  Zie http://netexpo.nl/projecten/
                                  
                                         POWERED BY EXPOSER 2 CMS

</code></pre></div>

####exposer-cms.nl

	                                                                :,    ;,
	                                                                it;  jjj
	                                                                :tt;tjj:
	                                                              ittttGGjjtii
	                                                             .ttttDKKDttii.
	                                                                 iLLGGi
	                                                                tff:;LGt
	                                                                if,  iGi
	                          .i
	                         :KWL
	    iti.        ifj,    :iWWG,,     iff;     ;;     i.   i; :tLt        ;ffi
	 L#WWWWW#t   tWWWDD##G  #WWWWW#D ;WWWGDWWE.  E##: LW#t  DWWWWEWWWW,  ,#W#KK#W#t
	;WWK   jW#, :KWK   .##t  .KWL    K#W    W#j   LW#jG#:   DW#K   .#WD. WW#    WWW,
	tWWL   ,##i i#WWWWWWW#D  :KWL   ,#WWWWWWW#D    ,WW,     DW#j    KWK:iW#G    LWWt
	tWWL   ,##i ,WWE         :KWL   .WWW          jW#ff#.   DW#K    WWE..WWW    KWW;
	tWWL   ,##i  LWWEiiL##,   EWWGt  j#WKtiLW#i  D#W, G##;  DWWWKfGWWWi  i#WWDGWW#f
	iWWj   :##;   ,KWWWWW;     EWW#f  ,EWWWWWi  :##j   f#D  DW#jWWWWE.    .D#WWWE:
	                                                        DW#t
	                                                        DW#t
	                                                        ,KD.

	Design & Development door Netexpo Internet Projecten
	Zie http://netexpo.nl/projecten/

	POWERED BY EXPOSER 2 CMS


####factory-dev.hu

	11111111111111111111111111111111111111111111111111111111111111111111111111111111111
	11000000001111000011111110000001111000000000011110000001111100000001111100111100111
	11000111111111000011111100011100111111100111111100111100111100111000111110011000111
	11000111111110000001111100111111111111100111111100111100111100111100111110000001111
	11000000011110011001111000111111111111100111111000111100011100010001111111000011111
	11000111111100011000111000111111111111100111111000111100011100000011111111100111111
	11000111111100000000111100111111111111100111111100111100111100110001111111100111111
	11000111111000111100011100011100111111100111111100011000111100111000111111100111111
	11000111111001111110011110000001111111100111111110000001111100111100011111100111111
	11111111111111111111111111111111111111111111111111111111111111111111111111111111111
	Powered by FACTORY CREATIVE STUDIO Ltd || Please visit our website at netfactory.hu
	Thank You for visiting this Source Code!-->
	
	
####bubue.com

	                _=
	              #*#*
	             ####
	             ####*
	   _.''-===########==_
	                ' '
	
	   Bubue (c) 2010-2011


####darklordday.com

	Created by: Dan Nawara
	Released by: 3Floyd's Brewing
	
	                      uuuuuuu
	                  uu$$$$$$$$$$$uu
	               uu$$$$$$$$$$$$$$$$$uu
	              u$$$$$$$$$$$$$$$$$$$$$u
	             u$$$$$$$$$$$$$$$$$$$$$$$u
	            u$$$$$$$$$$$$$$$$$$$$$$$$$u
	            u$$$$$$$$$$$$$$$$$$$$$$$$$u
	            u$$$$$$"   "$$$"   "$$$$$$u
	            "$$$$"      u$u       $$$$"
	             $$$u       u$u       u$$$
	             $$$u      u$$$u      u$$$
	              "$$$$uu$$$   $$$uu$$$$"
	               "$$$$$$$"   "$$$$$$$"
	                 u$$$$$$$u$$$$$$$u
	                  u$"$"$"$"$"$"$u
	        uuu        $$u$ $ $ $ $u$$       uuu
	       u$$$$        $$$$$u$u$u$$$       u$$$$
	        $$$$$uu      "$$$$$$$$$"     uu$$$$$$ 
	      u$$$$$$$$$$$uu    """""    uuuu$$$$$$$$$$
	      $$$$"""$$$$$$$$$$uuu   uu$$$$$$$$$"""$$$"
	       """      ""$$$$$$$$$$$uu ""$"""
	                 uuuu ""$$$$$$$$$$uuu
	        u$$$uuu$$$$$$$$$uu ""$$$$$$$$$$$uuu$$$
	        $$$$$$$$$$""""           ""$$$$$$$$$$$"
	         "$$$$$"                      ""$$$$""
	           $$$"                         $$$$"


####theatrics.com

	
	                              .o88o.
	                              o  '88  
	   8888o                88    '  d87     .88b<>        88     88
	   88 }8                88      '''      48 '          88     88
	   8888'   .o8o.  .o8o. 88.87 88 8848o. 8888 88  .o8o. 88  40V88
	   88888b 48...7 48/  ' 888{  88 88'`88  88  88 48...7 88 887'88
	   88  }8 Y8   . Y8b  . 88`8' 88 88  88  88  88 Y8   . 88 Y{  88
	   888887  Y8887  Y8887 88 `8'88 88  88  88  88  Y8887 88  Y8788
	 
	   M  A  S  S      P  A  R  T  I  C  I  P  A  T  I  O  N     T V


####compuccino.com

	|
	| Design + Umsetzung von 
	|                                                    _             
	|   _____ ____   ____ ___   ____   __  __ _____ _____ (_)____   ____ 
	|  / ___// __ \ / __ `__ \ / __ \ / / / // ___// ___// // __ \ / __ \
	| / /__ / /_/ // / / / / // /_/ // /_/ // /__ / /__ / // / / // /_/ /
	| \___/ \____//_/ /_/ /_// .___/ \__,_/ \___/ \___//_//_/ /_/ \____/ 
	|                      /_/
	|
	| http://compuccino.com
	| 
	| Wir haben auch Platz fÃ¼r Dich:
	| http://compuccino.com/jobs
	|



####lektronix.net

	    __         __   __                   _     
	   / /   ___  / /__/ /__________  ____  (_)  __
	  / /   / _ \/ //_/ __/ ___/ __ \/ __ \/ / |/_/
	 / /___/  __/ ,< / /_/ /  / /_/ / / / / />  <  
	/_____/\___/_/|_|\__/_/   \____/_/ /_/_/_/|_| 


####buzzboot.com

	       ______                       __                  ___ 
	      ||  __  \                    || |\              __| |\__ 
	      || ||_|  | __  _  ____  ____ || |_|_ ______ ___|__   __|\ 
	      ||  ___  \|| || |||_  |||_  ||| _  ||| _  ||| _  || ||\_\|
	      || ||_|| ||| || ||// /_// /_||| || ||| || ||| || || |||  
	      ||_______|||____||____||_____||____//|____|||____||_||| 
	       \_________\______\_____\_____\_____/\______\_____\__\| 


