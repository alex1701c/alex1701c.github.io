---
layout: post
title: Clazy 1.15 Released – New Checks, Better Stability
date: 2025-06-15
---

🎉 **New Clazy Release: Stability Boost & New Checks!**

We’re excited to roll out a new Clazy release packed with bug fixes, a new check, and improvements to existing checks.
This release included 34 commits from 5 contributors.

---

## 🔍 New Features & Improvements

- **New Check: `readlock-detaching`**  
  Detects unsafe and likely unwanted detachment of member-containers while holding a read lock.
  For example, when calling `.first()` on the mutable member instead of `.constFirst()`

- **Expanded Support for Detaching Checks**  
  Additional methods now covered when checking for detaching temporary or member lists/maps.
  This includes reverse iterators on many Qt containers and `keyValueBegin`/`keyValueEnd` on QMap.
  All those methods have `const` counterparts that allow you to avoid detaching.

- **Internal Changes**
  With this release, Clang 19 or later is a required dependency. All older versions needed compatibility logic
  and were not thouroughly tested on CI. In case you are on an older Version of a Debian based distro, consider
  using https://apt.llvm.org/ and compile Clazy from source ;)

---

## 🐞 Bug Fixes

- **`install-event-filter`**: Fixed crash when no child exists at the given depth.  
  [BUG: 464372](https://bugs.kde.org/show_bug.cgi?id=464372)

- **`fully-qualified-moc-types`**: Now properly evaluates `enum` and `enum class` types.  
  [BUG: 423780](https://bugs.kde.org/show_bug.cgi?id=423780)

- **`qstring-comparison-to-implicit-char`**: Fixed and edgecase where assumptions about function definition were fragile.  
  [BUG: 502458](https://bugs.kde.org/show_bug.cgi?id=502458)

- **`fully-qualified-moc-types`**: Now evaluates complex signal expressions like `std::bitset<int(8)>` without crashing.
  [#28](https://invent.kde.org/sdk/clazy/-/issues/28)

- **`qvariant-template-instantiation`**: Crash fixed for certain template patterns when using pointer types.

---

Also, thanks to Christoph Grüninger, Johnny Jazeix, Marcel Schneider and Andrey Rodionov for contributing to this release!
