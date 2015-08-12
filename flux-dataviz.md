# FLUX <!-- .element: class="photo-overlay huge white-shadow" -->
## for data visualization <!-- .element: class="photo-overlay white-shadow" -->
### (a preliminary manifesto) <!-- .element: class="photo-overlay white-shadow" -->
<!-- .slide: data-background="images/aurora-borealis.jpg" -->

Note:
This isn't the slide you're looking for. Just the title ;)


# Jana Beck
## Software Engineer, Tidepool
### <small>GitHub:</small> [@jebeck](https://github.com/jebeck) <small>(also Freenode IRC, Gitter, Slack, etc.)</small>
### <small>Twitter:</small> [@iPancreas](https://twitter.com/iPancreas)
<!-- .slide: data-background="images/lancets.jpg" -->

Note:
I'm an engineer at Tidepool. We're a not-for-profit, open-source effort to reduce the burden of type 1 diabetes through data technology. I work a lot on data, principally data ingestion and (our topic today) data visualization.

I'm jebeck on GitHub and in most other places. And I'm iPancreas on Twitter. Feel free to reach out - I really hope this talk will be a conversation starter.


# slides
## [janabeck.com/flux-for-dataviz](http://janabeck.com/flux-for-dataviz)
<!-- .slide: data-background="images/lancets.jpg" -->

Note:
While I'm still giving you a bit of a intro, here's the link to where these slides are hosted on the web. At the last conference I was at, everyone put their slides link on the last slide, and then it was a scramble to copy the links down, so write it down now if you want!

There are also a bunch of other links in the slides, so if you think you might want access to those links later, you might want to write down the URL now. :)


# WARNING <!-- .element: class="photo-overlay huge white-shadow" -->
<!-- .slide: data-background="images/construction.jpg" -->

Note:
Before we dive in, I want to pause and talk about goals. The goal of this talk for me - and I hope this is useful for you, too - is to talk through some of the challenges and problems I've been thinking through as I think about what the next version of our data visualization library at Tidepool might look like. What that means is that everything I'm about to talk to you about is quite preliminary. We're going to be talking about thoughts and ideas and experiments that are still very much under construction.

On the flip side, sometimes I get frustrated seeing so many simple theoretical examples when we're talking about new patterns like Flux. If you're like me and also get frustrated by endless to-do list examples, I hope you enjoy what follows. I'll use some simple, theoretical examples, but we'll also talk about big, sprawling, complex things - to the best of my ability to cover such things in a twenty-five minute talk, at least.



# the problem <!-- .element: class="photo-overlay huge white-shadow" -->
<!-- .slide: data-background="images/disaster.gif" -->

Note:
You could call the central problem I'm about to address by many names. "Hidden state" would be one way to put it. Tools that don't scale up to fit into a complex project in a developer-friendly way would be another way to put it. Let's keep going and define it a little more closely as we go.


<img data-src="images/D3.svg" title="D3.js logo" alt="D3.js logo" style="border: none; box-shadow: none;"/>

Note:
The title of this talk is "Flux for Data Visualization," and I'm definitely going to be talking about that. Specifically I will be talking about D3 - the JavaScript library that anchors the most commonly used toolset for interactive data visualization on the web. But a lot of what I'm about to talk about applies equally to any client-side application with a significantly complex UI.


# what's complex? <!-- .element: class="photo-overlay-bright huge white-shadow" -->
<!-- .slide: data-background="images/complexity.jpg" -->

Note:
What do I mean by significantly complex? Well, one of the things that leapt out at me when I was first learning about React and Flux by watching some of the videos from the Facebook team was the story of the zombie spurious unread message counter bug, always rising again from the dead, and the key point that what made debugging the situation so impossible in the pre-Flux days was the fact that many different places in the UI both displayed and had the power to manipulate the unread message count. So I've really latched onto this is my threshold for when a project or application is getting big enough to think about employing the Flux pattern: if the same information is being displayed and potentially changed in mutltiple places in the UI, *that's* my definition of complex.


# [<small>let's make</small> a bar chart](http://bost.ocks.org/mike/bar/)

Note:
So as a tiny intro to the ideas of D3 relevant to the the topic at hand I'm going to cherry-pick a bit from one of the fabulous intro articles by Mike Bostock, D3's primary author. The article walks through how to create a simple data visualization - a bar chart.


input:
```JavaScript
var data = [4, 8, 15, 16, 23, 42];
```

output:
<div class="chart">
  <div style="width: 80px;">4</div>
  <div style="width: 160px;">8</div>
  <div style="width: 300px;">15</div>
  <div style="width: 320px;">16</div>
  <div style="width: 460px;">23</div>
  <div style="width: 840px;">42</div>
</div>

Note:
What we want is to turn a small array of data into a bar chart for easy visual comparison. How could we do that?


## <small>(the straw man)</small> hard code it?

```HTML
<div id="chart">
  <div style="width: 40px;">4</div>
  <div style="width: 80px;">8</div>
  <div style="width: 150px;">15</div>
  <div style="width: 160px;">16</div>
  <div style="width: 230px;">23</div>
  <div style="width: 420px;">42</div>
</div>
```

Note:
The straw man solution is to hard code it in HTML, hand-calculating the width that each div should have in order to represent each item in the data array.


## D3 to the rescue!

```JavaScript
d3.select("#chart")
	.selectAll("div")
	.data(data)
	.enter()
	.append("div")
	.style("width", function(d) { return d * 10 + "px"; })
	.text(function(d) { return d; });
```

Note:
We can do it better with D3. Instead of manually creating divs representing each item in the data array, we can use D3 to bind the data to child divs of the main "chart" div we've set up. D3 will create the child divs if they don't exist yet, and we can assign the visual attributes that we want the divs to have with D3's `style` and `text` methods. (The former sets the width of each bar programmatically according to the value of each datum, and the latter adds the labels showing the value of each datum within each bar.)


```JavaScript
.append("div")
.style("width", function(d) { return d * 10 + "px"; })
```
vs.
```JavaScript
var chart = document.getElementById('chart');
for (var i = 0; i < data.length; ++i) {
	var thisDiv = document.createElement('div');
	thisDiv.setAttribute('width', data[i] * 10 + 'px');
	chart.appendChild(thisDiv);
}
```

Note:
Let's focus in on this bit from the previous slide. How would we accomplish the same thing as what D3's `append` and `style` methods do in "pure" JavaScript? Not so hard, perhaps, it just requires a simple loop.


# declarative

vs.

# procedural

Note:
What's the difference between the two approaches? The D3 version is quite a bit more declarative - we don't specify the operations in the DOM, we specify the result we want - a div added for each datum with a width 10 times the value of the datum, in pixels. The alternative - using a for loop - is by comparison quite procedural.


<!-- .slide: data-background="images/yawn.gif" -->

Note:
OK, but this is boring, this is too simple. What are we talking about again?


# + interactivity <!-- .element: class="photo-overlay-dark big white-shadow" -->
<!-- .slide: data-background="images/sprinkles.gif" -->

Note:
We're trying to get into a conversation about using Flux to manage the state of large, complex data visualizations. What makes such visualizations complex? The addition of interactivity is definitely a complexity multiplier.


## D3 tooltips
```JavaScript
d3.select("#chart")
	.selectAll("div")
	.data(data)
	.enter()
	.append("div")
	.style("width", function(d) { return d * 10 + "px"; })
	.on('mouseover', function(d) {
		d3.select(this).append("div.tooltip")
		... // position and style the tooltip div
	})
	.on('mouseout', function(d) {
		// remove the tooltip on mouseout
		d3.select(this).select('div.tooltip').remove();
	});
```

Note:
So bear with me on the tiny bar chart example for a bit longer. What if we want the value of the datum corresponding to each bar to appear in a tooltip instead of cluttering up the bar itself with a label? Here's the most common pattern in D3 for adding tooltips to elements of a visualization.

Now, here I have to pause and give a big hat tip to Nico Hery, a former colleague of mine, who was the first to point out this issue to me. What's the issue here? The issue is that this direct binding between events and DOM manipulation through the listener function is veering back into procedural territory. The state of the visualization with respect to the user's interactions is also completely hidden, encapsulated tightly within the D3 code and not exposed in a developer-friendly way (for testing, for design iteration, etc.)

Nico has a great blog post where he discusses this very problem and suggests the alternative of using D3 event listeners not to *draw* the tooltips, but merely to add the datapoint currently being moused over to an array of focus objects, all of which will then be provided as the data to another completely separate D3 module that renders the tooltips. The ideas I'm presenting today are a further logical extension of that idea - I'm advocating managing all state in a complex data visualization with the Flux architecture of one-way data flow. That means making all aspects of the state of a visualization explicit - nothing buried deep inside the D3 modules used for rendering.



## (interlude) <!-- .element: class="photo-overlay white-shadow" -->
# sandwiches <!-- .element: class="photo-overlay big white-shadow" -->
<!-- .slide: data-background="images/pb&j.jpg" -->

Note:
Now, let's take a little break and talk about peanut butter and jelly sandwiches. Just bear with me. Maybe this memory of mine will even be familiar to some of you. In my 9th grade biology class, we were given an exercise: write out detailed instructions for making a peanut butter and jelly sandwich. The teacher collected all of our exercises and then stood at the front of the class at a lab table with a loaf of bread, a knife, a jar of peanut butter, and a jar of jelly. Needless to say, she didn't succeed in making any peanut and jelly sandwiches. Our instructions were *terrible*.

What was the lesson here? The importance of precision and detail in *procedure*.


# the Elvis <!-- .element: class="photo-overlay big white-shadow" -->
<!-- .slide: data-background="images/elvis.jpg" -->

Note:
Now this? This is a variation on the classic PB & J - an Elvis sandwich. Peanut butter, grilled bananas, and bacon, toasted. It's *heavenly*.


## how to fry <!-- .element: class="photo-overlay white-shadow" -->
# bacon <!-- .element: class="photo-overlay big white-shadow" -->
<!-- .slide: data-background="images/bacon.jpg" -->

Note:
As a diner in a restaurant, if you want an Elvis instead of a PB & J, do you have to include instructions for how to fry the bacon in your order? Of course not. We have better interfaces for that.


## none pizza with left beef

<img width="850" src="images/none-pizza.jpg" alt="'None pizza with left beef'">

Note:
We have interfaces *so good*, in fact, that you can order a "none pizza with left beef" and get your order filled successfully, without having to explain any procedure at all.

My argument in this talk is that both from the developer's and the user's side of things, we want to strive to make the experience of a complex interactive data visualization more like the "none pizza with left beef" experience - exactly what you want, defined through a simple but powerful interface, with the implementation details totally independent (in other parts of the code, to take it back to programming).



# a solution? <!-- .element: class="photo-overlay-dark big white-shadow" -->
<!-- .slide: data-background="images/dominoes.gif" -->


## let's talk about <!-- .element: class="photo-overlay-dark white-shadow" -->
# weather (data) <!-- .element: class="photo-overlay-dark big white-shadow" -->
<!-- .slide: data-background="images/blizzard.jpg" -->


## hourly details
![hourly details](images/weather.png)


## 10-day forecast
![day-by-day forecast](images/weather.png)


## datatypes

- (cloud) condition
- temperature:
	+ low
	+ high
	+ "feels like"
- precipitation:
	+ type
	+ likelihood
- humidity
- wind:
	+ speed
	+ direction


## view determinators

- location
- time (past, now, future)
- user preference


# what you want <!-- .element: class="photo-overlay big white-shadow" -->
<!-- .slide: data-background="#2e5a7e" -->


# what you have <!-- .element: class="photo-overlay-dark big white-shadow" -->
<!-- .slide: data-background="#a9d8fe" -->



# a more complex example <!-- .element: class="photo-overlay big white-shadow" -->
<!-- .slide: data-background="images/starry-night-dominoes.gif" -->


# tideline <!-- .element: class="photo-overlay-dark huge white-shadow" -->
<!-- .slide: data-background="images/tideline.png" -->



# links!

["Integrating D3.js visualizations in a React app"](http://nicolashery.com/integrating-d3js-visualizations-in-a-react-app/) - Nicolas Hery

[medusa on GitHub](https://github.com/jebeck/medusa)