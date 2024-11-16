---
layout: post
title: "Reviving and reworking the KRunner help"
date: 2022-01-19 10:00:00 -0000
---

## Reviving and reworking the KRunner help

Currently KRunner does not provide any usage information itself, there
is only an bit of documentation on [the user base wiki](https://userbase.kde.org/Plasma/Krunner). Back in the KDE4 days KRunner had 
help button, but fundamental changes in the architecture meant that the code could not simply be ported to the current version.
 ![KDE4 Krunner usage help](https://www.linux.com/images/stories/41373/kde-desktop-krunner-2.jpg)
Since the help system required reimplementation, it was decided to implement it as a plugin for KRunner's powerful plugin infrastructure.
This plugin produces results when one types `?` or `help`. To make this more discoverable a button is added, which
puts the text `?`  in the search field.
![new help funtionality](https://i.imgur.com/dsbxtL4.png)
By having it as a plugin it is reusable in every case KRunner is used,
for example the KWin overview effect, the Application Launcher or the Plasma-Mobile search.

But it is not only under the hood different from the KDE4 version - there 
are quite a few changes to improve the usability:

For example only one usage example is displayed for each runner. This way one
can get a better overview over the different runners.

Also, runners can display their description instead of the first possible usage.
For example the sessions-runner can log out, suspend, switch user or reboot the PC. This would be too much information for the simple overview. Which is why the description is shown instead.

If the plugin has a configuration module, you can launch it as an action. ![configure action](https://i.imgur.com/8p7T5OE.png)
Otherwise you would have needed to click the configure button on the left, search for the runner and
then click the configure button for the specific runner.

When one click on of a match or selects it and presses enter, a detailed page of all the available usages of this runner is displayed:
![detailed help info](https://i.imgur.com/SkP1oC8.png)

Here you get all the available usage information displayed.
For getting a better overview, the queries are marked in bold.
Internally, this uses the styled text from Qt and every runner which has multiline text can utilize this feature.

When one runs one of the matches, the suggested query gets put in the KRunner search field.
The placeholder text is selected so that you can immediately overwrite it with your query, still get a little hint what the runner expects.
![autocompleted text](https://i.imgur.com/DWXl0jD.png)

Hopefully you like this feature and can be even more productive with KRunner :-)

In case you have developed a KRunner plugin yourself, check out the docs: [for DBus runners](https://invent.kde.org/frameworks/krunner/-/blob/955bb1d340af0b495dd4a1f1b0a32fc203a764ce/src/data/servicetypes/plasma-runner.desktop#L82) or [for C++ plugins](https://api.kde.org/frameworks/krunner/html/classPlasma_1_1RunnerSyntax.html).

