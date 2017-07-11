---
layout:     post
title:      Use Python & Pandas to Create a D3 Force Directed Network Diagram 
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
author: austin_taylor
author_github: austin-taylor
---



Our Goal
---
Create an [interactive force directed graph](http://bl.ocks.org/mbostock/4062045) to illustrate network traffic.

<iframe src="/static/examples/force_directed/index.html" width="960px" height="500px" frameborder="0" scrolling="no"></iframe>

_You may need to edit the width and height depending on the size of your network_

To get started save the following code to a file named index.html to your desktop or a path you'll remember.

## index.html

{% highlight html %}
<!-- python -m SimpleHTTPServer 8080 //-->
<!-- http://bl.ocks.org/mbostock/4062045 //-->

<!DOCTYPE html>
<head>
<meta charset="utf-8">
<title>Pandas D3 Force Directed Example - www.austintaylor.io</title>

<!-- JavaScript Libraries //-->
<script src="http://d3js.org/d3.v3.min.js"></script>

<!-- CSS Style //-->
<link href="http://fonts.googleapis.com/css?family=Source+Sans+Pro:300,900|Source+Code+Pro:300" rel="stylesheet" type="text/css">
<style>
body {
    font-family: 'Source Sans Pro', sans-serif;
    font-weight: 300;
}

b {
    font-weight: 900;
}

.outline {
    fill: none;
    stroke: #888888;
    stroke-width: 1px;
}

#tooltip {
    font-size: 10pt;
    font-weight: 900;

    fill: #000000;
    stroke: #ffffff;
    stroke-width: 0.25px;
}

.node {
    stroke: #ffffff;
    stroke-weight: 1px;
}

.link {
    fill: none;
    stroke: #888888;
    stroke-weight: 1px;
    stroke-opacity: 0.5;
}

.highlight {
    stroke: red;
    stroke-weight: 4px;
    stroke-opacity: 1.0;
}
</style>

<script>
var width  = 960;
var height = 500;
var margin = 20;
var pad = margin / 2;
var color = d3.scale.category20();
// Generates a tooltip for a SVG circle element based on its ID
function addTooltip(circle) {
    var x = parseFloat(circle.attr("cx"));
    var y = parseFloat(circle.attr("cy"));
    var r = parseFloat(circle.attr("r"));
    var text = circle.attr("id");
    var tooltip = d3.select("#plot")
        .append("text")
        .text(text)
        .attr("x", x)
        .attr("y", y)
        .attr("dy", -r * 2)
        .attr("id", "tooltip");
    var offset = tooltip.node().getBBox().width / 2;
    if ((x - offset) < 0) {
        tooltip.attr("text-anchor", "start");
        tooltip.attr("dx", -r);
    }
    else if ((x + offset) > (width - margin)) {
        tooltip.attr("text-anchor", "end");
        tooltip.attr("dx", r);
    }
    else {
        tooltip.attr("text-anchor", "middle");
        tooltip.attr("dx", 0);
    }
}
var vis = d3.select("#chart")
  .append("svg:svg")
    .attr("width", w)
    .attr("height", h)
    .attr("pointer-events", "all")
  .append('svg:g')
    .call(d3.behavior.zoom().on("zoom", redraw))
  .append('svg:g');
vis.append('svg:rect')
    .attr('width', w)
    .attr('height', h)
    .attr('fill', 'white');
function redraw() {
  console.log("here", d3.event.translate, d3.event.scale);
  vis.attr("transform",
      "translate(" + d3.event.translate + ")"
      + " scale(" + d3.event.scale + ")");
}
function drawGraph(graph) {
    var svg = d3.select("#force").append("svg")
        .attr("width", width)
        .attr("height", height);
    // draw plot background
    svg.append("rect")
        .attr("width", width)
        .attr("height", height)
        .style("fill", "#eeeeee");
    // create an area within svg for plotting graph
    var plot = svg.append("g")
        .attr("id", "plot")
        .attr("transform", "translate(" + pad + ", " + pad + ")");
    // https://github.com/mbostock/d3/wiki/Force-Layout#wiki-force
    var layout = d3.layout.force()
        .size([width - margin, height - margin])
        .charge(-120)
        .linkDistance(function(d, i) {
            return (d.source.group == d.target.group) ? 50 : 100;
        })
        .nodes(graph.nodes)
        .links(graph.links)
        .start();
    drawLinks(graph.links);
    drawNodes(graph.nodes);
    // add ability to drag and update layout
    // https://github.com/mbostock/d3/wiki/Force-Layout#wiki-drag
    d3.selectAll(".node").call(layout.drag);
    // https://github.com/mbostock/d3/wiki/Force-Layout#wiki-on
    layout.on("tick", function() {
        d3.selectAll(".link")
            .attr("x1", function(d) { return d.source.x; })
            .attr("y1", function(d) { return d.source.y; })
            .attr("x2", function(d) { return d.target.x; })
            .attr("y2", function(d) { return d.target.y; });
        d3.selectAll(".node")
            .attr("cx", function(d) { return d.x; })
            .attr("cy", function(d) { return d.y; });
    });
}
    function tick(e) {
  // Push different nodes in different directions for clustering.
  var k = 6 * e.alpha;
  graph.nodes.forEach(function(o, i) {
    o.y += i & 1 ? k : -k;
    o.x += i & 2 ? k : -k;
  });
  node.attr("cx", function(d) { return d.x; })
      .attr("cy", function(d) { return d.y; });
}
// Draws nodes on plot
function drawNodes(nodes) {
    // used to assign nodes color by group
    var color = d3.scale.category20();
    // https://github.com/mbostock/d3/wiki/Force-Layout#wiki-nodes
    d3.select("#plot").selectAll(".node")
        .data(nodes)
        .enter()
        .append("circle")
        .attr("class", "node")
        .attr("id", function(d, i) { return d.name; })
        .attr("cx", function(d, i) { return d.x; })
        .attr("cy", function(d, i) { return d.y; })
        .attr("r",  function(d, i) { return 4; })
        .style("fill",   function(d, i) { return color(d.group); })
        .on("mouseover", function(d, i) { addTooltip(d3.select(this)); })
        .on("mouseout",  function(d, i) { d3.select("#tooltip").remove(); });
}
// Draws edges between nodes
function drawLinks(links) {
    var scale = d3.scale.linear()
        .domain(d3.extent(links, function(d, i) {
           return d.value;
        }))
        .range([1, 6]);
    // https://github.com/mbostock/d3/wiki/Force-Layout#wiki-links
    d3.select("#plot").selectAll(".link")
        .data(links)
        .enter()
        .append("line")
        .attr("class", "link")
        .attr("x1", function(d) { return d.source.x; })
        .attr("y1", function(d) { return d.source.y; })
        .attr("x2", function(d) { return d.target.x; })
        .attr("y2", function(d) { return d.target.y; })
        .style("stroke-width", function(d, i) {
            return scale(d.value) + "px";
        })
        .style("stroke-dasharray", function(d, i) {
            return (d.value <= 1) ? "2, 2" : "none";
        });
}
</script>
</head>

<body>
<div align="center" id="force"></div>

<script>
d3.json("pcap_export.json", drawGraph);
</script>
</body>
</html>
{% endhighlight %}

  
It's easiest if the dataset and index.html are all in the same directory.




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
Now we need to get the data into a dataframe. If you've never used [Pandas](http://pandas.pydata.org/) before there is a great tutorial [here](http://pandas.pydata.org/pandas-docs/stable/10min.html).

Getting our data into a dataframe is simple with Panda's _read_csv_ module.

{% highlight python %}
import pandas as pd
import json
import re

pcap_data = pd.read_csv('packet_metadata_ipv4.csv', index_col='No.')
{% endhighlight %}

Verify data loaded properly

{% highlight python %}
dataframe = pcap_data
{% endhighlight %}
<br>


{% highlight python %}
dataframe
{% endhighlight %}


<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Time</th>
      <th>Source</th>
      <th>Src Port</th>
      <th>Destination</th>
      <th>Dst Port</th>
      <th>Protocol</th>
      <th>Length</th>
    </tr>
    <tr>
      <th>No.</th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>0.000000</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>TCP</td>
      <td>62</td>
    </tr>
    <tr>
      <th>2</th>
      <td>0.000035</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>TCP</td>
      <td>62</td>
    </tr>
    <tr>
      <th>3</th>
      <td>0.000225</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>TCP</td>
      <td>60</td>
    </tr>
    <tr>
      <th>4</th>
      <td>0.000455</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>HTTP</td>
      <td>360</td>
    </tr>
    <tr>
      <th>5</th>
      <td>0.000482</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>TCP</td>
      <td>54</td>
    </tr>
    <tr>
      <th>6</th>
      <td>0.000957</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>HTTP</td>
      <td>1315</td>
    </tr>
    <tr>
      <th>7</th>
      <td>0.003018</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>HTTP</td>
      <td>340</td>
    </tr>
    <tr>
      <th>8</th>
      <td>0.003181</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>TCP</td>
      <td>1514</td>
    </tr>
    <tr>
      <th>9</th>
      <td>0.003298</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>HTTP</td>
      <td>1194</td>
    </tr>
    <tr>
      <th>10</th>
      <td>0.003531</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>TCP</td>
      <td>60</td>
    </tr>
    <tr>
      <th>11</th>
      <td>0.039293</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>HTTP</td>
      <td>344</td>
    </tr>
    <tr>
      <th>12</th>
      <td>0.039672</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>HTTP</td>
      <td>579</td>
    </tr>
    <tr>
      <th>13</th>
      <td>0.092320</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>HTTP</td>
      <td>267</td>
    </tr>
    <tr>
      <th>14</th>
      <td>0.092593</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>HTTP</td>
      <td>575</td>
    </tr>
    <tr>
      <th>15</th>
      <td>0.303521</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>TCP</td>
      <td>60</td>
    </tr>
    <tr>
      <th>16</th>
      <td>4.466444</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>HTTP</td>
      <td>442</td>
    </tr>
    <tr>
      <th>17</th>
      <td>4.466706</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>TCP</td>
      <td>14654</td>
    </tr>
    <tr>
      <th>18</th>
      <td>4.466912</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>TCP</td>
      <td>60</td>
    </tr>
    <tr>
      <th>19</th>
      <td>4.466927</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>TCP</td>
      <td>16114</td>
    </tr>
    <tr>
      <th>20</th>
      <td>4.467095</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>TCP</td>
      <td>60</td>
    </tr>
  </tbody>
</table>

  
We'll want to structure our data in the same format as the infamous [miserables.json]('https://gist.github.com/fredbenenson/4212290#file-miserables-json')
  
Here is a sample of miserables.json

{% highlight json %}
json_data = {
  "nodes":[
    {"name":"Myriel","group":1},
    {"name":"Napoleon","group":1},
    {"name":"Mlle.Baptistine","group":1},
    {"name":"Mme.Magloire","group":1},
    {"name":"CountessdeLo","group":1},
  ],
  "links":[
    {"source":1,"target":0,"value":1},
    {"source":2,"target":0,"value":8},
    {"source":3,"target":0,"value":10},
    {"source":3,"target":2,"value":6},
    {"source":4,"target":0,"value":1},
    {"source":5,"target":0,"value":1},
  ]
}
{% endhighlight %}
  
### The data has 2 keys: <u>nodes</u> and <u>links</u>
  
  -Nodes: This data is used to create an object and give the node a name. The group represents the color.  
  -Links: The _source_ is used to identify the **index position** inside of the nodes list. For example "Napoleon" is in index position 1; same holds true for target. The value is the number of times the connection occurs.
  
### Let's get our PCAP data into the same format.

First, isolate source and destination.

{% highlight python %}

src_dst = dataframe[["Source","Destination"]]
{% endhighlight %}


Load 10 sample pieces of data from the dataframe to validate data.
{% highlight python %}

src_dst.sample(10)
{% endhighlight %}

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Source</th>
      <th>Destination</th>
    </tr>
    <tr>
      <th>No.</th>
      <th></th>
      <th></th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>58224</th>
      <td>10.16.92.103</td>
      <td>10.16.92.79</td>
    </tr>
    <tr>
      <th>37454</th>
      <td>10.16.92.103</td>
      <td>10.16.92.79</td>
    </tr>
    <tr>
      <th>22425</th>
      <td>10.16.92.79</td>
      <td>10.16.92.103</td>
    </tr>
    <tr>
      <th>72515</th>
      <td>10.16.92.79</td>
      <td>10.16.92.103</td>
    </tr>
    <tr>
      <th>124518</th>
      <td>10.16.92.103</td>
      <td>10.16.92.79</td>
    </tr>
    <tr>
      <th>93352</th>
      <td>10.16.92.103</td>
      <td>10.16.92.79</td>
    </tr>
    <tr>
      <th>166810</th>
      <td>10.16.92.79</td>
      <td>10.16.92.103</td>
    </tr>
    <tr>
      <th>73159</th>
      <td>10.16.92.103</td>
      <td>10.16.92.79</td>
    </tr>
    <tr>
      <th>114681</th>
      <td>10.16.92.79</td>
      <td>10.16.92.103</td>
    </tr>
    <tr>
      <th>156581</th>
      <td>10.16.92.103</td>
      <td>10.16.92.79</td>
    </tr>
  </tbody>
</table>
<br/>

Filter out any hostnames that were included (may not apply to your dataset):
{% highlight python %}
def ip_matcher(address):
    # Used to validate if string is an ipaddress
    ip = re.match(
        '^(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$', address)
    if ip:
        return True
    else:
        return False

src_dst.rename(columns={"Source":"source","Destination":"target"}, inplace=True)
src_dst['valid_src'] = src_dst.source.apply(ip_matcher)
src_dst['valid_target'] = src_dst.target.apply(ip_matcher)

valid_src_dest = src_dst[(src_dst.valid_src==True) & (src_dst.valid_target==True)]
{% endhighlight %}

Group by source and target fields and count number of connections

_Use inplace=True to rename the columns inplace without having to reassign to a new variable._

{% highlight python %}


grouped_src_dst = valid_src_dest.groupby(["source","target"]).size().reset_index()


{% endhighlight %}

Join source and target into consolidated index to be used for index position
{% highlight python %}
unique_ips = pd.Index(grouped_src_dst['source']
                      .append(grouped_src_dst['target'])
                      .reset_index(drop=True).unique())
{% endhighlight %}

Create subnet group  
_Note: We use regular expression here to group the various subnets to the third octect. For example, if you have 2 IP addresses (192.168.1.5, 192.168.2.5), they'd both be treated as 2 networks. We'll use this to group the subnets by color and create our groups._

{% highlight python %}
group_dict = {}
counter = 0
for ip in unique_ips:
    breakout_ip = re.match("^(\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})$", ip)
    if breakout_ip:
        net_id = '.'.join(breakout_ip.group(1,2,3))
        if net_id not in group_dict:
            counter += 1
            group_dict[net_id] = counter
        else:
            pass

{% endhighlight %}
  
Next we'll need to begin to structure our data which to reference later.
  
{% highlight python %}
grouped_src_dst.rename(columns={0:'count'}, inplace=True)
temp_links_list = list(grouped_src_dst.apply(lambda row: {"source": row['source'], "target": row['target'], "value": row['count']}, axis=1))
{% endhighlight %}

You should now have something like...
{% highlight python %}

temp_links_list
[{'source': '0.0.0.0', 'target': '255.255.255.255', 'value': 157},
 {'source': '10.16.11.5', 'target': '10.25.22.253', 'value': 24},
 {'source': '10.16.92.103', 'target': '10.16.92.79', 'value': 105742},
 {'source': '10.16.92.79', 'target': '10.16.92.103', 'value': 36543},
 {'source': '10.2.2.2', 'target': '10.22.11.9', 'value': 3410},
 {'source': '10.2.2.2', 'target': '10.25.22.253', 'value': 57},
 {'source': '10.21.22.1', 'target': '10.21.22.22', 'value': 1},
 {'source': '10.21.22.1', 'target': '10.21.22.23', 'value': 1},
 {'source': '10.21.22.1', 'target': '10.21.22.24', 'value': 1},
 {'source': '10.21.22.1', 'target': '10.21.22.253', 'value': 19},
 {'source': '10.21.22.10', 'target': '10.21.22.22', 'value': 54},
 {'source': '10.21.22.10', 'target': '10.21.22.23', 'value': 96},
 {'source': '10.21.22.10', 'target': '10.21.22.24', 'value': 156},
 {'source': '10.21.22.10', 'target': '10.21.22.253', 'value': 14},
 {'source': '10.21.22.22', 'target': '10.21.22.1', 'value': 3},
 {'source': '10.21.22.22', 'target': '10.21.22.10', 'value': 40},
 {'source': '10.21.22.22', 'target': '10.21.22.23', 'value': 6},
 {'source': '10.21.22.22', 'target': '10.21.22.24', 'value': 6},
 {'source': '10.21.22.22', 'target': '10.21.22.253', 'value': 20}]
 {% endhighlight %}
  
  Now we need to extract the index location for each unique source and destination (target) pair and append it to our links list.
{% highlight python %}

links_list = []
for link in temp_links_list:
    record = {"value":link['value'], "source":unique_ips.get_loc(link['source']),
     "target": unique_ips.get_loc(link['target'])}
    links_list.append(record)

{% endhighlight %}


Now that we have our links list, we'll need to create our nodes.

{% highlight python %}

nodes_list = []

for ip in unique_ips:
    breakout_ip = re.match("^(\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})$", ip)
    if breakout_ip:
        net_id = '.'.join(breakout_ip.group(1,2,3))
        nodes_list.append({"name":ip, "group": group_dict.get(net_id)})
{% endhighlight %}

Our **nodes_list** contains the IPs which we isolated earlier in **unique_ips**

_validate data_
{% highlight python %}
nodes_list[0:8]

[
   {'group': 1, 'name': '0.0.0.0'},
   {'group': 2, 'name': '10.16.11.5'},
   {'group': 3, 'name': '10.16.92.103'},
   {'group': 3, 'name': '10.16.92.79'},
   {'group': 4, 'name': '10.2.2.2'},
   {'group': 5, 'name': '10.21.22.1'},
   {'group': 5, 'name': '10.21.22.10'},
   {'group': 5, 'name': '10.21.22.22'}
 ]
{% endhighlight %}

You should now see the index positions of the values instead of the values themselves represented in the links_list.

_validate data_

{% highlight python %}
links_list

[
   {'source': 0, 'target': 58, 'value': 157},
   {'source': 1, 'target': 23, 'value': 24},
   {'source': 2, 'target': 3, 'value': 105742},
   {'source': 3, 'target': 2, 'value': 36543},
   {'source': 4, 'target': 11, 'value': 3410},
   {'source': 4, 'target': 23, 'value': 57},
   {'source': 5, 'target': 7, 'value': 1},
   {'source': 5, 'target': 8, 'value': 1},
   {'source': 5, 'target': 9, 'value': 1},
   {'source': 5, 'target': 10, 'value': 19},
   {'source': 6, 'target': 7, 'value': 54},
   {'source': 6, 'target': 8, 'value': 96},
   {'source': 6, 'target': 9, 'value': 156},
   {'source': 6, 'target': 10, 'value': 14},
   {'source': 7, 'target': 5, 'value': 3},
   {'source': 7, 'target': 6, 'value': 40},
   {'source': 7, 'target': 8, 'value': 6},
   {'source': 7, 'target': 9, 'value': 6},
   {'source': 7, 'target': 10, 'value': 20}
 ]
 {% endhighlight %}

Time to prep our data to be loaded as a json and rendered in d3. This moves us into the next phase...

Step 3: Load Data
---

Create a variable called json_prep and assign our two list as the values.
{% highlight python %}
json_prep = {"nodes":nodes_list, "links":links_list}

json_prep.keys()
   dict_keys(['links', 'nodes'])
{% endhighlight %}

_validate data (data sample)_
{% highlight python %}
json_dump = json.dumps(json_prep, indent=1, sort_keys=True)
print(json_dump)

{% endhighlight %}


{% highlight json %}

{
 "links": [
  {
   "source": 0,
   "target": 58,
   "value": 157
  },
  {
   "source": 1,
   "target": 23,
   "value": 24
  }
 ],
 "nodes": [
  {
   "group": 1,
   "name": "0.0.0.0"
  },
  {
   "group": 2,
   "name": "10.16.11.5"
  }
 ]
}
{% endhighlight %}
_**Looks good!**_

Finally let's write our data out to a file to be used in our D3 Force Directed Graph

{% highlight python %}

filename_out = 'pcap_export.json'
json_out = open(filename_out,'w')
json_out.write(json_dump)
json_out.close()

{% endhighlight %}


Finally, start a python webserver

![Start Webserver]({{ site.base }}/images/start_webserver_python.png)

Open http://localhost:8000/index.html in your favorite web browser and view your network diagram!


Notebook
--

For convenience, I've included a copy of a <a href="https://github.com/austin-taylor/austin-taylor.github.io/blob/master/static/examples/force_directed/pcap_to_d3_visual.ipynb">jupyter notebook</a> for you to follow along.



Bonus Examples
--
One caveat to the force directed diagram is it's scalability. If you have a very large network you might run into browser performance issues. Here is an example of the largest diagram I have been able to render. 

![Interactive Force Directed Network Graph]({{ site.base }}/images/zoomed_out1.png)

Hope you have found this helpful. Please leave any questions in the comments below.

