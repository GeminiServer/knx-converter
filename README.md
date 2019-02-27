# ets2openHAB 2.x conversion script

This is a repository of a experimental script to help moving from openHAB 1.x knx1 binding to the new openHAB 2.x knx
binding.  It converts existing knx1 item files and generates a knx.thing file.  Optionally it can also read the
`knxproj` file.

**Note:** This script only works if your are using config files.  It will not work if you use the UI for creating things
and items.  Furthermore it will most likely not work out of the box for everybody but may be a good start for whoever
wants to automate that process.

It has been tested with an **ETS 4** project file and runs on a Mac thus it should work on any Unix system.  Not sure about
Windows.

## Prerequisites

- You do need python3.
  I'm using Python 3.7.1.

## Installation

1. Make a backup.
2. Copy the bundle into a new directory.
3. Create a new sub-directory (e.g. `knxproj`) and enter it.
4. Export your `knxproj` file and unzip it into the sub-directory.
   Yes your `knxproj` project file is a zip file.
5. Search for a file  0.xml.
   In my case it is in the directory `knxproj/P-02A7/0.xml`.
6. Read and adjust the **config.py** file.
7. To run it call: `./convert-knx.py`

Steps 3-5 are optional.  You may also use the script w/o an ETS project file.  Then all your items will be assigned to a
generic device in the thing file and no controls are added.

Due to a general architecture change of the knx binding you only get events in OpenHAB if the item in KNX changes.
Furthermore you will not get any events from wall mounted switches or alike.  To overcome this problem you need so
called control items.  These will be created by this script automatically.  See below for details.

If you have an OH Item (group address) with exactly one *actor* assigned in your ETS the correct device ID will be used
in the thing file and items file.  If there is more than one actor in ETS assigned (e.g. ALL_LIGHTS_SWITCH) it will be
added to the generic device.

If one or more *control* (e.g. a wall mounted switch) is assigned in the ETS a *control* item is created in the generic
section and will be added to your items file directly below the main item.

All other items read from your ETS file which are not used in your items files will be written to an alternative file.
These can be used for copy & paste if you need to add items at a later stage.  Adjust **config.py** for your preferred
names:

```python
# knx1 openhab item file(s)
ITEMS_FILES = "../items/knx1/myhome.items , \
    ../items/knx1/heating.items, \
    ../items/knx1/window.items, \
    ../items/knx1/xbmc.items"

# converted item files will be created in this directory
ITEM_RESULT_DIR = "../items/"

# out file names
THINGS_FILE = "../things/knx.things"

THINGS_UNUSED_FILE = "unused.things"
ITEMS_UNUSED_FILE = "unused.items"
ITEMS_UNUSED_CONTROLS_FILE ="unused-control.items"
```

You definitly need to adjust the following settings that the script can determine what are your actors and what are your
controls.  This may be improved in future by extracting more information from the *knxproj* files.  See *TODOS.md*.

```python
# These are the primary addresses which will be used for read/write
ACTORS = "AKS, AKD, JAL, M-0051_H-hp, QUAD,"

# These will be added as -control items
CONTROLS = "TSM, -BE, ZN1IO, ZN1VI, LED,"

# These will ignored, uncomment to use
# IGNORE_DEVICES = "LED,"
```

Depending on your setup the following message may be ok if you have a *read-only* device or a dummy group address.  Or
it may mean that you need to adjust the above settings.

> INFO: No Actor found for: 4/0/24   	using: generic	BWM_Aussen_Garage

To find the names of your ETS components you can look into the files containing debug information about all your knx
and openhab items after a 1st run of this script.

```python
# files containing all information read
DEBUG_KNX = "knx.txt"
DEBUG_OH = "oh.txt"
```

If you have any issue feel free to contact me or open an issue in the repository.

Have fun...
