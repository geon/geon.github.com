---
layout: post
category : Programming
tags : [web, ccs, less, jade]
title: Flexbox Full Page Web App Layout
---


I was working on a web app layout, and since we require a modern browser (ie10+), I had the chance to use Flexbox. Yay!

The layout was designed by someone who apparently don't like the way the web tend to work, with unlimited page height. Instead the app is supposed to cover the entire window without scrolling, and have a footer stuck to the bottom of the screen. There are also sidebars with tools, and a 3D view in the center.

If the content in the sidebars it too large to fit in the window, it is important that the footer is not pushed down to make room, but that the sidebar *and only the sidebar* overflows and scrolls.

Let's say we have this basic document structure.

```html
<div class="full-screen">
	<div class="header">Header</div>
	<div class="main">
		<div class="left">Menu-stuff here. No way this will fit.
			<ul>
				<li>Menu Entry</li>
				<li>Menu Entry</li>
				<li>Menu Entry</li>
				<li>Menu Entry</li>
				<li>Menu Entry</li>
				<li>Menu Entry</li>
				<li>Menu Entry</li>
				<li>Menu Entry</li>
				<li>Menu Entry</li>
				<li>Menu Entry</li>
				<li>Menu Entry</li>
				<li>Menu Entry</li>
				<li>Menu Entry</li>
				<li>Menu Entry</li>
				<li>Menu Entry</li>
				<li>Menu Entry</li>
				<li>Menu Entry</li>
			</ul>
		</div>
		<div class="middle">Large stuff here</div>
		<div class="right">Another sidebar</div>
	</div>
	<div class="footer">Footer</div>
</div>
```

The first issue to take care of is making the `full-screen` div cover the full window exactly. It's pretty easy, but should be taken care of first, since the entire layout could break if you add it later.

...Which has happened to me. Sigh.

You need to set `height: 100%` on `full-screen`. But that isn't enough. As far as CSS is concerned, the `BODY` element surrounding the div of the document doesn't have a height on it's own, so you need the 100% height there too. Same thing with the `HTML` element surrounding *that*. The `BODY` also has a margin we need to get rid of.

```less

html,
body,
.full-screen {
	height: 100%;
}

body {
	margin: 0;
}
```

The flexbox-y stuff is pretty simple when you get it. Basically, you set `display: flex;` on a container, and optionally specify the `flex-grow` and `flex-shrink` on the children. If you want to set a fixed, explicit width on a child, you need to prevent it from flexing the width by setting both properties to `0`. (You can use the `flex` shorthand.)

To make a column of stuff stacked vertically, set `flex-direction: column;` on the container.

It's all in [this exellent guide](https://css-tricks.com/snippets/css/a-guide-to-flexbox/).

This is going to be pretty repetitive, tedious and error prone if you write it in plain CSS. Just like all styling. Don't do that. Use a preprocessor, like Less.

Let's make the outermost div a flexbox. The children will have a fixed height unless explicitly marked to use `flex-grow` (and `flex-shrink`) to fill the available space.

```less

// A container using the flexbox layout.
.flex-container {

	// See? Simple.
	display: flex;

	// Set all direct children to NOT flex...
	> * {

		flex: 0 0 auto;			
	}  
}

// ...unless explicitly marked as stretchy.
// Obviously, you can do it the opposite way.
.stretchy {
	
	flex: 1 1 auto;
}

// The .flex-container can be a column instead of a row.
.column {

	flex-direction: column;
}
```

Apply them as mixins to the `full-screen`, and add sizing to the children:

```less
.full-screen {
	
	.flex-container;
	.column;
	
	> .header,
	> .footer {
		height: 50px;
	}

	> .main {
		.stretchy;
	}
}
```

You need to do the same for every div you want to behave like this.

```less
.full-screen {
	
	.flex-container;
	.column;
	
	> .header,
	> .footer {
		height: 50px;
	}

	> .main {
		.stretchy;

		// See? It's recursive, but this time a row.
		.flex-container;

		> .left,
		> .right {
			width: 200px;
		}

		> .middle {
			.stretchy;
		}
	}
}
```

<p data-height="268" data-theme-id="0" data-slug-hash="dGBvej" data-default-tab="result" data-user="geon" class='codepen'>See the Pen <a href='http://codepen.io/geon/pen/dGBvej/'>dGBvej</a> by Victor Widell (<a href='http://codepen.io/geon'>@geon</a>) on <a href='http://codepen.io'>CodePen</a>.</p>

This still isn't working, though. Large content in `.main` will push down the footer of the page, instead of causing scrolling. (Or in an iframe like this Codepen, push the content down below the footer.) Adding `overflow: scroll;` to `.left`, `.right` and `.middle` is necessary, but not enough. We also need to make the parent stop trying to accommodate them, by setting `overflow: hidden;` on `.main`. I have no idea *why* that works, and the specs says nothing either, but the behavior is consistent over Chrome, Safari, Firefox and Explorer.

Let's add that to `.flex-container`.

```less
.flex-container {

	display: flex;
	overflow: hidden;

	> * {

		flex: 0 0 auto;			
		overflow: scroll;
	}
}
```

<p data-height="268" data-theme-id="0" data-slug-hash="obrWxm" data-default-tab="result" data-user="geon" class='codepen'>See the Pen <a href='http://codepen.io/geon/pen/obrWxm/'>obrWxm</a> by Victor Widell (<a href='http://codepen.io/geon'>@geon</a>) on <a href='http://codepen.io'>CodePen</a>.</p>

Notice how the left menu column scrolls independently, and the footer is locked to the bottom of the Codepen iframe.




<script async src="//assets.codepen.io/assets/embed/ei.js"></script>
