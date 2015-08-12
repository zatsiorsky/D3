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
			.attr("width", this.width - this.margin.left - this.margin.right)
			.attr("height", this.height - this.margin.top - this.margin.bottom)
			.attr("transform", "(" + this.margin.right + "," + this.margin.top + ")");

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
		.attr("x", 5)
		.attr("y", function(d, i) {
			return 10 * i;
		})
		.attr("r", function(d) {
			return d;
		});

The `x` attribute of each circle determines the x-position. In each case, the circles will have an x-position of 5.

The `y` attribute of each circle is determined dynamically. When `function(d, i)` is passed to `attr()`, you can access the datum that is bound to a particular element. `d` is the datum bound to the element, while `i` is the index of the datum in the data array. In this case, the three circles will have y-positions of 0, 10, and 20 respectively.

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
		.attr("x", 5)
		.attr("y", function(d, i) {
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










	




