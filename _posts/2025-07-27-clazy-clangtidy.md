---
layout: post
title: Bringing together Clazy and Clang-Tidy
tags: Clazy Clang-Tidy
---

# 🎉 Clazy Now Integrates with Clang-Tidy!

I am excited to announce a major improvement to the Clazy project: **Clazy now integrates seamlessly with Clang-Tidy**!

## 🧩 One Toolchain, All the Checks

Clazy now provides a plugin (on Unix `ClazyClangTidy.so`) that allows all its checks to run inside `clang-tidy`, unifying your static analysis workflow.
You no longer need to run two separate tools — just configure Clazy’s checks through `clang-tidy` itself.

This change needed quite a few refactorings to make the existing Clazy codebase more adaptable.
In total, changes were spread out to 9 different pull requests to gradually implement the needed changes.
Besides implementing the functionality, the testsuite was also adapted to ensure Clazy's `clang-tidy` provides proper results.

### ✅ How to Use

To load the plugin:

```sh
clang-tidy -load=ClazyClangTidy.so ...
```

> 🔒 If the plugin isn’t in a standard library path, either add it to your `LD_LIBRARY_PATH` or provide an absolute path to the plugin file.

Unfortunately, Clang-Tidy needs to have Clazy checks enabled explicitly and does not have a concept of "levels" to group checks.
While wildcards like `clazy-*` would also work, it enables all manual-level checks. Those have more false positives and can hurt performance.

As a helper, you can export environment variables containing the check names to concatenate the desired combination:

```bash
export CLAZY_LEVEL0=clazy-overloaded-signal,clazy-connect-by-name,clazy-connect-non-signal,clazy-qstring-comparison-to-implicit-char,clazy-wrong-qevent-cast,clazy-lambda-in-connect,clazy-lambda-unique-connection,clazy-qdatetime-utc,clazy-qgetenv,clazy-qstring-insensitive-allocation,clazy-fully-qualified-moc-types,clazy-unused-non-trivial-variable,clazy-connect-not-normalized,clazy-mutable-container-key,clazy-qenums,clazy-qmap-with-pointer-key,clazy-qstring-ref,clazy-strict-iterators,clazy-writing-to-temporary,clazy-container-anti-pattern,clazy-qcolor-from-literal,clazy-qfileinfo-exists,clazy-qstring-arg,clazy-empty-qstringliteral,clazy-qt-macros,clazy-temporary-iterator,clazy-wrong-qglobalstatic,clazy-lowercase-qml-type-name,clazy-no-module-include,clazy-use-static-qregularexpression
export CLAZY_LEVEL1=clazy-auto-unexpected-qstringbuilder,clazy-connect-3arg-lambda,clazy-const-signal-or-slot,clazy-detaching-temporary,clazy-foreach,clazy-incorrect-emit,clazy-install-event-filter,clazy-non-pod-global-static,clazy-post-event,clazy-qdeleteall,clazy-qlatin1string-non-ascii,clazy-qproperty-without-notify,clazy-qstring-left,clazy-range-loop-detach,clazy-range-loop-reference,clazy-returning-data-from-temporary,clazy-rule-of-two-soft,clazy-child-event-qobject-cast,clazy-virtual-signal,clazy-overridden-signal,clazy-qhash-namespace,clazy-skipped-base-method,clazy-readlock-detaching
export CLAZY_LEVEL2=clazy-ctor-missing-parent-argument,clazy-base-class-event,clazy-copyable-polymorphic,clazy-function-args-by-ref,clazy-function-args-by-value,clazy-global-const-char-pointer,clazy-implicit-casts,clazy-missing-qobject-macro,clazy-missing-typeinfo,clazy-old-style-connect,clazy-qstring-allocations,clazy-returning-void-expression,clazy-rule-of-three,clazy-virtual-call-ctor,clazy-static-pmf
```

Checks in Clang-Tidy can be disabled when prefixing them with "-", whereas Clazy uses "no-" prefixes.
An example clang-tidy command to use all level0 checks, with `overloaded-signal` being disabled and the `qt-keywords` manual check being enabled:

```bash
clang-tidy -load=ClazyClangTidy.so \
-checks="$CLAZY_LEVEL0,-overloaded-signal,qt-keywords" \
-p my_build_dir mydir/**.cpp
```

In case you want to speed up linting in the project, `run-clang-tidy` can be used for parallel execution.

### 🚧✨ Limitations & Tricks

Unlike using Clazy directly, clang-tidy has its own filter mechanism to only emit warnings from files that were provided as an input.
This means if a warning is emitted from a header file and not the ".cpp" file you provide as an input, Clang-Tidy will suppress it.
To see those warnings `-header-filter=".*"` can be added to the command.

### 💡💬 Getting it & Feedback

The Clang-Tidy plugin is currently not released, and some additional development on various checks is happening.
For trying it out, one has to compile the project from source. It is just a simple CMake setup - promise ;)
See the instructions for more details: <https://invent.kde.org/sdk/clazy/#build-instructions>

Any feedback and contributions are appreciated - let's make Clazy better together 😎.
Please report bugs or suggestions on <https://bugs.kde.org/enter_bug.cgi?product=clazy> or <https://invent.kde.org/sdk/clazy/-/issues>.
