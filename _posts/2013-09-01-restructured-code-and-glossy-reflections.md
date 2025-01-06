---
layout: post
category : Programming
tags : [glossy reflections, Gloss, Cornell box]
title: Restructured Code And Glossy Reflections
---


<style type="text/css">

	div.image-box {

		width: 256px;

		margin: 2em auto;
	}

	div.image-box i {

		display: block;
		text-align: center;
	}

</style>

The last week or so, I have been hacking away on my bidirectional path tracer [Gloss](https://github.com/geon/gloss). After [porting some old C++ code to C](http://geon.github.io/programming/2013/08/22/gloss-my-bidirectional-path-tracer/) and cleaning it up, I wanted to add support for materials. I had already tried to structure the code in an object oriented way and I used polymorphism to handle the diffrent renderable object types. But the object model was a clunky, quickly thrown together mess, so I decided to get [something more proper](http://geon.github.io/programming/2013/09/01/object-oriented-c/) running.

With the new object model, separating the Material class into independent sub classes was relatively straight forward.

<div class="image-box">
	<img src="/assets/posts/2013-09-01-restructured-code-and-glossy-reflections/01 - Almost perfect mirrors.png">
	<i>Almost perfect mirrors</i>
</div>

The first new feature I added was a material based on the Phong equation. Unlike the diffuse material that scatters light equally in all directions (Actually, [not *equally*](http://en.wikipedia.org/wiki/Diffuse_reflection) it seems. I have better fix that.), the Phong material also has a [specular](http://en.wikipedia.org/wiki/Specular_reflection) component, meaning that light is reflected like in a mirror surface, or like a pool ball against the sides of a pool table.

<div class="image-box">
	<img src="/assets/posts/2013-09-01-restructured-code-and-glossy-reflections/02 - Glossy boxes after 56 samples per pixel.png">
	<i>Glossy boxes after 56 samples per pixel</i>
</div>

<div class="image-box">
	<img src="/assets/posts/2013-09-01-restructured-code-and-glossy-reflections/03 - Foggy mirrors after 64 passes.png">
	<i>Foggy mirrors after 64 passes</i>
</div>

The fun part starts when you add an adjustable parameter for how imperfect that reflection is. The probabilities form a more or less tight lobe around the direction of perfect reflection. That is called glossy reflections.

Fun fact: In traditional [Phong shading](http://en.wikipedia.org/wiki/Phong_shading), the specular high light is actually a faked effect to simulate the glossy reflection of a point light source. In my renderer, you also get the proper glossy reflection of the environment "for free".

Somewhere around here, while doing a slight overhaul of the lighting system, I noticed the photons would not be emitted properly. Instead of having a randomized direction, they would follow the surface normal. For a spherical lamp, that is equivalent to a point light source. The shadows in the images so far are still soft, which I think is because of the light reflected in the ceiling.

<div class="image-box">
	<img src="/assets/posts/2013-09-01-restructured-code-and-glossy-reflections/04 - Without diffusely emitted photons.png">
	<i>Without diffusely emitted photons</i>
</div>

<div class="image-box">
	<img src="/assets/posts/2013-09-01-restructured-code-and-glossy-reflections/05 - Square light source.png">
	<i>Fixed. Also a square light source.</i>
</div>

You can see the lower image has far softer shadows. In part it is because a new bug that I introduced when I rebuilt the lighting system to handle light sources through generalized materials, instead of a special purpose member variable. Combined with a bug in the object transformation I used to scale the square light source, It made any light source too dim, while making the photons emitted too bright. The result is less "weight" on the direct light.

<div class="image-box">
	<img src="/assets/posts/2013-09-01-restructured-code-and-glossy-reflections/06 - The square light source is too dark in the reflection.png">
	<i>The square light source is too dark in the reflection</i>
</div>

<div class="image-box">
	<img src="/assets/posts/2013-09-01-restructured-code-and-glossy-reflections/07 - No matter how bright.png">
	<i>No matter how bright</i>
</div>

You can see the square light source in the reflection of the sphere, but it is *darker* than the surrounding ceiling. That is just wrong. Even turning up the brightness a lot didn't make the light source look bright in the reflection.

<div class="image-box">
	<img src="/assets/posts/2013-09-01-restructured-code-and-glossy-reflections/08 - Increasing the size of the light source doesn't affect the scene brightness.png">
	<i>Increasing the size of the light source doesn't affect the scene brightness</i>
</div>

<div class="image-box">
	<img src="/assets/posts/2013-09-01-restructured-code-and-glossy-reflections/09 - Nor does decreasing it, but it seems to absorb less of the colored light.png">
	<i>Nor does decreasing it, but it seems to absorb less of the colored light</i>
</div>

An even weirder bug was that changing the size of the light source did nothing to the overall scene brightness, even though the lighting overhaul was supposed to handle exactly that.

It turned out I needed to adjust the energy of emitted photons and the radiant flux of light source objects when using a scaling transformation.

Also, the code to extract the scale factor from a transformation matrix was all wrong. It seems like the cube root of the top left 3x3 sub matrix determinant is the scale factor, but I have no better basis for this than hearsay. Either way, extracting a scale factor from a arbitrary transformation matrix seems a bit like a hack. Perhaps storing the scale factor, rotation and translation separately would be just as efficient.

<div class="image-box">
	<img src="/assets/posts/2013-09-01-restructured-code-and-glossy-reflections/10 - Oh, look at that highlight, 590 samples per pixel.png">
	<i>Oh, look at that highlight, 590 samples per pixel</i>
</div>

The sessions of confused debugging was well worth it, though. Just look at that yummy, yummy high light!

<div class="image-box">
	<img src="/assets/posts/2013-09-01-restructured-code-and-glossy-reflections/11 - All together now! 425 passes.png">
	<i>All together now! 425 passes.</i>
</div>

The Cornell box looks pretty nice now. The downside is that the bright direct light introduces a lot of noise when rendering, so a lot more samples per pixel are necessary to get the same image quality smoothness-wise. Oh, well.

Next up: Refraction and dispersion?
