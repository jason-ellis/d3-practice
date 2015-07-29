# D3 notes

This is a brief overview of creating a chart with D3.

## Data Types

D3 can utilize the following data-types. Some forms of data might require additional conversion, so consult the docs.

* XMLHttpRequest
* text file
* JSON blob
* HTML document fragment
* XML document fragment
* comma-separated values (CSV) file
* tab-separated values (TSV) file

## Load D3

Preferably, host D3 locally. Optionally, it can be accessed with the URL below.

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/d3/3.5.6/d3.min.js" charset="utf-8"></script>
```

## Assign Variables for SVG

The variables that will be assigned during creation of the SVG element.
The dimensions can be static values, but the example below creates a responsive
chart.

```javascript
var margin = {top: 30, right: 30, bottom: 30, left: 30},
  width = parseInt(d3.select('body').style('width'), 10),
  width = width - margin.left - margin.right,
  height = 300 - margin.top - margin.bottom,
  tickFormat  = d3.format(".1%");
```

## Create SVG

Select the element within which to create an SVG and append the SVG element.
Assign the variables as attributes. To achieve responsiveness, we're actually
creating an SVG with the full dimensions and a `g` element for our viz inside
the SVG and margins. The `transform` attribute moves the `g` within the margins.

```javascript
var svg = d3.select("body")
  .append("svg")
  .attr("width", width + margin.left + margin.right)
  .attr("height", height + margin.top + margin.bottom)
  .append("g")
  .attr("transform", "translate(" + margin.left + "," + margin.top + ")");
```

## Create Scales
[Quantitative Scales](https://github.com/mbostock/d3/wiki/Quantitative-Scales)

Scales map input data values to new values suitable for display. D3 scales are *functions* whose parameters we define. After creation, we call the call the scale function, pass it a data value, and it returns a scaled output value.

`domain(array)` = min-max of input data

`range(array)` = min-max of output rendered

```javascript
var xScale = d3.scale.linear()
  .domain([0, d3.max(dataset, function(d) { return d[0] })])
  .range([0, width]);

var yScale = d3.scale.linear()
  .domain([0, d3.max(dataset, function(d) { return d[1] })])
  .range([height, 0]);

var rScale = d3.scale.linear()
  .domain([0, d3.max(dataset, function(d) { return d[1] })])
  .range([2,5]);
```

## Create Axes

Similar to scales, D3's axes are functions whose parameters we define. Axis
functions don't return a value, they instead generate the visual elements of
the axis when called.

After creation, an Axis must be passed as an argument to an SVG with the
`call()` method. This should be done after the data is plotted.

```javascript
var xAxis = d3.svg.axis()
  .scale(xScale)
  .orient("bottom")
  .ticks(5)
  .tickFormat(tickFormat);

var yAxis = d3.svg.axis()
  .scale(yScale)
  .orient("left")
  .ticks(5);
```

## Create Elements

The elements can be assigned to a variable for easier updating.

```javascript
var circles = svg.selectAll("circle")
  .data(dataset)
  .enter()
  .append("circle");

var labels = svg.selectAll("text")
  .data(dataset)
  .enter()
  .append("text");
```

## Assign Attributes

This can be combined with the element creation or done as a separate method
chain.

```javascript
circles.attr("cx", function(d) { return xScale(d[0]) })
  .attr("cy", function(d) { return yScale(d[1]) })
  .attr("r", function(d) { return rScale(d[1]) });

labels.text(function(d) { return d[0] + "," + d[1] })
  .attr("x", function(d) { return xScale(d[0]) })
  .attr("y", function(d) { return yScale(d[1]) })
  .attr("font-family", "sans-serif")
  .attr("font-size", "11px")
  .attr("fill", "blue");
```

## Add Axes

The axes are elements added to the SVG element and applied with the `call`
function.

```javascript
svg.append("g")
  .attr("class", "x axis")
  .attr("transform", "translate(0, " + height + ")")
  .call(xAxis);

svg.append("g")
  .attr("class", "y axis")
  .attr("transform", "translate(0,0)")
  .call(yAxis);
```

## Responsive D3 charts

To make the chart responsive, we need to write a resize function that will be called on window re

```javascript
d3.select(window).on('resize', resize);

function resize() {
  // update width
  width = parseInt(d3.select('body').style('width'), 10);
  width = width - margin.left - margin.right;

  // resize the chart
  xScale.range([0, width]);
  d3.select(svg.node().parentNode)
    .style('height', (yScale + margin.top + margin.bottom) + 'px')
    .style('width', (width + margin.left + margin.right) + 'px');

  circles.attr("cx", function(d) { return xScale(d[0]) })
    .attr("cy", function(d) { return yScale(d[1]) });

  labels.attr("x", function(d) { return xScale(d[0]) })
    .attr("y", function(d) { return yScale(d[1]) });

  // update axes
  svg.select('.x.axis').call(xAxis);
  svg.select('.y.axis').call(yAxis);
}
```

## SVG Notes

SVG is supported by all browsers except Internet Explorer 8 and older.

### `SVG` Element

SVG is XML based, so elements can be self closing.

All drawing must be done within an `SVG` element. Coordinates 0,0 start at top-left.

### Rectangle `rect`

* `x` - left
* `y` - top
* `width` - left to right
* `height` - top to bottom

### Circle `circle`

* `cx` - horizontal center
* `cy` - vertical center
* `r` - radius

### Ellipse `ellipse`

* `cx` - horizontal center
* `cy` - vertical center
* `rx` - horizontal radius
* `ry` - vertical radius

### Line `line`

* `x1` - start horizontal
* `y1` - start vertical
* `x2` - end horizontal
* `y2` - end vertical
* `stroke` - color of line

### Text `text`

Text goes between opening and closing `<text></text>` tags. Text follows CSS cascading style rules.

* `x` - left edge
* `y` - vertical position of the baseline

### Path `path`


### Styling

SVG’s default style is a black fill with no stroke. If you want anything else,
you’ll have to apply styles to your elements. Common SVG properties are:

* `fill` - A color value. Just as with CSS, colors can be specified as
  * named colors - `orange`
  * hex values - ``#3388aa` or ``#38a`
  * RGB values - `rgb(10, 150, 20)`
  * RGB with alpha transparency - `rgba(10, 150, 20, 0.5)`
* `stroke` - A color value.
* `stroke-width` - A numeric measurement (typically in pixels).
* `opacity` - A numeric value between 0.0 (completely transparent) and 1.0 (completely opaque).

With text, you can also use these properties, which work just like in CSS:

* `font-family`
* `font-size`

SVG styles can be applied inline or through CSS.

## Resources

* [Responsive Charts with D3](http://eyeseast.github.io/visible-data/2013/08/28/responsive-charts-with-d3/)
* [Scott Murray's D3 Tutorial](http://alignedleft.com/tutorials/d3)
* [Margin Convention](http://bl.ocks.org/mbostock/3019563)
