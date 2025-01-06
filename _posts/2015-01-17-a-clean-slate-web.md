---
layout: post
category : Programming
tags : [web, ccs, javascript]
title: A Clean Slate Web
---


This is going to be a bit of a rant. Sorry.


The Rant
--------
TL;DR: Crufty layers sucks. Let's talk about it.

I have thought a lot about the horribly byzantine state of the modern web. Early bad decisions are still hurting us, as backwards compatibility have become so holy that it would be inconceivable to fix them.

It doesn't help that the web has grown organically, driven by culture, technology and commercial interests into something completely diffrent from what was originally envisioned. In a way, the epitome of feature creep and last minute spec changes.

The entire web stack is just layer upon layer of bad design and re-implementations to fix what the lower layers got wrong in the first place.

My favorite part is CSS. Bret Victor [expressed it very nicely](http://worrydream.com/MagicInk/#p255):

> CSS, a language for specifying visual appearance on the web, is … so complex that it has never been implemented correctly; yet, successive versions specify even more complexity. At the same time, it is so underpowered that many elementary graphic designs are impossible or prohibitively difficult

In case you don't write a lot of CSS, have look at this:

	.collumns {
		overflow: auto;
	}
	.collumn {
		float: left;
	}

Two rules, the first for making a block have scrollbars if the content is too large, and the second for making text flow around a smaller, nested block, like an image or quote. Together, they were until very recently (ie10) the idiomatic way of placing multiple blocks next to each other. 

Seriously.

People have wanted this feature since the late nineties, and it took Microsoft until 2012 to have it implemented in Internet Explorer.

The entirety of CSS is just full of side effect like this. The language is deceptively simple. But getting a browser to display your design the way you want is *really hard*. Being a front-end web developer (among other things), I cry when I think of how much time I have spent fighting browsers. It's sad when your professional life can be summed up in a [mug](http://www.zazzle.com/css+is+awesome+mugs).

Then we have Javascript. It gets a lot of hate for being written in 10 days for making a monkey dance when you hovered over it. I won't go into that here. Some is justified, other points is mostly language flamewar. I write a lot of JS, and I even kind of like it. You run into the gotchas and limitations sometimes, but it is still a powerful language that lets you write good code with some discipline.

Some of the worst issues are addressed in the upcoming version ES6. But since they fix the very foundation of the language, and you still must have that oh-so-holy backward compatibility you end up with new, duplicate syntax for everything.

The bigger problem is in how JS gets used. I won't complain about bad web designers using it to "fix" their broken CSS. That's just bad developers. No, I'm thinking about how it's used to paper over the broken foundation below.

We have frameworks like React, that *[re-implement](http://calendar.perfplanet.com/2013/diff/)* the DOM structure, including event handling, in Javascript. Because it's faster. Think about that for a moment. But why stop there? The [React Canvas](http://engineering.flipboard.com/2015/02/mobile-web/) project implements also the rendering, and parts of CSS to completely circumvent the browser. You just built a browser in Javascript, running inside the browser.

Meanwhile, Epic Games is compiling their Unreal game engine, written in C++, to JS. The output is pretty much the equivalent of machine code, manipulating a large array of integers. And because of the enormous resources spent on optimizing the various JS runtimes, it's actually fast. Like half native fast.

That's amazing. In a perverted kind of way.


So What?
--------

What would it look like if we designed the web stack from the ground up today? What would we do diffrent? What did they get right from the beginning? For what it's worth, Stanford asked themselves [the same thing](http://cleanslate.stanford.edu/) about the internet.

Let's have that discussion! Please leave a comment.

### Talking Points

#### Let the document format be for documents!
Could we make the HTML/DOM/CSS/JS environment optional? Could something like NaCl run any type of client natively, where the browser is just a sandbox? And for actual documents, why not send Markdown instead. A Markdown environment would be able to render that just as well (or better) as a current browser renders HTML.

#### What would a better layout language look like?
Should it be turing complete? Is it ever enough to just expose settings? What about something like Latex? Or [Auto Layout](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/Introduction/Introduction.html)?

#### Why JS?
I should be able to write my next web framework in any language I feel like. Having only JS is riddiculous. What is needed to make this a reality?

#### Networking protocols!
HTTP2 Looks interesting. P2P between browsers sounds neat too.
