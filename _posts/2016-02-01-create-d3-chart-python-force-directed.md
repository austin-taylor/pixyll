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



Our Goal
---

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
Now we need to get the data into a dataframe. If you've never used [Pandas](http://pandas.pydata.org/) before there is a great tutorial [here](http://pandas.pydata.org/pandas-docs/stable/10min.html).

Getting our data into a dataframe is simple with Panda's _read_csv_ module.

{% highlight python %}
import pandas as pd
import json
import re

pcap_data = pd.read_csv('packet_metadata_ipv4.csv', index_col='No.')
{% endhighlight %}

Verify data loaded properly

```python
dataframe = pcap_data
```
<br>
```
dataframe
```

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
      <th>Info</th>
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
      <td>2546  &gt;  80 [SYN] Seq=0 Win=64240 Len=0 MSS=14...</td>
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
      <td>80  &gt;  2546 [SYN, ACK] Seq=0 Ack=1 Win=14600 L...</td>
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
      <td>2546  &gt;  80 [ACK] Seq=1 Ack=1 Win=64240 Len=0</td>
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
      <td>GET / HTTP/1.1</td>
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
      <td>80  &gt;  2546 [ACK] Seq=1 Ack=307 Win=15544 Len=0</td>
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
      <td>HTTP/1.1 200 OK  (text/html)</td>
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
      <td>GET /default.css HTTP/1.1</td>
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
      <td>[TCP segment of a reassembled PDU]</td>
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
      <td>HTTP/1.1 200 OK  (text/css)</td>
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
      <td>2546  &gt;  80 [ACK] Seq=593 Ack=3862 Win=64240 L...</td>
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
      <td>GET /img/stripes.gif HTTP/1.1</td>
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
      <td>HTTP/1.1 404 Not Found  (text/html)</td>
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
      <td>GET /favicon.ico HTTP/1.1</td>
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
      <td>HTTP/1.1 404 Not Found  (text/html)</td>
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
      <td>2546  &gt;  80 [ACK] Seq=1096 Ack=4908 Win=63194 ...</td>
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
      <td>GET /files/TrafficSystemNetworkMap.pdf HTTP/1.1</td>
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
      <td>[TCP segment of a reassembled PDU]</td>
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
      <td>2546  &gt;  80 [ACK] Seq=1484 Ack=19508 Win=58400...</td>
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
      <td>[TCP segment of a reassembled PDU]</td>
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
      <td>2546  &gt;  80 [ACK] Seq=1484 Ack=35568 Win=42340...</td>
    </tr>
    <tr>
      <th>21</th>
      <td>4.467107</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>TCP</td>
      <td>16114</td>
      <td>[TCP segment of a reassembled PDU]</td>
    </tr>
    <tr>
      <th>22</th>
      <td>4.467273</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>TCP</td>
      <td>60</td>
      <td>2546  &gt;  80 [ACK] Seq=1484 Ack=51628 Win=26280...</td>
    </tr>
    <tr>
      <th>23</th>
      <td>4.467324</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>TCP</td>
      <td>19034</td>
      <td>[TCP segment of a reassembled PDU]</td>
    </tr>
    <tr>
      <th>24</th>
      <td>4.467507</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>TCP</td>
      <td>60</td>
      <td>2546  &gt;  80 [ACK] Seq=1484 Ack=70608 Win=7300 ...</td>
    </tr>
    <tr>
      <th>25</th>
      <td>4.467518</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>TCP</td>
      <td>7354</td>
      <td>[TCP Window Full] [TCP segment of a reassemble...</td>
    </tr>
    <tr>
      <th>26</th>
      <td>4.467639</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>TCP</td>
      <td>60</td>
      <td>[TCP ZeroWindow] 2546  &gt;  80 [ACK] Seq=1484 Ac...</td>
    </tr>
    <tr>
      <th>27</th>
      <td>4.467978</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>TCP</td>
      <td>60</td>
      <td>[TCP Window Update] 2546  &gt;  80 [ACK] Seq=1484...</td>
    </tr>
    <tr>
      <th>28</th>
      <td>4.467988</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>TCP</td>
      <td>2974</td>
      <td>[TCP segment of a reassembled PDU]</td>
    </tr>
    <tr>
      <th>29</th>
      <td>4.468165</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>TCP</td>
      <td>60</td>
      <td>2546  &gt;  80 [ACK] Seq=1484 Ack=80828 Win=840 L...</td>
    </tr>
    <tr>
      <th>30</th>
      <td>4.468730</td>
      <td>10.25.22.253</td>
      <td>2546</td>
      <td>10.25.22.250</td>
      <td>80</td>
      <td>TCP</td>
      <td>60</td>
      <td>[TCP Window Update] 2546  &gt;  80 [ACK] Seq=1484...</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>170545</th>
      <td>1319065.601918</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135877788 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170546</th>
      <td>1319065.601926</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135879136 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170547</th>
      <td>1319065.601929</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135880484 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170548</th>
      <td>1319065.601933</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135881832 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170549</th>
      <td>1319065.601937</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135883180 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170550</th>
      <td>1319065.602523</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135884528 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170551</th>
      <td>1319065.602531</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135885876 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170552</th>
      <td>1319065.602534</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135887224 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170553</th>
      <td>1319065.602536</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>653</td>
      <td>80  &gt;  51085 [ACK] Seq=135888572 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170554</th>
      <td>1319065.602540</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135889159 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170555</th>
      <td>1319065.602543</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135890507 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170556</th>
      <td>1319065.603154</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135891855 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170557</th>
      <td>1319065.603162</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135893203 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170558</th>
      <td>1319065.603166</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135894551 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170559</th>
      <td>1319065.603169</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135895899 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170560</th>
      <td>1319065.603172</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135897247 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170561</th>
      <td>1319065.603175</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135898595 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170562</th>
      <td>1319065.603289</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135899943 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170563</th>
      <td>1319065.603745</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135901291 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170564</th>
      <td>1319065.603752</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135902639 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170565</th>
      <td>1319065.603758</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135903987 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170566</th>
      <td>1319065.603765</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135905335 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170567</th>
      <td>1319065.603892</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135906683 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170568</th>
      <td>1319065.604386</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135908031 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170569</th>
      <td>1319065.604393</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135909379 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170570</th>
      <td>1319065.604396</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135910727 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170571</th>
      <td>1319065.604399</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135912075 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170572</th>
      <td>1319065.604513</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135913423 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170573</th>
      <td>1319065.604519</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>TCP</td>
      <td>1414</td>
      <td>80  &gt;  51085 [ACK] Seq=135914771 Ack=1 Win=404...</td>
    </tr>
    <tr>
      <th>170574</th>
      <td>1319065.635116</td>
      <td>10.16.92.79</td>
      <td>51085</td>
      <td>10.16.92.103</td>
      <td>80</td>
      <td>TCP</td>
      <td>66</td>
      <td>51085  &gt;  80 [ACK] Seq=1 Ack=135916119 Win=666...</td>
    </tr>
  </tbody>
</table>
<p>167829 rows × 8 columns</p>





```python
#isolate source and destination
src_dst = dataframe[["Source","Destination"]]
```


```python
src_dst.sample(10)
```



Step 3: Load Data
---

To be continued...
