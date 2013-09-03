---
layout: post
category : Programming
tags : [vector, tangent, normal, Gloss]
title: Generating Tangents From A Normal
---
{% include JB/setup %}

###Edit: As pointed out by several helpful readers, my original method was broken.

Sorry for spreading confusion! I'm leaving the entire post up, together with this fix. I suppose that is what you get for coding by wistful thinking.

	Vector vTangent(const Vector normal) {
		
		return vCross(normal, makeVector(
			-normal.z,
			 normal.x,
			 normal.y
		));
	}

The cross product guarantees the result is a tangent, so I couldn't just eliminate it. Still, it is far more elegant than normalizing a random vector.

Some readers pointed out that there are infinitely many tangents, and that is true. My code just needed *any* tangent, so this function generates *a* tangent, not *the* tangent.

<hr>

I was hacking on some 3D math code where I needed a tangent vector of a surface normal. I had used a rather convoluted method of generating one, where I used the cross product of the normal and another, arbitrary vector. That would be fine, except I didn't want to use a hard coded vector like (1, 0, 0), since it could be identical to the surface normal, in which case the resulting tangent would be garbage. My stupidly slow solution had been to generate a random unit vector by generating 3 random numbers for a coordinate, and normalizing it. There was still a chance it could be identical to the normal, but at least the error would be spread evenly...

<del>
Then, when seeing this mess, I realized I could just jumble the axes of the normal! The new code is extremely simple:
</del>

	// BROKEN CODE!!!

	Vector vTangent(const Vector v) {
		
		return makeVector(
			-v.z,
			 v.x,
			 v.y
		);
	}

<del>
As you can see, the order of the axes are rotated, and negated when wrapping around. You can apply it again to the result for two tangents. Calling the function repeatedly has a periodicity of 6, then you get the original vector back:
</del>

	 a, b, c
	-c, a, b
	-b,-c, a
	-a,-b,-c
	 c,-a,-b
	 b, c,-a

<del>
Perhaps this is facepalm obvious to everyone except me, or it isn't something a lot of people find useful. Either way, I couldn't find any reference to this on the web, and I lack the mathematical skill to prove that this actually works for all vectors. But I did try it out with my 3D renderer [Gloss](http://geon.github.io/programming/2013/08/22/gloss-my-bidirectional-path-tracer/), and it seems to work perfectly.
</del>