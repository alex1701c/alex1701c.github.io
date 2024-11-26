---
layout: post
title: "Web-Search-Keywords in KRunner, tales of optimizations"
---

### Web-Search-Keywords in KRunner, tales of optimizations

In my last blog post about performance in KRunner I wrote about the applications and system settings runners. I did lots of further investigation and work, some of which is still in progress and not merged yet.  
When looking at the benchmark results in the amazing tool Hotspot, I noticed that operations in the “Web Search Keywords” runner were surprisingly heavy in terms of CPU usage.

Looking at the graph, one could see there being two nearly identical callstacks that are responsible for parsing desktop files, which contain the search keywords, texts, and URLs. This of course doesn't really make sense, because they were not changed while benchmarking.
The issue was quite simple: When the class where the parsed desktop files are stored is instantiated, the desktop files are loaded. In the method to load configuration, we read some settings like the default shortcut, but also reload the keywords. And due to the config being loaded after the class is created, it is done twice.  
The solution is trivial: Suppress the parsing when loading the config for the first time. In case we read the config a second time, it is due to us having changed settings or have added new keywords. Then we of course want to reload all entries.

![](https://i.imgur.com/xaBzxtI.png)
<em>Profiling excerpt from the match-method, which produces results for your typed query</em>

The other, far more tricky issue was that while we were only supposed to have one central engine for interacting with the web search keywords, we in fact had two of them! Meaning, while we have halved the times the desktop files are loaded, it is still done twice needlessly.

![](https://i.imgur.com/3I2hgtT.png)
<em>Profiling excerpt from all plugins being loaded (main thread)</em>

After some debugging, I noticed that the class was compiled into two different plugins. Surprising is that the Q_GLOBAL_STATIC macro does not really result in a global singleton, because it is being compiled into different plugins. Once that was clear, the fix was straightforward: Create a small helper-library that includes the necessary files and use this library in the two plugins. This also means that we don't build the containing classes twice, which should decrease build times.  
Loading the keywords took ~90 milliseconds on my system. On systems with a weaker CPU and without a SSD, this would take even longer. Because the loading was done twice in the main thread, KRunner startup should be a bit less sluggish.

Hopefully you enjoyed this read, and I am eager to further work on optimizing KRunner & Frameworks.
