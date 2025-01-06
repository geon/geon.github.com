---
layout: post
category : Programming
tags : [dyslexia, typoglycemia, Javascript]
title: Dsxyliea
---


A friend who has dyslexia described to me how she experiences reading. She *can* read, but it takes a lot of concentration, and the letters seems to "jump around".

I remembered reading about [typoglycemia](https://en.wikipedia.org/wiki/Typoglycemia). Wouldn't it be possible to do it interactively on a website with Javascript? Sure it would.

Feel like making a bookmarklet of this or something? [Fork it](https://github.com/geon/geon.github.com/blob/master/_posts/2016-03-03-dsxyliea.md) on github.

> Dyslexia is characterized by difficulty with learning to read fluently and with accurate comprehension despite normal intelligence. This includes difficulty with phonological awareness, phonological decoding, processing speed, orthographic coding, auditory short-term memory, language skills/verbal comprehension, and/or rapid naming.

> Developmental reading disorder (DRD) is the most common learning disability. Dyslexia is the most recognized of reading disorders, however not all reading disorders are linked to dyslexia.

> Some see dyslexia as distinct from reading difficulties resulting from other causes, such as a non-neurological deficiency with vision or hearing, or poor or inadequate reading instruction. There are three proposed cognitive subtypes of dyslexia (auditory, visual and attentional), although individual cases of dyslexia are better explained by specific underlying neuropsychological deficits and co-occurring learning disabilities (e.g. attention-deficit/hyperactivity disorder, math disability, etc.). Although it is considered to be a receptive language-based learning disability in the research literature, dyslexia also affects one's expressive language skills. Researchers at MIT found that people with dyslexia exhibited impaired voice-recognition abilities.

*Source: [Wikipedia](http://en.wikipedia.org/wiki/Dyslexia)*




<script type="text/javascript" src="//cdnjs.cloudflare.com/ajax/libs/jquery/2.0.3/jquery.min.js"></script>
<script type="text/javascript">

"use strict";

$(function(){

	var getTextNodesIn = function(el) {
	    return $(el).find(":not(iframe,script)").addBack().contents().filter(function() {
	        return this.nodeType == 3;
	    });
	};

	// var textNodes = getTextNodesIn($("p, h1, h2, h3"));
	var textNodes = getTextNodesIn($("*"));



	function isLetter(char) {
		return /^[\d]$/.test(char);
	}


	var wordsInTextNodes = [];
	for (var i = 0; i < textNodes.length; i++) {
		var node = textNodes[i];

		var words = []

		var re = /\w+/g;
		var match;
		while ((match = re.exec(node.nodeValue)) != null) {

			var word = match[0];
			var position = match.index;

			words.push({
				length: word.length,
				position: position
			});
		}

		wordsInTextNodes[i] = words;
	};


	function messUpWords () {

		for (var i = 0; i < textNodes.length; i++) {

			var node = textNodes[i];

			for (var j = 0; j < wordsInTextNodes[i].length; j++) {

				// Only change a tenth of the words each round.
				if (Math.random() > 1/10) {

					continue;
				}

				var wordMeta = wordsInTextNodes[i][j];

				var word = node.nodeValue.slice(wordMeta.position, wordMeta.position + wordMeta.length);
				var before = node.nodeValue.slice(0, wordMeta.position);
				var after  = node.nodeValue.slice(wordMeta.position + wordMeta.length);

				node.nodeValue = before + messUpWord(word) + after;
			};
		};
	}

	function messUpWord (word) {

		if (word.length < 3) {

			return word;
		}

		return word[0] + messUpMessyPart(word.slice(1, -1)) + word[word.length - 1];
	}

	function messUpMessyPart (messyPart) {

		if (messyPart.length < 2) {

			return messyPart;
		}

		var a, b;
		while (!(a < b)) {

			a = getRandomInt(0, messyPart.length - 1);
			b = getRandomInt(0, messyPart.length - 1);
		}

		return messyPart.slice(0, a) + messyPart[b] + messyPart.slice(a+1, b) + messyPart[a] + messyPart.slice(b+1);
	}

	// From https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math/random
	function getRandomInt(min, max) {
		
		return Math.floor(Math.random() * (max - min + 1) + min);
	}


	setInterval(messUpWords, 50);
});


</script>
