---
layout: post
title: "Unifying the KRunner sorting mechanisms for Plasma6 & further plans"
date: 2024-11-14 10:00:00 -0000
---

# Unifying the KRunner sorting mechanisms for Plasma6 & further plans

In Plasma5, we had different sorting implementations for KRunner and Kicker. This had historical reasons, because Kicker only used a subset of the available KRunner plugins. Due to the increased reliability, we decided to allow all available plugins to be loaded. However, the model still hard-coded the order in which the categories are displayed.
This was reported in [this bug](https://bugs.kde.org/show_bug.cgi?id=431204) which received numerous duplicates.

To address this concern, I focused on refactoring and cleaning up KRunner as part of KDE Frameworks 6. Among the significant architectural changes was the integration of KRunner's model responsible for sorting into the KRunner framework itself. This integration enabled easier code sharing and simplified code maintenance. Consequently, the custom sorting logic previously present in Kicker could be removed.

### Further plans
Now you know some of the improvements that have been done, but more interesting might be the future plans!
While the sorting in KRunner was in lots of regards better than the one from Kicker, it still has some flaws.
For instance, tweaking the order of results from a plugin developer's perspective proved challenging, since rearranging categories could occur unintentionally.
Also, KRunner implements logic to prioritize often launched results. In practice, this did not work quite well, because it only changed one of two sorting factors that are basically the same  (sounds messy, I know :D).

The plan is to have two separate sorting values: One for the categories and one for the results within a category. This allows KRunner to more intelligently learn which categories you use more the most and prioritize them for further queries.

Another feature request to configure the sorting of plugins. With the described change, this is far easier to implement. Some of the visuals were already discussed at the Plasma sprint last month.

Stay tuned for updates!
