---
category : Programming
tags : [object orientation, C, Gloss]
title: Object Oriented C
---



Disclaimer
----------

I am a C noob. Everything I write in this article is likely wrong, stupid and/or harmful. That being said...


Objects in C? Just... Why?
--------------------------

OO is just one of the tools in the toolbox. It is nice to have sometimes. That it isn't built into the language doesn't mean it can't be done.

You will have to rely a lot more on the programmer to follow convention, rather than having the compiler do it for you, which obviously is a source of possible errors. One example is how C++ calls the destructor at the end of the scope. That can't really be done in C. On the other hand Objective C has destructors, but on iOS you had to call them semi-manually until version 6.

I was working on my bidirectional path tracer, [Gloss](https://github.com/geon/gloss). I needed to implement materials, and polymorphism is a great too for that. I already had it running for the different renderable geometry object types, but the implementation was increasingly limiting, so I wanted something better before going forward.


Stack vs. Heap
--------------

When you build your OO by yourself, you have a few choices to make.

Objective C makes all objects be allocated on the heap and reached through pointers (with a few [weird exceptions](http://stackoverflow.com/questions/2659072/copying-blocks-ie-copying-them-to-instance-variables-in-objective-c)). It makes it very consistent, because objects are always accessed through pointers (except when they aren't). In C, that also means you can forward declare the object type and only have the actual struct visible to the implementations file, achieving encapsulation, and preventing cyclic dependencies.

I wanted my objects to stay on the stack as much as possible. I'm not sure I have good reasons for this, but at least you don't have to deal with allocation and deallocation. Please chime in if you know better.

If you put the base class struct at the beginning of your sub class struct, you can just cast your sub class structs to the base class to pass them to functions on the stack. However (and correct me if I'm wrong), the argument will be truncated to the size of your base class, so polymorphism kind of breaks down.

I believe in a certain level of Worse Is Better and KISS, so my first object system kept all member variables in the base struct, where sub classes could access them. Being able to pass polymorphic objects around on the stack seemed like a great idea at first. There is the obvious downside that all objects take up the same memory as your largest one, but since I didn't plan to have any large objects, it could have been acceptable.

But the real show stopper was that I couldn't have a tree-like structure of objects, like required for doing [CSG](http://en.wikipedia.org/wiki/Constructive_solid_geometry). 

I also had to do transformations back and forth for all objects, even the ones that didn't need it. Someone on Github actually hacked on my code and [noticed that matrix operations was a huge bottleneck](https://github.com/giannitedesco/gloss/commit/689dbfcc46ba1005d3db667bd1b43f6b23be9926). Having a tree structure for the scene, and only transforming the ones that needs it should help.


Conclusion
----------

My current object model looks something like this:


### Material.h

	struct MaterialVTableStruct {
		Photon (*materialSampleBRDF)(const Material *material, const Intersection intersection, const Photon incoming);
		/* ...and so on */
	};

	struct MaterialStruct {
		const MaterialVTable *vTable;
	};

	Material makeMaterial(const MaterialVTable *vTable);

	Photon materialSampleBRDF(const Material *material, const Intersection intersection, const Photon incoming);

### Material.c

	#include "Material.h"

	Material makeMaterial(const MaterialVTable *vTable) {

		return (Material) {vTable};
	}

	Photon materialSampleBRDF(const Material *material, const Intersection intersection, const Photon incoming) {
		
		return material->vTable->materialSampleBRDF(material, intersection, incoming);
	}

As you can see, the implementation of `materialSampleBRDF` is just a convenient wrapper around a call to the function in the "v table". I chose to use a struct with named members instead of an array, since it IMHO is cleaner.

### MaterialDiffuse.h

	typedef struct {
		const Material parent;
		Color reflectivity;
	} MaterialDiffuse;

	MaterialDiffuse makeMaterialDiffuse(const Color reflectivity);

	Photon materialDiffuseSampleBRDF(const Material *material, const Intersection intersection, const Photon incoming);

### MaterialDiffuse.c

	// The vtable for this specific class.
	const MaterialVTable materialDiffuseVTable = (MaterialVTable) {
		&materialDiffuseSampleBRDF,
		&materialDiffuseBRDF,
		&materialDiffuseIrradience
	};

	MaterialDiffuse makeMaterialDiffuse(const Color reflectivity) {

		// The vtable is automatically set correctly when you use the relevant constructor.		
		return (MaterialDiffuse) {makeMaterial(&materialDiffuseVTable), reflectivity};
	}

	Photon materialDiffuseSampleBRDF(const Material *superObject, const Intersection intersection, const Photon incoming) {
		
		MaterialDiffuse *material = (MaterialDiffuse *) superObject;
		
		return makePhoton(makeRay(vAdd(intersection.position, vsMul(intersection.normal, vEpsilon)), vSampleHemisphere(intersection.normal)), cMul(incoming.energy, material->reflectivity));
	}

So far I have only used purely virtual methods (using C++ terminology). It shouldn't be a problem to implement overridable methods if I need it later.

<hr>

It is also nice to have a simple way to create heap allocated objects. I solved it by using separate allocation and initialization functions, just like Objective C. Since all allocators are identical, except for the types, I wrote a macro for that.

	#define defineAllocator(type) \
	type * allocate##type(type data) { \
		type *object = malloc(sizeof(type)); \
		*object = data; \
		return object; \
	} \

If you are a C noob like me, you might winder what the double hash in `allocate##type` does. It is replaced by nothing, so that the argument `type` can be recognized right after "allocate" without having any whitespace. The effect is like string concatenation in the macro code. Calling the macro with the type `MaterialDiffuse` expands to:

	MaterialDiffuse * allocateMaterialDiffuse(MaterialDiffuse data) {
		MaterialDiffuse *object = malloc(sizeof(MaterialDiffuse));
		*object = data;
		return object;
	}

All this does is take an object on the stack, allocate space for it and store it on the heap and return the pointer to the new object. I built a special case of my container macro class to handle pointers, so using it is pretty convenient:

	// The container
	int capacity = 10;
	MaterialDiffusePointerContainer materials = makeMaterialDiffusePointerContainer(capacity);

	// Adding a material
	makeMaterialDiffusePointerContainerAddValue(
		&materials,
		(Material *) allocateMaterialDiffuse(makeMaterialDiffuse(makeColorWhite()))
	);

With this new object model in place I was able to easily rebuild the data structure of my scene graph, and add polymorphic materials. Yay for me! More about that in the next blog post.
