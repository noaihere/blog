---
layout: post
title:  "Singapore cov19 daily charts"
date:   2020-04-25 17:54:05 +0800
categories: charts
---

Head over to this [link][link] to see the application.


**Question**<br>
We are in the midst of a global cov19 pandemic. Each day, the ministry of health (moh) in Singapore releases a report on the number of new cases of infection and other related informaion.
An example of a daily report is [this link][link2]. Is it possible to extract key information from the text?
This would allow a quick view of the daily numbers without have to read the entire text and also allow us to build visualisations and do more analysis.
Can we also make it automated, so no human intervention is needed. Can the charts update daily based on new reports posted by moh.

**Process**<br>
We attempt to answer the question above in this application.<br><br>
There are multiple components needed to make this work.

1) Scraping the text report from moh website
2) Extracting key information from the text into a table like structure
3) Plotting the charts
4) Scheduling the jobs

**Details**<br>
We coded the application in python.

[link]: https://covdailycharts.herokuapp.com/
[link2]: https://www.moh.gov.sg/news-highlights/details/57-more-cases-discharged-1-016-new-cases-of-covid-19-infection-confirmed

