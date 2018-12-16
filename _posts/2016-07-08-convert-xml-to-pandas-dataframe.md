---
layout:     post
title:      Convert XML structure into a Pandas DataFrame
date:       2016-07-08 19:06:30
time:   5
summary:    A quick and easy way to convert XML structure into a Pandas dataframe with headers.
categories: lxml python pandas XML dataframe
products:
 - top-level: Python 2
   arbitrary: Required Modules
   nested-products:
    - nested: Pandas, [lxml](https://docs.python.org/2/library/xml.etree.elementtree.html)
      sub-arbitrary: pip install pandas

author: austin_taylor
author_github: austin-taylor
---

_Tested with Python 3 and updated December 16, 2019: Special thanks to all the contributors in the comments section!_



TLDR;
-----

### Gather XML Data

{% highlight python %}

import requests

user_agent_url = 'http://www.user-agents.org/allagents.xml'
xml_data = requests.get(user_agent_url).content

{% endhighlight %}

### Parse XML Data
---

{% highlight python %}
import xml.etree.ElementTree as ET
import pandas as pd
class XML2DataFrame:

    def __init__(self, xml_data):
        self.root = ET.XML(xml_data)

    def parse_root(self, root):
        """Return a list of dictionaries from the text and attributes of the
        children under this XML root."""
        return [parse_element(child) for child in root.getchildren()]

    def parse_element(self, element, parsed=None):
        """ Collect {key:attribute} and {tag:text} from thie XML
         element and all its children into a single dictionary of strings."""
        if parsed is None:
            parsed = dict()

        for key in element.keys():
            if key not in parsed:
                parsed[key] = element.attrib.get(key)
            if element.text:
                parsed[element.tag] = element.text                
            else:
                raise ValueError('duplicate attribute {0} at element {1}'.format(key, element.getroottree().getpath(element)))           

        """ Apply recursion"""
        for child in list(element):
            self.parse_element(child, parsed)
        return parsed

    def process_data(self):
        """ Initiate the root XML, parse it, and return a dataframe"""
        structure_data = self.parse_root(self.root)
        return pd.DataFrame(structure_data)

xml2df = XML2DataFrame(xml_data)
xml_dataframe = xml2df.process_data()
{% endhighlight %}



Our Goal
---
Convert XML file into a pandas dataframe.

I found a lot of examples on the internet of how to convert XML into DataFrames, but each example was very tailored. Our version will take in most XML data and format the headers properly. Some customization may be required depending on your data structure.

For this example, we're going to convert [a User Agent tracker XML feed](http://www.user-agents.org/allagents.xml)

Sample Data
-----------
![Sample XML Data]({{ site.base }}/images/xml.png)

Before running the sure to download the xml file above. 
{% highlight python %}

import requests

user_agent_url = 'http://www.user-agents.org/allagents.xml'
xml_data = requests.get(user_agent_url).content

{% endhighlight %}

Code Walkthrough
----------------

It's fairly straight forward, so I'll comment each line to explain.

### Parse XML Data
---

{% highlight python %}
import xml.etree.ElementTree as ET
import pandas as pd
class XML2DataFrame:

    def __init__(self, xml_data):
        self.root = ET.XML(xml_data)

    def parse_root(self, root):
        """Return a list of dictionaries from the text and attributes of the
        children under this XML root."""
        return [parse_element(child) for child in root.getchildren()]

    def parse_element(self, element, parsed=None):
        """ Collect {key:attribute} and {tag:text} from thie XML
         element and all its children into a single dictionary of strings."""
        if parsed is None:
            parsed = dict()

        for key in element.keys():
            if key not in parsed:
                parsed[key] = element.attrib.get(key)
            if element.text:
                parsed[element.tag] = element.text                
            else:
                raise ValueError('duplicate attribute {0} at element {1}'.format(key, element.getroottree().getpath(element)))           

        """ Apply recursion"""
        for child in list(element):
            self.parse_element(child, parsed)
        return parsed

    def process_data(self):
        """ Initiate the root XML, parse it, and return a dataframe"""
        structure_data = self.parse_root(self.root)
        return pd.DataFrame(structure_data)

xml2df = XML2DataFrame(xml_data)
xml_dataframe = xml2df.process_data()

{% endhighlight %}

If all went well you should now have a DataFrame as seen below:

Converted DataFrame
-------------------
<table border="1" class="dataframe" style="font-size:12px">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>ID</th>
      <th>String</th>
      <th>Description</th>
      <th>Type</th>
      <th>Comment</th>
      <th>Link1</th>
      <th>Link2</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>id_a_f_3</td>
      <td>!Susie (http://www.sync2it.com/susie)</td>
      <td>Sync2It bookmark management &amp; clustering engine</td>
      <td>C R</td>
      <td>None</td>
      <td>http://www.sync2it.com</td>
      <td>None</td>
    </tr>
    <tr>
      <th>1</th>
      <td>id_a_f_6</td>
      <td>&lt;a href='http://www.unchaos.com/'&gt; UnChaos &lt;/a...</td>
      <td>UnCHAOS search robot</td>
      <td>R</td>
      <td>Site is dead</td>
      <td>http://www.unchaos.com/</td>
      <td>None</td>
    </tr>
    <tr>
      <th>2</th>
      <td>id_a_f_7</td>
      <td>&lt;a href='http://www.unchaos.com/'&gt; UnChaos Bot...</td>
      <td>UnCHAOS search robot</td>
      <td>R</td>
      <td>Site is dead</td>
      <td>http://www.unchaos.com/</td>
      <td>None</td>
    </tr>
  </tbody>
</table>


Thanks for reading!
-------------------

If you found this helpful, please feel free to share and comment below.
