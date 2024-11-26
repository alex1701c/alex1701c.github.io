---
layout: post
title: "Profiling & Optimizing KRunner"
---

## Profiling & Optimizing KRunner

One central topic of this year's Akademy was energy efficiency and performance of software. I took this occasion to give KRunner another look in regard to profiling, because the multithreading refactor simplified lots of plugin code and allowed for more optimizations.

When I did some benchmarking around two years ago, one of my major performance surprises was the windowed widgets runner. This runner queried all available applets for each letter typed. https://invent.kde.org/plasma/plasma-workspace/-/merge_requests/322/ reused the queried metadata for the duration of the match session.  
When I created the "Systemsettings" runner plugin, I took the same approach toward reusing data. But with some better understanding of performance and improved KRunner APIs, I took another stance at improving this and the “Applications” runner. I choose these two, because they are one of the most central plugins for the search experience. Also, the underlying data that is queries (config modules and applications) is quite large.

For the applications runner, I already did an optimization to reuse the loaded apps during the lifetime of the match session around a month ago. All the described improvements are based on top of that work. My approach was to benchmark the runner using hotspot, look at the results and identify unexpectably expensive calculations.

In both plugins, I noticed that checks for an application being authorized or not were surprisingly expensive. In systemsettings, we did the check twice, once by appending the old KF5 ".desktop" postfix to the plugin ID. This could simply be removed in KF6. For the applications runner, the check was done inside KService to determine if the application is hidden. With KF6, the concept of ServiceTypes has gone away, and thus we don't need this check anymore (https://invent.kde.org/frameworks/kservice/-/merge_requests/154).  
While cleaning up the visibility check in KService provided faster results, it is even better to check the visibility once we load the applications and not when we filter multiple times for each letter typed.

There were also some pretty significant gains in the systemsettings runner by reducing lookups of the KPluginMetaData JSON object. For example, the user-visible name can be read once inside the loop and then stored as a simple variable. Another issue was that we created a QueryMatch object and assigned some properties, before checking if the given config module matches the query. This was a simple fix and avoided lots of unneeded object creations and temporary memory allocations.

Some other smaller improvements were: 

- Systemsettings runner:
    - Reusing calculated values, like the query being split in individual words
    - Only checking the name for short queries, this also avoid results to appear "random"
- Applications runner:
    - Also avoid creating unneeded QueryMatch, though this affected far fewer cases
    - Reduce double writes/lookups for internal de-duplication check
    - Inline some trivial, internal methods
    - Use temporary variables for KService properties when we need it as a variable in the current scope anyway (avoiding usage of iterator on temporary)
    - Remove unneeded lambdas for simple filtering. This was a leftover for when we used KApplicationTrader for each letter typed
   - Remove unneeded X-KDE-More check, we don't use this in any meaningful way
   - Remove check if app may be shown on current platform. In case this check would return false, the app would be considered hidden.

## What it accomplished

Now you know about the technical changes, let's look at the actual performance measurements! In the original merge requests, I have benchmarked using a test application that is part of the KRunner repo. Since that required me manually typing though, I have later on created a tool for automatically running queries.

Systemsettings runner:
The total CPU cycles for matching and querying of the config module were reduced by around 30%. If one only looks at the CPU cycles for matching, the reduction was around two thirds. This means that data initialization was slightly improved, but is still the heaviest part. But once you have typed the first letter and the data is loaded, querying KRunner or the application launcher until it is closed is a lot more efficient!

**BEFORE**:  
![before.png](https://i.imgur.com/mvX3OMn.png)
 **AFTER**:  
![after.png](https://i.imgur.com/sWMqpcy.png)


Applications runner:
The total CPU cycles were reduced by 24% and unlike the other runner, data initialization for the match session took slightly longer. This is due to the check if the app should be shown or not being made during loading and not filtering. But partly due to this tradeoff, querying is almost 60% faster!

The real-world gains depend on the user behavior. For the system settings runner, I have queried “theme” and emulated closing the launcher. For the applications runner, I have used “firefox” instead. In case you have a longer query, the optimizations would come more into play.

**BEFORE**:  
![before.png](https://i.imgur.com/RurNMjt.png)
**AFTER**:  
![after.png](https://i.imgur.com/4JtAz7n.png)


Besides improving performance, I also made sure to simplify and clean up the code. Compared to the previous state, the changes have removed 40 lines in total. There is for sure room for further optimizations, but I intentionally decided against changes that would cause a complexity vs. performance tradeoff. 

I hope you enjoyed this read!


