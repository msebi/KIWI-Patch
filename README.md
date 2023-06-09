# KIWI-Patch

This repo contains the changes made to the KIWI browser in order to auto install an extension

## Setup 

Instructions on building kiwi can be found [here](https://github.com/kiwibrowser/src)

Work started from commit:

```
commit e71f9649c5d609c55029509f8c0af65bf761451a (HEAD, origin/master, origin/HEAD)
Author: Alexander Timin <altimin@chromium.org>
Date:   Wed Mar 17 21:51:41 2021 +0000

    Use perfetto::StaticString to plumb static strings in inspector_trace_events

    R=caseq@chromium.org
    BUG=1137154

    Change-Id: Ib0b865f44517433f9f2e4f8fa9c1008289193148
    Reviewed-on: https://chromium-review.googlesource.com/c/chromium/src/+/2769878
    Auto-Submit: Alexander Timin <altimin@chromium.org>
    Reviewed-by: Andrey Kosyakov <caseq@chromium.org>
    Commit-Queue: Andrey Kosyakov <caseq@chromium.org>
    Cr-Commit-Position: refs/heads/master@{#863975}
```

## Build (win 10, Ubuntu 20.20)

Chromium was built both on windows 10 (for testing purposes) and Ubuntu 20.20. A virtual machine image of Ubuntu 

was created and run in VirtualBox since KIWI can only be built on Linux. Building on windows was needed to speed up 

development. Instructions for building on windows 10 can be found [here](https://chromium.googlesource.com/chromium/src/+/HEAD/docs/windows_build_instructions.md) 

See this [link](https://chromium.googlesource.com/chromium/src/+/HEAD/docs/windows_build_instructions.md#why-is-my-build-slow) for speeding up the build as well 

as this [one](https://chromium.googlesource.com/chromium/src/+/HEAD/docs/windows_build_instructions.md#using-visual-studio-intellisense) for building and debugging using Visual Studio. 

For building on ubuntu, follow the steps [here](https://github.com/kiwibrowser/src#building).

Development was done in parallel (both on Win 10 and Ubuntu); it was more convenient to debug and test on windows 10 and then port the respective changes to Ubuntu.

Building in Ubuntu takes longer because of the extra layer added by the vm. 

## Notes 

All file paths are relative to the *source root directory* of chromium. For example,

`extensions\browser\extension_function_histogram_value.h`

is in 

`<path\to\chrome\clone\src>\extensions\browser\extension_function_histogram_value.h`

When looking for changes in the code base grep for the strings *"TELE"* and *"KIWI"* (no quotes) as all changes 

are prefixed and sufixed by these strings. 

A diff of the whole project can be found in `./kiwi_patch.diff`

## Structure 

Each change is described in a separate directory; each directory has a README.md that describes the change as well as the files 

that the respective change affects, the reason being that, at least at the time, managing forking/creating a repo the size of chromium 

was not possible (due to size constraints). 



