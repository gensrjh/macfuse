# The Accessibility File System for MacFUSE #

To use the MacFUSE version of the Accessibility file system, you must have the MacFUSE Core package installed. Please read the [QUICKER\_START\_GUIDE](QUICKER_START_GUIDE.md) document to learn how to install that package.

[AccessibilityFS can be downloaded here](http://code.google.com/p/macfuse/downloads/list).

## Introduction ##

AccessibilityFS is a demonstration MacFUSE filesystem that exposes all of the applications that you are running from an accessibility perspective. It shows off some of the weird and wonderful things you can do with a file system.

## Using AccessibilityFS ##

Double click on the AccessibilityFS icon and it will mount the file system and show it to you in the Finder. You should be able to see all of the apps currently running as folders with their name and process id. By opening up these folders you can navigate through the user interface elements exposed by the individual applications.

This is interesting, but things get far more fun when you start playing with these things with the Terminal.

Try running 'xattr -l' on the various items and you will see what attributes that they expose. You can modify these attributes using 'xattr -w' and have the various elements react. If you want to send an action to an element, try using 'echo "AXPress" > element'. The 'AccessibilityFS.Actions' attribute will list the actions that various elements will respond to.

## Sample ##
```
Foo:/Volumes/Accessibility/TextEdit (1680) dmaclach$ xattr -l standard\ window\ \:\ Untitled/
AccessibilityFS.AXRole: AXWindow
AccessibilityFS.AXRoleDescription: standard window
AccessibilityFS.AXSubrole: AXStandardWindow
AccessibilityFS.AXTitle: Untitled
AccessibilityFS.AXFocused: NO
AccessibilityFS.AXParent: application : TextEdit
AccessibilityFS.AXChildren: { close button, zoom button, minimize button, scroll area, text, grow area }
AccessibilityFS.AXPosition: {542, 149}
AccessibilityFS.AXSize: {708, 496}
AccessibilityFS.AXMain: YES
AccessibilityFS.AXMinimized: NO
AccessibilityFS.AXCloseButton: close button
AccessibilityFS.AXZoomButton: zoom button
AccessibilityFS.AXMinimizeButton: minimize button
AccessibilityFS.AXToolbarButton: 
AccessibilityFS.AXProxy: 
AccessibilityFS.AXTitleUIElement: text
AccessibilityFS.AXGrowArea: grow area
AccessibilityFS.AXDefaultButton: 
AccessibilityFS.AXCancelButton: 
AccessibilityFS.AXDocument: 
AccessibilityFS.AXModal: NO
AccessibilityFS.Actions: { AXRaise }
Foo:/Volumes/Accessibility/TextEdit (1680) dmaclach$ xattr -w "AccessibilityFS.AXPosition" "{0,0}" standard\ window\ \:\ Untitled/
Foo:/Volumes/Accessibility/TextEdit (1680) dmaclach$ echo "AXPress" > close\ button
```

## History ##

| 2008-01-14 | Initial public release | 0.5 | dmaclach |
|:-----------|:-----------------------|:----|:---------|
| 2008-01-22 | Fixed up icon, and changed docs | 0.5.1 | dmaclach |