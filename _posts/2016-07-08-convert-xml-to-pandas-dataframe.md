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

TLDR;
-----

{% highlight python %}
import xml.etree.ElementTree as ET
from lxml import etree
import pandas as pd

def xml2df(xml_data):
    tree = ET.parse(xml_data)
    root = tree.getroot()
    all_records = []
    headers = []
    for i, child in enumerate(root):
        record = []
        for subchild in child:
            record.append(subchild.text)
            if subchild.tag not in headers:
                headers.append(subchild.tag)
        all_records.append(record)
    return pd.DataFrame(all_records, columns=headers)
{% endhighlight %}

Our Goal
---
Convert any XML file into a pandas dataframe.

I found a lot of examples on the internet of how to convert XML into DataFrames, but each example was very tailored. Our version will take in any XML file and format the headers properly.

For this example, we're going to convert [a User Agent tracker XML feed](http://www.user-agents.org/allagents.xml)

Sample Data
-----------
{% highlight xml %}
<user-agents>
<user-agent>
<ID>id_a_f_3</ID>
<String>!Susie (http://www.sync2it.com/susie)</String>
<Description>Sync2It bookmark management & clustering engine</Description>
<Type>C R</Type>
<Comment/>
<Link1>http://www.sync2it.com</Link1>
<Link2/>
</user-agent>
<user-agent>
{% endhighlight %}


Code Walkthrough
----------------

It's very fairly straight forward, so I'll comment each line to explain.

{% highlight python %}
'''BEGIN IMPORTS'''
import xml.etree.ElementTree as ET
from lxml import etree
import pandas as pd
'''END IMPORTS'''

def xml2df(xml_data):
    tree = ET.parse(xml_data) #Initiates the tree Ex: <user-agents>
    root = tree.getroot() #Starts the root of the tree Ex: <user-agent>
    all_records = [] #This is our record list which we will convert into a dataframe
    headers = [] #Subchildren tags will be parsed and appended here
    for i, child in enumerate(root): #Begin looping through our root tree
        record = [] #Place holder for our record
        for subchild in child: #iterate through the subchildren to user-agent, Ex: ID, String, Description.
            record.append(subchild.text) #Extract the text and append it to our record list
            if subchild.tag not in headers: #Check the header list to see if the subchild tag <ID>, <String>... is in our headers field. If not append it. This will be used for our headers.
                headers.append(subchild.tag)
        all_records.append(record) #Append this record to all_records.
    return pd.DataFrame(all_records, columns=headers) #Finally, return our Pandas dataframe with headers in the column.
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

If you found this helpful, please feel free to share and comment below.
