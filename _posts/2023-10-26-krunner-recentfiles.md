---
layout: post
title: "Reworking Recent Files search for Plasma 6"
---

## Reworking Recent Files search for Plasma 6

For Plasma 6, lots of KRunner plugins and framework functionalities were improved.
I took quite a bit of time to work on the recent files search plugin used by KRunner and Plasmas application launchers like Kickoff. This included performance improvements, usability improvements and technical refactorings.

Let's start off with the usability improvements: While the runner was named "Recent Files", it still provided results for directories.
In Dolphin on the other hand, "Recent Locations" are a separate location you may access next to "Recent Files".
Luckily, functionality for only querying files already exists in the KActivities-Stats framework :).  
The search for files was also improved. Natalie Clarius added some time ago a patch to avoid false positives when the query is part of the file path.
For example, when you type for "myfile" and "/home/user/myfiles/test" was a recent file, this file would be returned by the KActivities-Stats framework.
With Natalie's logic, those files would not be shown in KRunner, but still they fill out the limit of recent files we want to query.
Of course, one could look for more files in case we discarded too many, but that can cause a performance penalty due to additional SQL queries.  
Instead, I have added the ability to filter by the filename in KActivities-Stats. This means false positives are always avoided, and you will for sure get the results you are looking for!

Another useful, but not yet user-facing change is special handling for queries shorter than 3 characters. For those queries, a substring check would cause too many unintended results to be useful.
Instead, the filename must start with your given query. Similar to how the Applications-runner handles it.
There are plans to make this feature more easily accessible, but for now, it can only be done on the command line using `krunner --runner=krunner_recentdocuments`.  

![Example of single-plugin mode](https://i.imgur.com/PhMaas4.png)  
<em>Screenshot of KRunner in single-plugin mode with the recent files plugin</em>

The final issue that everyone will benefit from being fixed is a memory leak. This means for each letter typed, the runner would allocate a bit more RAM for results from the query. This can add up over time in KRunner and Plasmashell.
The KActivities-Stats that were "leaking" also caused CPU overhead, because they were notified about changes to the recent files list – even though they were never used again.  
Luckily, you don't have to worry about that at all from now on :).

### Optimizations

Let us start with the small improvements: In my previous posts, I mentioned that constructing a QueryMatch object before being sure that the
current item (application or systemsettings module) matches, causes a performance overhead.
The same applied to the recent files plugin, because we discarded some files due to the false positive detection mentioned above.  
Getting the filename was also optimized/simplified, because instead of using Qt API to get the filename from the URL, we can just use the resource title from KActivities-Stats.  
String comparisons were speed up by reusing the results. For example, when checking if the filename contains the query, we can get the resulting index. If that index is 0, the query is contained and the filename also starts with it.
Meaning we'd only do one string comparison for each recent file instead of up to 4.

Getting the appropriate icon is also way faster, because previously a method from KIO was used, which gets the file path, checks some extra cases, but usually gets the icon for the determined mime type.
Because the extra cases were not relevant when only allowing files, we can directly get the icon for the respective mimetype. Luckily, we don't even need to determine the mimetype, because the model already contains it.

#### Reusing previously fetched data
This is the most interesting part and was my original idea to improve performance: In KDE Frameworks 6, each runner lives in its own thread. Meaning, one may access member variables in a thread safe way.
The data we want to reuse is the KActivities-Stats ResultsModel. This contains paths and additional metadata of recently used files. The number of files is limited to 20, which is the maximum number of entries is KRunner.  
Implementing was straightforward: If one previously typed "firef" and after that "firefox", the previous model can be reused as long as the limit of 20 did not exceed the previously found results.
In addition to reusing the data, it is required to check the filenames again, because a file called "firef.test" should not match a query called "firefox".
Due to the preexisting logic to determine the match relevance, this is minimal additional work.  
When measuring the real-world impact, it is important to note that fetching results from KActivities SQLite database is the most expensive part. Accessing the model is relatively cheap.
Meaning the longer your queries are and if long as the number of results was not exceeded, your performance gains will be significant!

![Unoptimized](https://i.imgur.com/9x605pY.png)
<em>Profiling of the unoptimized version, loading the correct Icon takes a large porting of CPU cycles</em>
![Optimized](https://i.imgur.com/bCxklII.png)
<em>Optimized version, CPU cycles are significantly reduced and loading of icons is barely distinguishable from other, minor costs</em>

In benchmarks, one is able to see that the CPU cycles with this patch are roundabout the same among the different measurements, whereas the CPU cycles before this patch correlated with the length of the query.
The query was typed in letter by letter, keep in mind that the runner skips queries shorter than 3 characters unless the single runner mode is activated.
For benchmarks, I decided against activating this mode, because it is not the normal usecase.

| Query | CPU Cycles before | CPU Cycles with change | Reduction |
| --- | --- | --- | --- |
| mytextfile | 1.73E+10 | 3.592E+09 | 79% |
| user | 8.291E+09 | 3.148E+09 | 62% |
| firefox | 1.325E+10 | 3.601E+09 | 73% |


