---
layout:     post
title:      Use Pandas to Create a D3 Force Directed Diagram
date:       2016-02-01 19:06:30
time:	5
summary:    Network professionals often try to visualize their network connections. This tutorial will show you how to convert your network traffic into a beautiful interactive illustration.
categories: d3 python pandas
products:
 - top-level: Python 3
   arbitrary: Required Modules
   nested-products:
    - nested: Pandas
      sub-arbitrary: pip install pandas
    - nested: IP Address Module
      sub-arbitrary: pre-installed with Python 3.x
 - top-level: A text editor
   arbitrary: Your choice
   nested-products:
    - nested: My Favorites
      sub-arbitrary: Sublime Text 3, iPython Notebook
 - top-level: Optional
   arbitrary: You can get iPython Notebook and Pandas together by installing Anaconda 3
---



*Our Goal*

Create an [interactive graph](http://bl.ocks.org/mbostock/4062045) force directed graph to illustrate network traffic.


<style>

.node {
  stroke: #fff;
  stroke-width: 1.5px;
}

.link {
  stroke: #999;
  stroke-opacity: .6;
}

</style>

<script src="//d3js.org/d3.v3.min.js"></script>
<script>


var width = 960,
    height = 500;

var color = d3.scale.category20();

var force = d3.layout.force()
    .charge(-120)
    .linkDistance(30)
    .size([width, height]);

var svg = d3.select("body").append("svg")
    .attr("width", width)
    .attr("height", height);


d3.json("/static/miserables.json", function(error, graph) {
  if (error) throw error;

  force
      .nodes(graph.nodes)
      .links(graph.links)
      .start();

  var link = svg.selectAll(".link")
      .data(graph.links)
    .enter().append("line")
      .attr("class", "link")
      .style("stroke-width", function(d) { return Math.sqrt(d.value); });

  var node = svg.selectAll(".node")
      .data(graph.nodes)
    .enter().append("circle")
      .attr("class", "node")
      .attr("r", 5)
      .style("fill", function(d) { return color(d.group); })
      .call(force.drag);

  node.append("title")
      .text(function(d) { return d.name; });

  force.on("tick", function() {
    link.attr("x1", function(d) { return d.source.x; })
        .attr("y1", function(d) { return d.source.y; })
        .attr("x2", function(d) { return d.target.x; })
        .attr("y2", function(d) { return d.target.y; });

    node.attr("cx", function(d) { return d.x; })
        .attr("cy", function(d) { return d.y; });
  });
});

</script>


The [dataset](http://pen-testing.sans.org/holiday-challenge/2013) we're going to use is from a [SANS Holiday Challenge in 2013](http://pen-testing.sans.org/holiday-challenge/2013) which is [available here](http://pen-testing.sans.org/sansholidayhack2013.pcap)


Getting started...
---

---
<head>
  <title>{{ page.title }}</title>
</head>

<body>

<h4>Required:</h4>
<ul>{% for product in page.products %}
  <li>{{ product.top-level }}: {{ product.arbitrary }}{% if product.nested-products %}
    <ul>
    {% for nestedproduct in product.nested-products %}  <li>{{ nestedproduct.nested }}: {{ nestedproduct.sub-arbitrary }}</li>
    {% endfor %}</ul>
  {% endif %}</li>{% endfor %}
</ul>
</body>

Step 1: Extract Data
---
In this example, we're going to export the metadata from our PCAP using wireshark.

Set your filter<br>
<small>_Type ip into the filter for IPv4 addresses_</small>

![Set Filter]({{ site.base }}/images/set_filter.png)
<br>
Mark the packets for export.
<br>
<small>_Edit > Mark All Displayed_</small>

![Mark Packets for Export]({{ site.base }}/images/pre_marked.png)

Save/Export packets as CSV format.
<br>
<small>_File > Export Packet Dissections > Save as CSV_</small>
![Export Marked Packets]({{ site.base }}/images/marked_export.png)

Name your file something you'll remember. I named mine <b>packet_metadata.csv</b>


Step 2: Transform Data
---

Step 3: Load Data
---


