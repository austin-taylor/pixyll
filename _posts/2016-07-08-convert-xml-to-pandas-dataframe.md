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

_Updated June 10th, 2017: Special thanks to all the contributors in the comments section!_


TLDR;
-----

{% highlight python %}
import xml.etree.ElementTree as ET
import pandas as pd

xml_data = open('/path/user_agents.xml').read()

def xml2df(xml_data):
    root = ET.XML(xml_data) # element tree
    all_records = []
    for i, child in enumerate(root):
        record = {}
        for subchild in child:
            record[subchild.tag] = subchild.text
        all_records.append(record)
    return pd.DataFrame(all_records)

{% endhighlight %}



Our Goal
---
Convert XML file into a pandas dataframe.

I found a lot of examples on the internet of how to convert XML into DataFrames, but each example was very tailored. Our version will take in any XML file and format the headers properly.

For this example, we're going to convert [a User Agent tracker XML feed](http://www.user-agents.org/allagents.xml)

Sample Data
-----------
![Sample XML Data]({{ site.base }}/images/xml.png)

Before running the sure to download the xml file above. 

{% highlight bash %}
wget http://www.user-agents.org/allagents.xml
{% endhighlight %}


Code Walkthrough
----------------

It's fairly straight forward, so I'll comment each line to explain.

{% highlight python %}
# BEGIN IMPORTS
import xml.etree.ElementTree as ET
import pandas as pd
# END IMPORTS

xml_data = open('/path/user_agents.xml').read() #Loading the raw XML data

def xml2df(xml_data):
    root = ET.XML(xml_data) # element tree
    all_records = [] #This is our record list which we will convert into a dataframe
    for i, child in enumerate(root): #Begin looping through our root tree
        record = {} #Place holder for our record
        for subchild in child: #iterate through the subchildren to user-agent, Ex: ID, String, Description.
            record[subchild.tag] = subchild.text #Extract the text create a new dictionary key, value pair
        all_records.append(record) #Append this record to all_records.
    return pd.DataFrame(all_records) #return records as DataFrame
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
