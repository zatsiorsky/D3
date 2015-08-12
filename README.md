# D3 Walkthrough
## Introduction
The process of rendering a graph in D3 goes through the same basic steps (details of each step are explained later):

1.	**Initialization** - Initialize constants and append static HTML/SVG elements.
2. **Build** - Modify the data to be in a form suitable for the particular visualization you are making.
3. **Update** - Update the visualization. This step handles drawing dynamic elements for the first time, updating existing elements, and removing elements.



## Initialization

D3 visualizations are created using mainly SVG elements. Generally, a visualization has a parent `<svg>` element, with a container `<g>` element that houses the visual elements. The position and dimensions of the container `<g>` element are determined by the height, width, and margin constants set at the top of the file. This is illustrated below:

![](https://github.com/zatsiorsky/D3/blob/master/img-1.png)

To reproduce the above in code, the `<svg>` and `<g>` elements could be set up as such:

		this.svg = d3.select(this.parentElement).append("svg")
            .attr("width", this.width)
            .attr("height", this.height);

        this.g = this.svg.append("g")
			.attr("transform", "(" + this.margin.left + "," + this.margin.top + ")");

### Specifics in *GraphController.js*

The following are completed in the initialization function in *GraphController.js*:

- Set some options for the graph, such as graph type, graph orientation, and tooltip hovering.
- Appending the parent `<svg>` element and container `<g>` element to the parent `<div>` passed as a parameter to the `this.initialize` function.
- Append `<g>` elements for the x and y-axes.
- Add a title to the top of the graph if the graph settings specify. If a title should be appended, the top margin is adjusted to account for it.
- Set up binding for window resize and graph refreshing.
- Fetch data for the visualization.

## Build

After the data is acquired, it is sent to be built into the desired format for visualization. This format will vary for each visualization. To demonstrate, we will walk through the data builds for the visualizations in *GraphController.js*.

### Original format

The data fetched by the controller is an array of objects formatted as the following:

	[
		{
			Column1: value,
			Column2: value,
			...
		},
		{
			Column1: value,
			Column2: value,
			...
		},
		...
	]

### Line and Bar Charts

For the line and bar charts, the data can be in the same format. The data is built to look like the following:

	[
		{
			name: "Age"
			values: [
			{
				name: "Age",
				x: 2,
				y: 21
			},
			{
				name: "Age",
				x: 3,
				y: 25
			},
			...
			]
		},
		...
	]

Each object in the array corresponds to a data set that should be displayed on the graph. For line charts, multiple data sets can be displayed at once. So, for example, the array above could have a second object with the property `name: "Hours Worked"`. There would then be a second line displayed on the chart. **However, for bar charts, currently only one data set can be displayed at a a time.**

### Sunburst Visualization

[This is](http://bl.ocks.org/mbostock/4063423) a link to Mike Bostock's example of a sunburst visualization. The data in *GraphController.js* is built to match the example.

	{
	 	"name": "root",
	 	"children": [
	  	{
	   		"name": "Samsung",
	   		"children": [
	    	{
		     	"name": "Video Transcriptions",
				"size": 1000
	    	},
			{
				"name": "Street Events",
				"size": 250
			},
			...
		},
		...
	    ]
	}

### Stacked Bar Chart

[This is](http://bl.ocks.org/mbostock/1134768) a link to Mike Bostock's example of a stacked bar chart. The data in *GraphController.js* is built to match the example. Each element of the array (in this example) corresponds to the steps associated with a particular customer. 

	[
		[
			{
				name: "Samsung",
				x: "Audio Transcription and Analysis - French",
				y: 57610,
				y0: 0
			},
			{
				name: "Samsung",
				x: "Company Profile Classifications",
				y: 0,
				y0: 0
			},
			...
		],
		[
			{
				name: "Thomson Reuters",
				x: "Audio Transcription and Analysis - French",
				y: 0,
				y0: 57610
			},
			{
				name: "Thomson Reuters",
				x: "Company Profile Classifications",
				y: 0,
				y0: 0
			},
			...
		],
		...
	]

After the data is built, it is passed to the update stage of the visualization.

## Update

Now comes the fun part of using D3. The visualization update flow of D3 follows a general pattern of *update, enter, exit*. Let's do a simple example:

### Example

Let's say we are given the data set `var data = [1, 2, 3]` and want to create some circles with it.

The D3 way to make a selection of all circles in a parent `<svg>` is:

	d3.select("svg").selectAll("circle");

We are starting with a blank canvas, so we will get an empty array in return. Instead, let's bind some data to the selection:

	var circles = d3.select("svg").selectAll("circle").data(data, function(d) { return d; });

Let's quickly discuss the `.data(data, function(d, i) { return d; })` segment. We are binding `data` to the circle elements, and the second parameter assigns a key for data-binding.  By default, the key is the index of the datum. In our case, we want each circle to be assigned to the actual value of the datum. The parameter `d` corresponds to the datum, while `i` corresponds to the index of the datum.

Unfortunately, we will still get an empty array in return, since D3 is trying to match existing circles with the data we bound to the selection. Thankfully, we can use the `enter()` selection to create new circles.

`circles.enter()` will return an array with length 3, since we have 3 data elements that have not yet been bound to circle elements.

Let's append those circles to the `<svg>`:

	var circlesEnter = circles.enter()
		.append("circle");

Now we've appended a `<circle>` element for each new data point. But, we haven't added any attributes to the new circles. Let's do that:

	circlesEnter
		.attr("cx", 5)
		.attr("cy", function(d, i) {
			return 10 * i;
		})
		.attr("r", function(d) {
			return d;
		});

The `cx` attribute of each circle determines the x-position. In each case, the circles will have an x-position of 5.

The `cy` attribute of each circle is determined dynamically. When `function(d, i)` is passed to `attr()`, you can access the datum that is bound to a particular element. `d` is the datum bound to the element, while `i` is the index of the datum in the data array. In this case, the three circles will have y-positions of 0, 10, and 20 respectively.

The `r` attribute determines the radius of the circle. We are only interested in the datum (not the index), which is why we pass in `function(d)` rather than `function(d, i)`.

Great! Now we've appended the circles to the `<svg>` and assigned them attributes. Now we will go through the update process again:

	var circles = d3.select("svg").selectAll("circle")
		.data(data, function (d) { return d; });

This time around, `circles` is an array of length 3, since the circles corresponding to the data aleady exist in the DOM. We can update existing elements right off of the `circles` selection, 

	circles
		.attr("r", 5);

We simply updated the radii of all of the circles to 5.

Let's see what happens when the data changes.

`data = [1, 3, 4]`

We've removed `2` from the array, and added `4` to the array. Naturally, we expect that one additional circle will be appended to the DOM corresponding to `4`, and that the circle corresponding to `2` will be removed.

Let's make our selection again.

	var circles = d3.select("svg").selectAll("circle")
			.data(data, function (d) { return d; });

Now, `circles` has length 2, since only `1` and `3` correspond to circles currently in the DOM.

Let's get rid of the extra circle using the `exit()` selection:

	circles.exit().remove();

That got rid of the circle corresponding to the datum `2`.

Now let's add the new circle:

	circles.enter().append("circle")
		.attr("cx", 5)
		.attr("cy", function(d, i) {
			return 10 * i;
		})
		.attr("r", function(d) {
			return d;
		});

This added the circle corresponding to the new datum, `4`. Now `circles` contains all of the circles shown in the DOM (including those just added).


**Review**

1. *update* - update attributes of existing elements. The selection is made using something like: `var circles = d3.select("svg").selectAll("circle").data(data);`
2. *enter* - new elements. The selection is made using something like `circles.enter()`
3. *exit* - elements to be removed. The selection is made using `circles.exit()`

### Transitions

In the example above, attribute changes occur instantaneously. It is simple to animate the attribute changes in D3 using *transitions*. Let's look at a simple example by modifying the code block above.

	circles.enter().append("circle")
		.attr("cx", 5)
		.attr("cy", function(d, i) {
			return 10 * i;
		})
		.attr("r", 0)
	.transition()
		.attr("r", function(d) {
			return d;
		});

This code sets the initial radius of the circles to 0, and then transitions the radius to the final value over time. By default, the duration of a transition is 250ms. This can be changed using `.transition().duration(n)`, where `n` is the number of milliseconds you would like the transition to take. There is no delay on the transition by default, but this can also be specified using `.transition().delay(m)`, where `m` is the number of milliseconds you would like to delay. For example, to transition the color of an circle from green to red over two seconds after a delay of 500ms, you could do the following:

	circles
		.attr("fill", "green")
	.transition().duration(2000).delay(500)
		.attr("fill", "red);

#### Chaining Transitions

It is possible to chain multiple transitions on the same element. For example, we could change the color of the circles from green to red, and then back to green with the following code:

	circles
		.attr("fill", "green")
	.transition()
		.attr("fill", "red")
	.transition()
		.attr("fill", "green")

Some important notes about transitions:

- **New transitions on elements overwrite old ones.** If you need to call two simultaneous transitions on the same element without overwriting, you need to specify a transition name. This can be done simply with `.transition("transitionName")`. In *GraphController.js*, one instance of this occurs in the bar chart visualization. When the user hovers over one of the bars, the opacity of the fill attribute changes. When the data is sorted, the bars transition to their new positions. These are two distinct transitions that can occur simultaneously on the same element. Without specifying transition names, hovering over a bar during the position transition would cause the bar to freeze at its current location, since the position transition was stopped.

- **Specify attribute defaults.** Let's say that you want to change the fill opacity of a circle when you hover over it, and then reverse the change when you mouse out. This can be done with the following code (when circles are appended to the DOM).

		circles.enter().append("circle")
			.attr("cx", 20)
			.attr("cy", 20)
			.attr("r", 5)
			.on("mouseover", function(d) {
				d3.select(this)
					.transition()
					.attr("fill-opacity", 0.5);
			})
			.on("mouseout", function(d) {
				d3.select(this)
					.transition()
					.attr("fill-opacity", 1);
			});
If you ran this visualization, you would notice that there is strange behavior the first time you mouse over a circle. This is because D3 does not have a starting point for the fill-opacity attribute. To fix this, simply specify a starting attribute value after appending the `<circle>` elements.

		circles.enter().append("circle")
			.attr("fill-opacity", 1) // specify initial value
			...

- **Use the "end" event to chain transitions with delays.** In the transition chaining example above, there are no delays associated with the transitions. If any of the chained transitions require a delay, it is recommended to use the "end" event to trigger the next transition. The issue is not well-documented, but switching out of the browser window during chained transitions with delays can cause the chain to break unless you use the "end" event. Here's an example where the circles transition from red to green, delay 500ms, transition from green to red, delay 1s, and transition from red to blue:

		circles.enter().append("circle")
			... // set position and such
			.attr("fill", "red")
		.transition()
			.attr("fill", "green")
			.each("end", secondTransition);
		
		function secondTransition () 
		{
			d3.select(this)
				.transition()
				.delay(500)
				.attr("fill", "red")
				.each("end", thirdTransition);
		}

		function thirdTransition () 
		{
			d3.select(this)
				.transition()
				.delay(1000)
				.attr("fill", "blue");
		}
More info on `transition.end()' [here.](https://github.com/mbostock/d3/wiki/Transitions#control)
- **You can control the "feel" of transitions with `.ease()`.** By default, transitions have an ease of "cubic-in-out" (slow-fast-slow). `.ease("linear")` is sometimes used in *GraphController.js* for a constant transition. Check [this](https://github.com/mbostock/d3/wiki/Transitions#easing) out for more info on easing.

Check out [this](http://plnkr.co/edit/EjvvfC8jZNlZtS9u2Qic?p=preview) cool D3 transitions firework show which demonstrates some of these things.

## Other *GraphController.js*-specific things

### Legend

*GraphController.js* uses a small package called *d3.legend* to render a legend. The following code draws the legend:

	this.legend = this.g.append("g")
                .attr("class", "legend")
                .style("font-size", "10px")
                .call(d3.legend);

    this.legendWidth = this.legend.node().getBBox().width;

    this.legend
        .attr("translate(" + (that.width - that.legendWidth) + ",0)")

The `this.legendWidth` code gets the width of the legend `<g>` element. With it, we can calculate the precise position so that the legend fits nicely into the top-right of the view.

Besides drawing the legend, it is important to specify which data should be used for it. This is done in the *Update* phase. Let's assume we want to add a legend for our `<circle>` elements from before. Then it would be as simple as:

	circles.enter().append("circle")
	... // set initial attributes
    .attr("data-legend", function(d) {
		return d;
	})
	...

The above code adds an element to the legend for each datum value. For datum objects that have multiple properties, you can pass a particular field (like `d.name`) to the data-legend attribute. Currently, in *GraphController.js*, the legend only renders on the first load of the graph. To support legend updating, some additional work would need to be done.

### Tooltip

*GraphController.js* uses a small package called *d3.tip* to enable data hovering. The following code initializes the tooltip:

	this.tip = d3.tip()
        .attr('class', 'd3-tip')
		// move 10px left of calculated position
        .offset([-10, 0]) 
		// HTML you want to display in tooltip
		// d is the datum bound to the element you are hovering over
        .html(function (d) 
        {
			return "<h1>" + d + "</h1>";
		});
	// call tip on the SVG element you want it to be active in
	this.g.call(this.tip);

Activating the tooltip also requires adding mouseover and mouseout events to the SVG elements of interest. The following will do:

	circles.enter().append("circle")
		... // set initial attributes
	    .on("mouseover", this.tip.show)
		.on("mouseout", this.tip.hide)
		...


    














	




