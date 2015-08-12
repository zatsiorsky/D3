# D3 Walkthrough
## Introduction
The process of rendering a graph in D3 goes through the same basic steps (details of each step are explained later):
	1. **Initialization** - Initialize constants and append static HTML/SVG elements.
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





