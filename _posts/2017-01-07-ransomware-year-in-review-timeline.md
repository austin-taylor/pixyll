---
layout:     post
title:      Ransomware 2016 May-December Chronological Timeline
date:       2017-01-07 08:00:00
time:   5
summary:   Statistics and a visualization to illustrate the various ransomware events for the last half of 2016.

categories: ransomware visualization
author: austin_taylor
author_github: austin-taylor

---

<style>
     dl
     {
         width: 500px;
         background: #fff;
         border: 1px solid #000;
         padding: 5px 15px;
         text-align:center;
         margin-left:auto;
         margin-right:auto;
      }

      dt, dd
      {
         /*display: inline;*/

      }


</style>

<h2>Overview</h2>
[Privacy PC](http://privacy-pc.com/articles/ransomware-chronicle.html) released a comprehensive report on ransomware-related events covering a time frame of May through December 2016. The report is beautifully done. As an analyst I wanted to break down each category to get a better understanding of each group. Therefore, I wrote a quick script to scrape the data from the site and use it for a timeline visualization.

Below is a time graph which you can use to explore the ransomware events. The number next to each category is the event count. If you see an event that seems interesting, you can hover to get more information. Additionally, the chart is interactive and allows you to zoom for more precision. Enjoy!

<h1 class="center"> Ransomware Timeline 2.0</h1>
<h4 class="center">Total Events: 377</h4>
<hr>

<h2 class="center">Color Legend</h2>


<div id="legend" class="center">
<dl>
<dt class="eventblue"></dt>
<dd style="color: #4298c3;"><strong>New ransomware released: 173</strong></dd>
<dt class="eventorange"></dt>
<dd style="color: #f4b183;"><strong>Old ransomware updated: 104</strong></dd>
<dt class="eventred"></dt>
<dd style="color: #f06262;"><strong>Ransomware decrypted: 49</strong></dd>
<dt class="eventgreen"></dt>
<dd style="color: #a9d18e;"><strong>Other important ransomware related events: 51</strong></dd>
</dl>
</div>

{% include ransomware_timeline.html %}

Events per month
---
{: style="text-align: center;"}

| Month | Event Count |
|:-------------:|:-------------:|
|2016-12 |   86 |
|2016-11 |   70 |
|2016-09 |   43 |
|2016-10 |   43 |
|2016-06 |   39 |
|2016-08 |   38 |
|2016-07 |   31 |
|2016-05 |   27 |

There was a steady increase in Ransomware events throughout the year and I predict the trend will continue through 2017.

![Malware_Trend]({{ site.base }}/images/upward_trend.png){: class="center-image"}

The converted raw dataset is [available here]({{ site.base }}/static/ransomware_data_2016.csv)

Hope you found this post helpful and if you have any questions, please post them in the comments section below.







