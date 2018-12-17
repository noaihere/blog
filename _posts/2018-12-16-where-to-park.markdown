---
layout: post
title:  "Where to park"
date:   2018-12-16 17:54:05 +0800
categories: API
---

Head over to this [link][link] to see the application.


**Problem**<br>
In city Singapore, it is sometimes a hassle to find a parking lot. We waste time and petrol searching for one.
How good would it be if we could know where to park.

**Description**<br>
This is a application that allows users to know where to park using data from LTA DataMall API.<br><br>
*Map*<br>
Users search for a specific parking location and the map will show the number of available parking lots for that location (centre of map) and surrounding locations.
The symbol will be red if the number of parking lots is less than 5 and green otherwise.
It also shows the distance from the searched parking lot in metres. 
<br><br>
*Table*<br>
Users can also look through a table to see the surrounding parking locations and the distances from their searched parking lot. 
The nearest ones are shown higher up in the table. They can also sort the locations by the number of available parking lots by clicking on the vertical arrow in the column header *AvailableLots*.
<br><br>
*How to use*<br>
If users' searched lot is available, they now know that they can park there. Otherwise users can find out which nearby parking lots are available and how far it is from their searched lot.
This is not a navigation app as the map is not detailed enough.<br><br>
*Data*<br>
The data will be refreshed every 20 seconds.
When a new location is selected, please wait for the loading<br>

**Conclusion**<br>
We now have a bird's eye, near real time view of available parking lots in Singapore.
In the future posts, I will describe the process of building this application. 


[link]: https://sgparkwhere.herokuapp.com/

