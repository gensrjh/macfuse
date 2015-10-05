_aka What Disk Arbitration is, and Why You Should Care_

**The issues described in this document are addressed to a large extent in MacFUSE Core version 0.1.7. If you are using an earlier release, please upgrade to the latest. The document is still here for posterity.**

# Introduction #

MacFUSE lives down deep at the Darwin kernel level, but needs to interact with the Core File Manager. It does so by talking to a framework called Disk Arbitration. It works well most of the time, but there are still issues with the current build that can cause funkiness with the Finder.

# Details #

## A Paranoid KEXT ##

MacFUSE lives a very tricky life and has to be accordingly paranoid. While it lives in the kernel, it implements a filesystem described by a process in userland (the "filesystem daemon"). This requires from MacFUSE a decent level of paranoia. What if the filesystem daemon isn't ready yet? What if it's crashed?

Most of the time, when the kernel asks MacFUSE for information about the file system, MacFUSE can just turn around and pass the request up to the filesystem daemon. However, there are several cases when it cannot.

One that's particularly important is during initialization. The initialization request from MacFUSE to the filesystem daemon is asynchronous, and while a reply is pending a `statfs` request may come in. Since MacFUSE isn't sure if the filesystem daemon is ready, it will return a zero-ed out buffer.

## The File Manager and Disk Arbitration ##

Now let's look at the difference between the classic MacOS view of the world and the Unix view of the world.

In the classic MacOS view (as in Windows), the filesystem is a forest of trees. There are multiple volumes, each rooting its own file hierarchy. In the classic Unix view, there is one filesystem hierarchy spanning multiple devices. Each separate device is grafted onto the tree at a _mountpoint_.

## Reconciliation ##

How does Mac OS X, which is based on Unix, reconcile these different approaches? Deep down, it implements the Unix-based one-filesystem approach. Multiple volumes have mountpoints in the `/Volumes` folder.

For the higher layers in the system (notably the Core File Manager), a manager called Disk Arbitration manages the view of multiple disks. While the Unix layer may just see one filesystem with multiple directories in `/Volumes`, Disk Arbitration knows that the multiple directories in `/Volumes` are really separate disks. It will notify its clients (including the Finder) when new volumes come online and go offline.

There are two special flags, specific to the Mac platform, that MacFUSE implements to handle this. The first is `-ovolname=name`, where `name` is the volume name you'd like to use.  While at the Unix layer volumes don't have names because they're just attached at mountpoints, at the higher level, every volume that Disk Arbitration knows about must have a name. MacFUSE will assign you a bland one if you don't care ("Fuse Volume n"), but if you use `-ovolname`, you can customize it.

The more interesting one is `-oping_diskarb`. When mounting a new disk, Disk Arbitration will remain blissfully ignorant of its existence until you tell it. The details are in Apple's [QA1491](http://developer.apple.com/qa/qa2006/qa1491.html). MacFUSE handles that for you, though. If you specify `-oping_diskarb`, MacFUSE will tell Disk Arbitration about your new volume, and it'll show up in the Finder.

## The Race Condition ##

There is, however, a subtle problem that exists today with MacFUSE and the `-oping_diskarb` flag. If you specify `-oping_diskarb`, MacFUSE will initialize the filesystem, and then, **without waiting for the reply from the filesystem daemon**, tell Disk Arbitration that there is a new disk. At this point, there's a race condition. As soon as the Finder recognizes that there is a new filesystem, it will `statfs` it. But is the filesystem daemon ready?

Suppose it is. Suppose the filesystem daemon starts up quickly, replies to the MacFUSE initialization call, and then the stat comes in. MacFUSE then will forward the call to the filesystem daemon, it'll reply, and all will be well.

But suppose the filesystem daemon isn't ready. If the `statfs` request comes in _before_ the filesystem daemon acknowledges the initialization request, MacFUSE has little choice but to return a zero-filled result buffer. And the Finder has a fit. It sees a file system with things like a zero block size and zero capacity, and it never fully recovers from the traumatic shock.

## What Is To Be Done? ##

The solution is, of course, to ensure that when `-oping_diskarb` is used, that the filesystem daemon is fully initialized before Disk Arbitration is notified of the new disk. This might depend on the filesystem in question, but we have some ideas on how a generic solution could be implemented that works in practice almost always. If we get around to implementing that solution, we will, of course, release the source.

But until then, we wanted to do a quick writeup to fill you guys in on what's going on behind the scenes.