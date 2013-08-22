---
layout: post
category : Programming
tags : [ray tracing, bidirectional path tracing, Gloss, Cornell box]
title: Gloss - My Bidirectional Path Tracer 
---
{% include JB/setup %}

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

I wrote the first version of [Gloss](https://github.com/geon/gloss) on a vacation 3 years ago. It was written in C++, and I managed to render some nice Cornell boxes before code became messy I lost interest. This summer, I decided to rewrite it in C. I have been interested in pure C lately, and I really need to level up my experience.

Porting over the bulk of the code from C++ was relatively easy. Getting it to compile took some time though. I wanted to use a object oriented style with constructors and polymorphism. Since it isn't supported directly, you are free to implement it as you feel like. It is sort of refreshing, actually. I also had to build some equivalent to the C++ container classes I used before. I was looking at [Cello](http://libcello.org/), a C library for higher level programming. In the end, I felt it was overkill for my needs. Besides, I wanted to build it for the learning experience. My solution is a few macros that define functions to manipulate an array encapsulated in a struct.

So anyway, I want to share some images from the development.

<div class="image-box">
	<img src="/assets/posts/2013-08-22-gloss-my-bidirectional-path-tracer/01 - First render.png">
	<i>The first render</i>
</div>

Getting it to compile and starting to plot pixels the first time was fun. The image is pretty much garbage, but you can make out the sides of the Cornell box, the light source and a sphere on the floor. Not bad, considering that the value returned from the BRDF function was undefined.

You can see the sky color peeking through the box in the middle. A bug caused the far wall of the box to be clipped.

<div class="image-box">
	<img src="/assets/posts/2013-08-22-gloss-my-bidirectional-path-tracer/02 - Got the full Cornell box working.png">
	<i>Got the full Cornell box working</i>
</div>

The backside of the Cornell box is back, and the colors changed for some reason.

<div class="image-box">
	<img src="/assets/posts/2013-08-22-gloss-my-bidirectional-path-tracer/03 - With the blocks.png">
	<i>With the blocks</i>
</div>

I implemented the box object so I could build the complete Cornell box.

<div class="image-box">
	<img src="/assets/posts/2013-08-22-gloss-my-bidirectional-path-tracer/04 - The photons seems to be spawned unevenly from the sphere, and I probably should implement the BRDF.png">
	<i>Time to implement the BRDF?</i>
</div>

The BRDF shading function wasn't even implemented, so nothing would look remotely OK until it was done. You can also make out some weird artifacts in this image, like the circular shadow in the right wall.

<div class="image-box">
	<img src="/assets/posts/2013-08-22-gloss-my-bidirectional-path-tracer/05 - Diffuse shading.png">
	<i>Diffuse shading</i>
</div>

As soon as the diffuse shading BRDF function worked, the scene looked a lot more realistic. Even without the photons bouncing correctly, it clearly shows the scene and the lighting.

<div class="image-box">
	<img src="/assets/posts/2013-08-22-gloss-my-bidirectional-path-tracer/06 - Correctly bouncing photons.png">
	<i>Correctly bouncing photons</i>
</div>

Photons are supposed to bounce around in the scene for the Global Illumination to work. The direction in which they bounce mirrors the BRDF function, but instead of modulating the amount of light passed through, it is the probability to bounce in a certain direction that is affected. For a 100% diffuse material, a photon should be able to bounce with equal probability in any direction over the hemisphere of the normal of the surface. As soon as I had it working, it looked pretty much perfect.

<div class="image-box">
	<img src="/assets/posts/2013-08-22-gloss-my-bidirectional-path-tracer/07 - More global illumination.png">
	<i>More global illumination</i>
</div>

I did some artistic tweaking of the scene parameters to make the Global Illumination effects stand out more. I increased the reflectivity of all materials so the secondary light wouldn't be absorbed so quickly, and tuned down the light source a bit to match.

<div class="image-box">
	<img src="/assets/posts/2013-08-22-gloss-my-bidirectional-path-tracer/08 - Yeah, the photons are NOT spawned evenly.png">
	<i>Yeah, the photons are NOT spawned evenly</i>
</div>

I suspected the photons were spawned unevenly from the spherical light source, and decided to investigate it. I tried hard to find some tool that could plot points in 3D, but they all seemed to have a pretty steep learning curve or didn't really offer what I needed. Then I realized it would be really easy to use my own renderer to plot them. Doh!

And indeed, the photons didn't spawn correctly at all.

<div class="image-box">
	<img src="/assets/posts/2013-08-22-gloss-my-bidirectional-path-tracer/09 - That's really even.png">
	<i>That's really even</i>
</div>

The upside was that I got the opportunity to implement stratified sampling, something my old C++ renderer was missing. Purely randomized samples tend to cluster, which creates noise. That's a bit counter intuitive, but you don't really want to sample randomly - you want the samples to be spread evenly. The idea is to place the samples on a grid rather than just randomly, and then offset each sample within it's cell.

<div class="image-box">
	<img src="/assets/posts/2013-08-22-gloss-my-bidirectional-path-tracer/10 - With random jitter, there is no visible pattern.png">
	<i>With random jitter, there is no visible pattern</i>
</div>

This is the same grid as above, but with random jitter. As you can see, the grid pattern disappears completely. You might wonder why the points don't cluster around the poles of the sphere since I clearly use polar coordinates. The solution is very simple, don't simply plot the (u,v) coordinates, but (u,acos(v)).

<div class="image-box">
	<img src="/assets/posts/2013-08-22-gloss-my-bidirectional-path-tracer/11 - Exact number of photons spawned, note the top ring.png">
	<i>Exact number of photons spawned, note the top ring</i>
</div>

Since you can't divide 99 samples evenly over an n\*m grid, spawning a fixed number of photons is a bit tricky. I could cheat and spawn slightly more or less photons than requested, but I thought I could do better.

My solution was to use fewer samples in the last row and make that row slightly thinner to compensate. Look at the top ring in the picture above. It has about half as many photons as the other rings, spread over a proportionally smaller area.

<div class="image-box">
	<img src="/assets/posts/2013-08-22-gloss-my-bidirectional-path-tracer/12 - So smooth, 256 samples per pixel.png">
	<i>So smooth, 256 samples per pixel</i>
</div>

And it helped! Now the light looks really even. I also let the rendering run it run a bit longer. It is butter smooth after 256 samples per pixel.

Next up: more materials?
