---
layout: post
title: "Cleaning up KDE's metadata - the little things matter too"
---

## Cleaning up KDE's metadata - the little things matter too

Lots of my KDE contributions revolve around plugin code and their metadata, meaning I have a good overview of where and how metadata is used.
In this post, I will highlight some recent changes and show you how to utilize them in your Plasma Applets and KRunner plugins!

### Applet and Containment metadata

Applets (or Widgets) are one of Plasma's main selling points regarding customizability.
Next to user-visible information like the name, description and categories, there is a need for some technical metadata properties.
This includes `X-Plasma-API-Minimum-Version` for the compatible versions, the ID and the package structure, which should always be "Plasma/Applet".

For integrating with the system tray, applets had to specify the `X-Plasma-NotificationArea` and `X-Plasma-NotificationAreaCategory` properties.
The first one says that it may be shown in the system tray and the second one says in which category it belongs. But since we don't want any applets
without categories in there, the first value is redundant and may be omitted! Also, it was treated like a boolean value, but only `"true"` or `"false"` were expected.
I stumbled upon this when correcting the types in metadata files.  
This was noticed due to preparations for improving the JSON linting we have in KDE. I will resume working on it and might also blog about it :).

What most applets in KDE specify is `X-KDE-MainScript`, which determines the entrypoint of the applet.
This is usually "ui/main.qml", but in some cases the value differs. When working with applets, it is confusing to first have to look at the
metadata in order to find the correct file. This key was removed in Plasma6 and the file is always `ui/main.qml`.
Since this was the default value for the property, you may even omit it for Plasma5.  
The same filename convention is also enforced for QML config modules (KCMs).

What all applets needed to specify was `X-Plasma-API`. This is typically set to `"declarativeappletscript"`,
but we de facto never used its value, but enforced it being present. This was historically needed because in Plasma4, applets could
be written in other scripting languages. From Plasma6 onward, you may omit this key.

In the Plasma repositories, this allowed me to clean up over 200 lines of JSON data.


#### metadata.desktop files of Plasma5 addons

Just in case you were wondering: We have migrated from metadata.desktop files to only metadata.json files in Plasma6.
This makes providing metadata more consistent and efficient. In case your projects still use the old format, you can run `desktoptojson -i pathto/metadata.desktop`
and remove the file afterward.  
See https://develop.kde.org/docs/plasma/widget/properties/#kpackagestructure for more detailed information. You can even do the conversion when targeting Plasma5 users!

### Default object path for KRunner DBus plugins

Another nice addition is that "/runner" will now be the default object path. This means you can omit this one key.
Check out the template to get started: https://invent.kde.org/frameworks/krunner/-/tree/master/templates/runner6python.
DBus-Runners that worked without deprecations in Plasma5 will continue to do so in Plasma6!
For C++ plugins, I will make a porting guide like blog post soonish, because I have started porting my own plugins to work with Plasma6.

----

Finally, I want to wish you all a happy and productive new year!
