---
category : Programming
tags : [client-side templating, CSS transitions, javascript, CSS, Handlebars, Mustache]
title: Client-Side Templating Breaks CSS Transitions 
---


<style type='text/css'>

	iframe{
		display: block;
		width: 616px;
		height: 250px;
	}

</style>

If you are anything like me, you just love those smooth CSS transitions. Really, they add a lot of polish to your project, and in some cases can help with usability when they prevent jerky state changes.

Client-side templating is all the rage these days, and with good reason. Keeping your front-end code in the browser, and letting the server handle the backend separates things neatly. At least that is the theory, reality is always a bit hairier.

...Which brings me to the issue of the title. Rendering a template with a templating engine such as Handlebars or Mustache completely re-creates the DOM elements and replaces the ones in the page. The old element is discarded, CSS transitions and all. Thrown away like yesterdays Flash. (See what I did there?)

The result is that any transition that you would expect to be triggered, just doesn't animate.

Am I Affected?
--------------

If you use CSS transitions (or would like to), and you indicate state changes in your app with CSS classes, you are affected. Otherwise you are probably OK. I have not tested it in every templating engine/browser combination, but *as fas as I know*, this is a universal issue. Actually, it is the only sensible behavior.

An example: You have a webshop backend with a list of orders. Each order can be in any of the states

* pending
* processing
* delivered


This state is indicated by the background color of the row, and when it changes you want it to do a quick fade, rather than just snap. To get this effect, you have a template something like this:
{% raw %}

	<script id="order-template" type="text/x-handlebars-template">
		<li class="{{status}}">...</li>
	</script>

{% endraw %}
The class of the `li` element changes depending on the order status, and the CSS is set up to have different colors depending on the class and to do a transition.

Here is some code trying to do that with Handlebars. Watch the order element snap to yellow after a second, without any transition: (Click "Result" to see it in action.)

<iframe src='http://jsfiddle.net/3e3QR/4/embedded'>jekyll breaks without this</iframe>

What can I do?
----------------

If CSS transitions are important to you, you might not want to use a templating engine like Handlebars or Mustache to do the state changes of your elements. Instead, set the class attribute manually.

The example above can be changed to manipulate the CSS class of the element instead of regenerating it. (Click "Result" to see it in action.)

<iframe src='http://jsfiddle.net/JqJUj/2/embedded/'>jekyll breaks without this</iframe>

Look at that! The transition works, and everyone is happy, except the developers who need to transition (pun intended) from fully template driven code to more manual meddling.

One of my Backbone views looks something like this:

	// The order view class, showing only the relevant code.
	var OrderView = Backbone.View.extend({

		initialize: function() {

			this.model.on("change", this.updateDOM, this);
		},

		updateDOM: function(order) {

			this.$el.attr("class", order.get("status"));
		},
	});

The `updateDOM` function sets the view up completely, so it's used to initialize the view too:

	// The order collection view, still showing only the relevant code.
	var OrdersView = Backbone.View.extend({

		initialize: function() {

			this.collection.on("add", this.prependOrder, this);
		},

		prependOrder: function(model) {

			var orderElement = $(this.template);

			model.view = new OrderView({
				el: orderElement,
				collection: this.collection,
				model: model
			});
			
			// Set up the view with the correct data.
			model.view.updateDOM(model);

			// Show the new element.
			this.$el.prepend(orderElement);
		},
	}


You can do it pretty neat and structured... Or better yet - build The Next Big Templating Engine to handle this use case.

Happy coding!
