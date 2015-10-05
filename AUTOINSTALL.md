Summary: MacFUSE has a new install/update mechanism that greatly simplifies and improves things both for end users and for developers who use MacFUSE in their software.

# Background #

MacFUSE is a software building block. It provides an API that lets you easily create innovative and imaginative file systems. There are dozens of MacFUSE-based products out there: various combinations of open-source, closed-source, commercial, and "free" software. (The MacFUSE license is permissive enough to allow such a range.)

MacFUSE installation and updating raises some interesting issues, perhaps for everybody involved, but particularly for MacFUSE itself. You see, MacFUSE is unique in the Mac OS X world in that it's not just a regular "library" that developers build upon. Sure, there are libraries in MacFUSE, but there also is a kernel extension. Unlike libraries, which can be embedded within application bundles and made to "run" within private namespaces, kernel extensions load and run within the kernel's namespace. There's only one kernel, and the namespace we are talking about is system wide and is globally shared. Therefore, you can load and run only one copy of the MacFUSE kernel extension at a time.

# The Problem #

Why does this matter? Think about what happens when vendors of MacFUSE-based products all want to bundle their own copy of MacFUSE with their products. (This already is the case, by the way.) It's potentially chaotic: only one copy of MacFUSE will "win", but it may not be the "best" copy because these vendors might bundle different (perhaps outdated) MacFUSE versions. The MacFUSE version one vendor wants may not be desired by another. Vendors can also overwrite each other's MacFUSE installations. Worse, vendors could (inadvertently or otherwise) articifially change the version number of the MacFUSE distribution they install, throwing off future MacFUSE updates, making it harder to track bugs, and creating a maintenance nightmare.

Users could also install MacFUSE from various other sources such as MacPorts, Fink, and the multitude of Mac "Software Update" sites. As if all this weren't complex enough, there are separate MacFUSE binaries for Tiger and Leopard. This makes it even harder for vendors rolling in their own MacFUSE packages to get it "right".

Yes, it is understandable that vendors would like to ship MacFUSE along with their software. One alternative--that of "telling" the user to go download the official MacFUSE package--is simpler and saner, but unlikely to be acceptable to vendors. After all, user experience is supposed to be one or the hallmarks of the Mac platform, and we understand the desire to maintain great user experience.

In light of all this, we really do want to solve this problem once and for all: for MacFUSE users, for developers that build atop MacFUSE, and for ourselves.

# The Solution #

MacFUSE will now include a lightweight but sophisticated install/update engine. Like MacFUSE itself, this engine is open source. No, we are not reinventing any technologies that already exist in Mac OS X--the MacFUSE install/update engine uses standard Mac OS X package formats and installation tools. The functionality it provides does not exist in Mac OS X.

To begin with, users and developers will not have to deal with Tiger- and Leopard-specific MacFUSE packages. There will be only one official MacFUSE package. At the time of this writing, the current MacFUSE release is 1.7, which means there is one current MacFUSE package that can be found inside the official MacFUSE disk image (MacFUSE-1.7.dmg). The package contains both Tiger and Leopard versions of MacFUSE. Users can simply download and install this package--the appropriate version of MacFUSE for your platform will automatically be installed.

Actually, it's better than that. When installation begins, the MacFUSE package will attempt to look up the latest version of MacFUSE. To do so, it will attempt to retrieve a signed plist file that lives in the MacFUSE source tree (http://macfuse.googlecode.com/svn/trunk/CurrentRelease.plist). If it is determined that there is a version of MacFUSE that's newer than the one embedded in the MacFUSE package being installed, the built-in engine would download the newer version and install that instead. If this process fails (say, you have no network connectivity, etc.), installation would fall back to the version of MacFUSE embedded within the package. Even then, installation would not overwrite a newer MacFUSE version with an older MacFUSE version. To sum up, installation would try its best to install the "best" version of MacFUSE for your platform. This process works the same whether an end user directly installs the MacFUSE package or a vendor includes the MacFUSE package within a metapackage.

# Using the Install/Update Engine #

We just saw that MacFUSE installation knows how to find and install the latest version of MacFUSE. The built-in install/update engine also knows how to update MacFUSE to the latest version. The engine lives in MacFUSEAutoinstaller.bundle in the MacFUSE support directory. (`/Library/Filesystems/fusefs.fs/Support/` on Leopard and `/System/Library/Filesystems/fusefs.fs/Support/` on Tiger.) You can "drive" this engine using the `autoinstall-macfuse-core` command-line tool, which lives in the same directory.

```
$ /Library/Filesystems/fusefs.fs/Support/autoinstall-macfuse-core
Usage: autoinstaller -[plLiv]
  --print,-p    Print info about the currently installed MacFUSE
  --list,-l     List MacFUSE update, if one is available
  --plist,-L    List MacFUSE update in plist XML format
  --install,-i  Download and install MacFUSE update, if available
  --verbose,-v  Print VERY verbose output
```

You can use the tool to show some information on the currently installed MacFUSE version.

```
$ /Library/Filesystems/fusefs.fs/Support/autoinstall-macfuse-core -p
<KSTicket:0x116260
	productID=com.google.filesystems.fusefs
	version=1.7.1
	xc=<KSPathExistenceChecker:0x114390 path=/>
	url=http://macfuse.googlecode.com/svn/trunk/CurrentRelease.plist
	creationDate=2008-07-22 16:12:22 -0700>
```

Note that if you do NOT have a version of MacFUSE installed, but you somehow get/run the tool, the `-p` option would show version 0 as installed. This is as designed.

You can use the tool to check if there is an update available on the official MacFUSE site.

```
$ /Library/Filesystems/fusefs.fs/Support/autoinstall-macfuse-core -l
###  Update
Codebase='http://macfuse.googlecode.com/svn/releases/MacFUSE-1.9.dmg'
Predicate='SystemVersion.ProductVersion beginswith "10.5" AND Ticket.version != "1.9.1"'
Size='1745061'
Hash='rBINayKwMU9VaZTse1baolkKx1k='
Version='1.9.1'
ProductID='com.google.filesystems.fusefs'
```

If no update is available, the `-l` option will produce no output.

To download and install an update, use the `-i` option. This option requires superuser privileges.

```
$ sudo /Library/Filesystems/fusefs.fs/Support/autoinstall-macfuse-core -i
$
```

Messages are logged to `/tmp/.macfuse-install.log`. You can use the `-v` option to generate more verbose messages.

# Developer Releases #

I want to emphasize that the best version of MacFUSE is the latest one. The development model is that with a new MacFUSE release, the previous version immediately becomes obsolete from a support standpoint. Some developers might be worried about a new MacFUSE version "surprising" them or "breaking" their software.

To address this, there will be developer prereleases of MacFUSE before a regular release. These will allow developers to test their file systems against MacFUSE release candidates, and report bugs and compatibility issues before an official release happens. The install/update engine makes it possible to easily use these preleases. You can switch your MacFUSE installation to developer prereleases by creating a `/Library/Preferences/com.google.macfuse.plist` file that contains the following key.

```
$ cat /Library/Preferences/com.google.macfuse.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>URL</key>
  <string>http://macfuse.googlecode.com/svn/trunk/DeveloperRelease.plist</string>
</dict>
</plist>
```

`com.google.macfuse.plist` must be root owned and must not be world or group writable.

You can switch back to regular releases by removing this file. Note that the install/update engine will never overwrite a newer version of MacFUSE with an older one. Developer prereleases will always have a higher version number than the latest regular release. Therefore, if you want to switch back to a lower regular release number, you will have to manually uninstall the developer prerelease.