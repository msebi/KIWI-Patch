# KIWI-Patch
This repo contains the changes made to the KIWI browser in order to auto install an extension

## Setup 

## Notes 

All file paths are relative to the *source root directory* of chromium. For example,

`extensions\browser\extension_function_histogram_value.h`

is in 

`<path\to\chrome\clone\src>\extensions\browser\extension_function_histogram_value.h`

When looking for changes in the code base grep for the strings *"TELE"* and *"KIWI"* (no quotes) as all changes 

are prefixed and sufixed by these strings. 

## Structure 

Each change is described in a separate directory; each directory has a README.md that describes the change as well as the files 

that the respective change affects, the reason being that, at least at the time, managing forking/creating a repo the size of chromium 

was not possible (due to size constraints). 



