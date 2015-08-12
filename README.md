# D3 Walkthrough
## Introduction
The process of rendering a graph in D3 goes through the same basic steps (details of each step are explained later):
1. **Initialization** - Initialize constants and append static HTML/SVG elements.
2. **Build** - Modify the data to be in a form suitable for the particular visualization you are making.
3. **Update** - Update the visualization. This step handles drawing dynamic elements for the first time, updating existing elements, and removing elements.



## Initialization

D3 visualizations are created using mainly SVG elements. Generally, a visualization has a parent `<svg>` element, with a container `<g>` element that houses the visual elements. The position and dimensions of the container `<g>` element are determined by the height, width, and margin constants set at the top of the file.