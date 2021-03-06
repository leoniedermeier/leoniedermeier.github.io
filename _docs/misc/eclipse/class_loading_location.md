---
title: How to Determine Where a Java Class is Loaded From
tags: [eclipse]
category: eclipse
summary: "How to determine from where a java class is loaded in the debug view"
---

# Getting the Class Location

Sometimes it is useful to know from where a class was loaded.

This can be done by changing the detail formatter of the class to
~~~java
getClass().getProtectionDomain().getCodeSource().getLocation(); 
~~~

In the variables view of the debug perspective open the context menu:

![edit detail formatter](class_loading_location/setDetailFormatter.png "edit detail formatter"){:height="200px" }

Set the detail formatter

![set detail formatter](class_loading_location/detailFormatter.png "set detail formatter"){:height="300px" }


Set a breakpoint where an instance of the class is available.
The result can be seen in the variables view:

 ![](class_loading_location/classLocation.png){:height="300px" }

{: .danger title="Tip"}
Do not forget to reset the detail formatter afterwards

# References
* <http://henryranch.net/tutorials/how-to-determine-where-a-java-class-is-being-loaded-from/>
