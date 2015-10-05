If the question you have isn't answered here, try posting it to the [macfuse](http://groups.google.com/group/macfuse) group.

# 1. Introductory Questions #

**Q 1.1. What is MacFUSE? Is it the same as FUSE on Linux?**

MacFUSE is software that allows you to write arbitrary file systems as user-space programs. You can think of it as a library for easily developing Mac OS X file systems. Another crude way to look at this would be to think of MacFUSE as something that makes Mac OS X work like a _microkernel_ for the purpose of writing/running file systems. MacFUSE has two major components: an in-kernel loadable file system and a user-space library (`libfuse`). The in-kernel file system is specific to Mac OS X and is **not** based on Linux FUSE. (Some of its code is based on the FreeBSD implementation of FUSE.) The user-space library (`libfuse`), which provides the developer-visible FUSE API, has numerous Mac OS X specific extensions and features.

**Q 1.2. What is "MacFUSE-x.y.z.dmg"?**

A: MacFUSE-x.y.z.dmg (where x, y, and z constitute the MacFUSE version number) is the disk image containing the official distribution of fundamental MacFUSE software you need to use other software built atop MacFUSE. The core MacFUSE software consists of the following components **when installed** on your system:

  * MacFUSE file system bundle (`/Library/Filesystems/fusefs.fs`)
  * MacFUSE Objective-C framework (`/Library/Frameworks/MacFUSE.framework`)
  * MacFUSE C-based libraries (`/usr/local/lib/*fuse*.dylib`) and headers (`/usr/local/include/fuse*`)
  * MacFUSE preference pane (`/Library/PreferencePanes/MacFUSE.prefPane`)
  * MacFUSE project templates for Xcode (`/Library/Application Support/Developer/Shared/Xcode/Project Templates/MacFUSE`)

**NB**: On Mac OS X 10.4 (Tiger), the file system bundle is installed under `/System/Library/Filesystems`/.

The `fusefs.fs` bundle in turn contains most of the core MacFUSE software. Besides bundle metadata, its components include:

  * MacFUSE kernel extension (`fusefs.kext`)
  * MacFUSE mount program (`mount_fusefs`)
  * MacFUSE kernel extension loader (`load_fusefs`)
  * MacFUSE auto-update mechanism (`autoinstall-macfuse-core`)
  * MacFUSE uninstall script (`uninstall-macfuse-core`)

You should not need to call the MacFUSE uninstall script manually any more should you need to uninstall. The MacFUSE preference pane uses this script. Also see Q 2.3.

**Q 1.3. I'm experiencing serious problems (fill in details of serious problems here) right after I installed MacFUSE. What can I do?**

A: Circumstantial evidence such as "right after I installed MacFUSE" is unfortunately not enough to go on. To realize why MacFUSE **cannot** cause the kind of problems people sometimes ascribe to it, you need to understand **what** MacFUSE is, and just as importantly, what it's **not**.

MacFUSE is not an application. It's not a background process. It's not something that launches automatically at boot time or any other time. It does not configure, reconfigure, or modify the normal working of OS X by itself. It does not alter settings of any OS X components besides placing its preference pane for System Preferences to use.

MacFUSE is a bunch of libraries and headers along with a kernel extension. Developers use MacFUSE APIs to write applications--just like how developers use Apple's Cocoa APIs to write applications. Just as Cocoa is not a user-tangible entity, MacFUSE isn't either.  (As an end user, you don't "run" or "launch" Cocoa, do you?)

The MacFUSE kernel extension is **not** loaded automatically at boot time or any other time. It's dynamically loaded when a MacFUSE-based application runs. Even when loaded, the kernel extension just occupies some memory--its code doesn't kick in until you actually **mount** a MacFUSE-based file system.

The MacFUSE updater runs only when explicitly asked by the user, say, by going to the MacFUSE preference pane.

With this context, no matter how convinced you are that "it happened right after/only when you installed MacFUSE", keep in mind that MacFUSE cannot do things such as the following:

  * Affect the display
  * Affect wireless networking
  * Slow down the computer
  * Affect the computer's sleep or hibernate cycles
  * Prevent the computer from booting
  * Affect Finder settings, or any other program's settings
  * Do anything to your computer's firmware
  * Do anything to your computer's optical drive
  * Cause HFS+ corruption or any other non-MacFUSE-based file system's corruption
  * Cause car trouble

**NB**: "MacFUSE" refers to the official version that you download from this web site. Everything I've said here should generally apply to other repackagings/redistributions of MacFUSE, but they aren't officially supported.

If you remain convinced that your woes are MacFUSE's fault, please provide more information than circumstantial evidence.

# 2. Install/Uninstall Questions #

**Q 2.1. What is the preferred way to install MacFUSE?**

A: The preferred way is to install the `MacFUSE.pkg` from the latest `MacFUSE-<version>.dmg` available from:

http://code.google.com/p/macfuse/downloads/list

**Q 2.2. How can I keep MacFUSE up-to-date?**

A: The tools to keep MacFUSE up-to-date (query the installed version, query for available updates, download and install updates, and so on) are built into MacFUSE. Simply launch the Mac OS X System Preferences application and go to the MacFUSE preference pane. It should tell you if there's and update and if so, will also let you download and install it.

**Q 2.3. How can I uninstall MacFUSE?**

A: Launch the Mac OS X System Preferences application and go to the MacFUSE preference pane. Click on the "Remove MacFUSE" button. This will uninstall all MacFUSE components except the preference pane itself.

You can keep the MacFUSE preference pane around should you decide to install MacFUSE again in the future. If you do wish to remove it, you do it just like how you would remove any other non-Apple preference pane: In System Preferences, control-click (right-click) on the MacFUSE icon and you will see "Remove ..." as an action.

**Only** if you had the "Show Beta Versions" button checked in the MacFUSE prefpane, you'll have a property list (plist) file remaining at this point: `/Library/Preferences/com.google.macfuse.plist`. To remove it, you should uncheck the button before you remove the prefpane as described above.

Note that an official (correct) MacFUSE installation will have a preference pane if you have MacFUSE 2.0 or later installed. If you have an older version of MacFUSE, you can uninstall as follows.


---

**Deprecated Installation Information**

Run the `uninstall-macfuse-core.sh` script that resides in the `Support` subdirectory of the MacFUSE file system bundle. The bundle itself resides in `/System/Library/Filesystems/` on Mac OS X 10.4.x and in `/Library/Filesystems/` on Mac OS X 10.5.x.

For example, to uninstall MacFUSE on Mac OS X 10.4.x, you would run the following command in the Terminal:

```
sudo /System/Library/Filesystems/fusefs.fs/Support/uninstall-macfuse-core.sh
```

To uninstall MacFUSE on Mac OS X 10.5.x and above, you would run:

```
sudo /Library/Filesystems/fusefs.fs/Support/uninstall-macfuse-core.sh
```

If the file system bundle in your MacFUSE installation doesn't have a `Support` subdirectory, that means you have an incredibly ancient version of MacFUSE. Please look for the uninstall script within the `fusefs.fs/` directory itself.

---


**Q 2.4. Am I required to reboot my system after installing MacFUSE? Why?**

A: No, you are not **required** to do so. See [this](http://www.osxbook.com/blog/2009/03/02/why-macfuse-installation-recommends-a-reboot/) for more details.

**Q 2.5. I'm using some software that needs an old/deprecated version of MacFUSE. I don't see that version available from your web site. What to do?**

A: Please don't use old/deprecated versions of MacFUSE. The only "good" version of MacFUSE is the latest one. Older versions become obsolete/unsupported as soon as a new release is out. It's an important goal of MacFUSE to not break existing applications written atop it. As you can read in the [AUTOINSTALL](AUTOINSTALL.md) wiki page, developers are encouraged to test their software against prerelease versions of MacFUSE to help meet this goal.

# 3. File System Specific Questions #

## ntfs-3g ##

**Q 3.1. Why is ntfs-3g's write performance so poor with MacFUSE on OS X?**

A: The ntfs-3g program opens and does I/O to the block device (`/dev/diskN`) of the NTFS volume in question. **Mac OS X** does not have a VM buffer cache for block devices when they are accessed in this way. That's the most overwhelming factor, because both metadata operations and file data I/O boil down to read/writes by ntfs-3g to the block device.

Suppose we somehow automagically provided unified buffer caching for block devices by essentially making a disk look like a giant file. Even then, OS X and its buffer cache is really happy only when you do I/O that is in units of page size (4KB) and aligned on a page boundary. To get the most out of the I/O subsystem in OS X, ntfs-3g (or any other program for that matter) would really want to do I/O in multiples of 4KB.

For comparison, you should try writing to an NTFS _disk image_--you will see that it's considerably faster because you do have some caching in that case.

There are versions of ntfs-3g available that have additional user-space caching with drastically improved performance. (See http://macntfs-3g.blogspot.com/).

**Q 3.2. After installing ntfs-3g, my Boot Camp volume stopped showing up in the Startup Disk preference pane. I am devastated. Is reinstalling Mac OS X, Boot Camp, and Windows the only recourse?**

A: Relax. The Startup Disk preference pane is simply filtering out (that is, not displaying) any mounted volumes that it doesn't consider bootable. Its definition of a Boot Camp volume includes that the mounted volume either be of type `msdos` or `ntfs`--this is hardcoded into the preference pane plugin. This doesn't mean your Boot Camp volume has become unbootable. It's merely not showing up in the GUI. You can hold the `opt` key during startup and choose the Windows partition to boot from. You can also remount it (read-only) using the NTFS file system built into Mac OS X and it should start showing up in Startup Disk.

**Q 3.3. After installing MacFUSE, I can't mount any disk images, optical discs, etc. What's going on?**

A: It's likely that you installed a broken ntfs-3g package you found on the Internet. The package might be interfering with the disk image/disc mounting process. Try removing the `/System/Library/Filesystems/ntfs-3g.fs/` directory. Remember that MacFUSE itself is not a "program" that runs--it's akin to a library. It's only when you actually run a MacFUSE-based file system that MacFUSE even comes into the picture.

## sshfs ##

**Q 3.3. I'm using sshfs and changed a file "externally" (not through sshfs/MacFUSE) on the server, but in the sshfs volume, I'm not seeing the changes. In fact, when I copy the file through sshfs, I get the old content, etc.**

A: MacFUSE itself isn't a distributed remote file system! It's a mechanism for building arbitrary file systems. If you change things "externally" to MacFUSE (like, a file on the remote server in the case of sshfs), in general, things need to be done proactively to tell MacFUSE that something has changed, otherwise you'll get such "incorrect" behavior. In particular, sshfs isn't meant to _replace_ things such as NFS, AFP, and SMB--it's meant to be a substitute when you don't have any remote file sharing access to a computer, but you do have sftp access. When you use sshfs, from the server's standpoint, you are just accessing it using sftp. It's not as if the server is going to notify an sshfs client of modifications by other clients.

To make sshfs (or any other file system) "catch up" better with "remote" changes, there are a few things you can do. Beginning with MacFUSE 1.9, you can use the `-o auto_cache` option. This would make MacFUSE purge a file's in-kernel buffer cache if a change in the file's size or modification time is detected.

A worse way to have this mode of operation (where you can change things remotely at any time) is to disable caching in sshfs (look at the various `-o cache` options in sshfs, in particular, `-o cache=no`). Then, additionally, you need to tell MacFUSE not to cache things on its end too. You can use the `-o nolocalcaches` option, which turns off readaheads, the unified buffer cache, and the pathname resolution cache (all in the kernel). At the cost of some overhead, which could be substantial in certain cases, this will give you the behavior where most requests will have to go to the server and will therefore have more up-to-date information. Note that `auto_cache` is vastly preferred over this approach.

An example command line for this mode of operation could look like the following:

```
$ sshfs user@host:/dir /tmp/ssh -ocache=no -onolocalcaches -ovolname=ssh
```

If you are developing a MacFUSE file system and you want to handle such scenarios better within your file system, you should consider mounting your file system with the `auto_cache` option. You also have the option of using the `fuse_purge_np()` library function directly, although it's likely to be overkill.

**Q 3.4. I found a bug in sshfs.app (the GUI) and/or I have a feature request for sshfs.app (the GUI). I want you to do something about it. What would it take?**

A: `sshfs.app` is not a supported product. It started life as a one-off GUI wrapper for a conference demo and that's pretty much where it remained. Issues specific to `sshfs.app` (the GUI wrapper) should **not** be reported under the "Issues" tab of the MacFUSE project page. `sshfs.app` is how it is (minimal, skeletal) because it was written only for a single demo.

It's fine by me if people discuss `sshfs.app` related matters in the mailing list, because unlike a formal "issue report", it doesn't create extra and unnecessary work for me.

Issues with `sshfs`/`sshfs-static` (the _real_ command-line program behind the GUI) _could_ be MacFUSE issues though, but it's often obvious that they're not. If you think you've found a bug in MacFUSE through your use of `sshfs`, then, by all means please do report it by opening a ticket.

There's a thread about this here:

http://groups.google.com/group/macfuse/t/5dbb1af9eb4b3e06


**Q 3.5. sshfs isn't reporting the correct "disk space" for the remote "volume"? It seems to have 1000GB or some such number hardcoded.**

A: Yes, indeed. Remember that there really isn't an sshfs "volume" per se: sshfs just uses SFTP to provide an apparently local view of a remote _directory_. SFTP doesn't give you disk usage or availability for such a remote _directory_, so sshfs doesn't really have a choice but to cook up some value.

# 4. Other Usage Questions #

**Q 4.1. Why do MacFUSE volumes show up with "server" (or "network volume") icons?**

A: To be precise, by default MacFUSE volumes show up as _nonlocal_ volumes, which the Finder unfortunately treats the same as "servers". It's a good question as to why MacFUSE normally tags its volumes as nonlocal. Some people think that in the case of disk-based file systems such as ntfs-3g, MacFUSE _must_ tag the volume as local. Well, let us see.

For a vfs to be local on Mac OS X, you need a "real" disk device--a `/dev/disk*` style node. Such a real disk device node in MacFUSE's case is problematic: at mount time, for a local volume, the kernel would _itself_ open the device node and pass it to MacFUSE. In doing so, the kernel would make sure that the device is not currently in use (for one, to disallow multiple mounts of the same device). This happens before control passes to MacFUSE and mounting can proceed. This would have been fine if the entire file system lived in the kernel, but in MacFUSE's case, the user-space file system program would _also_ want to (exclusively) open the disk device.

Moreover, Disk Arbitration also gets involved in mounting and unmounting "local" volumes--this may be undesirable in some cases.

Technical reasons aside, there are arguments to justify that all MacFUSE volumes are, in fact, "remote". (The semantics of "local" and "remote" are debatable, but that's another discussion.) As far as the MacFUSE vfs is concerned, any and all volumes are remote in that their backing store lives across the kernel-user boundary, in a program that encapsulates backing store knowledge. MacFUSE doesn't care about where the real backing store is--across the network, in the user program's memory, or on a "local" disk device.

One way to think of MacFUSE user-space file systems is indeed as "servers"--in the microkernel sense.

All this said, it _is_ possible to tag a specific mount point as "local" at mount time. If you wish to do so, you can use the `-o local` MacFUSE mount-time option. This has caveats though: the Finder (and the operating system in general) may try to do things differently with local volumes. Some of these different things may not be what you want--for example, you may not want a `.Trashes` directory created on your volume. (It may not even be possible to create that directory in the case of certain file systems.) **Make sure** you read the description of `-o local` on the [OPTIONS](OPTIONS.md) page.

**Q 4.2. I mounted a MacFUSE volume but I don't see a volume icon on the Desktop. Why?**

A: Check if the Finder preference for showing mounted servers on the Desktop is enabled. (Also see **Q 4.1** to understand why we are talking about "servers" here.) On Mac OS X "Leopard", this preference is _not_ enabled by default. You can either check if `Finder->Preferences->General->Connected servers` is checked, or you can use the `defaults` command from a shell:

```
$ defaults read com.apple.finder ShowMountedServersOnDesktop
0
```

If the preference is unchecked, you can check it within the GUI or use `defaults` to set its value to 1:

```
$ defaults write com.apple.finder ShowMountedServersOnDesktop 1
```

Alternatively, you can use the `-o local` MacFUSE mount-time option to tag the volume being mounted as "local", in which case it would show up on the Desktop unless you have disabled "external disks" from showing up on the Desktop.

**Q 4.3. On Mac OS X "Leopard", I mounted a MacFUSE volume but I don't see a volume icon in the Finder's sidebar. I've looked at all relevant Finder preferences, but still nothing. What's happening?**

A: Ah the Finder and its infinite wisdom. On "Leopard", the Finder by default won't show a MacFUSE volume under the "DEVICES" section of the sidebar because the volume isn't "local". The Finder won't show the volume under "SHARED" (even though it's "nonlocal") because it apparently thinks that nonlocal volumes must be of types such as AFP, NFS, SMB, and such. If you really want the volume to show up in the sidebar, you can use the `-o local` MacFUSE mount-time option. Also see **Q 4.1** and **Q 4.2**.

**Q 4.4. I tried to use file system ${whatever} with MacFUSE and I am not happy because of ${whatever else}. What to do?**

A: MacFUSE is like a programming library. Individual MacFUSE file systems (like ntfs-3g, procfs, sshfs, etc.) are programs written using that library. The quality and behavior of individual programs--of any kind--can vary wildly, and may or may not be an issue with the underlying library. If you have poor experience with an individual MacFUSE file system, please determine (or help determine) if it's an issue specific to that file system or if it's a general issue with MacFUSE. File systems are complex beasts. MacFUSE does simplify their implementation enormously, but nontrivial user-space file systems can still be complex, and their performance/behavior can depend upon numerous factors besides MacFUSE itself. When in doubt, feel free to post your questions on the [macfuse](http://groups.google.com/group/macfuse) group.

**Q 4.5. I'm trying to access a MacFUSE volume but I keep getting access/permission denied errors. I'm doing this as root--what's going on? Isn't everything allowed as root?**

A: For several reasons, by default, MacFUSE allows access to a volume _only_ to the user who mounted the volume. All other users, _including_ the superuser is denied access. You can change this behavior if you need to. Refer to the `allow_other` and `allow_root` mount-time options in the [OPTIONS](OPTIONS.md) document.

**Q 4.6. Can I enable Spotlight on a MacFUSE file system?**

A: Yes, but by default, Spotlight processes running as root would not be able to access the volume, so you have to use certain mount-time options. See the answer to the previous question.

**Q 4.7. I tried to run `gdb` on an executable residing on a MacFUSE file system but I got an "operation not permitted" error. What's going on?**

A: `gdb` is a setgid executable on Mac OS X (see `/usr/libexec/*gdb*`). Therefore, under MacFUSE's default mode of operation, it won't have access to MacFUSE volumes. See the previous questions.

**Q 4.8. How should I unmount my MacFUSE file system? I can't find the `fusermount` program anywhere.**

A: Just use the standard `umount` command in Mac OS X. You don't need the Linux-specific `fusermount` with MacFUSE.

**Q 4.9. All my file systems are showing up in the Finder as "MacFUSE Volume n (X)" for some number n and some file system type X. What should I do?**

A: Use the `volname` command-line option. See [OPTIONS](OPTIONS.md) for details.

# 5. Developer Questions #

**Q 5.1. I'm having problem XYZ compiling a FUSE file system in the MacFUSE environment. What to do?**

A: Besides any other file-system-specific compile- or configure-time options, you need to have `-D__FreeBSD__=10` in your `CFLAGS`. Try something like:

```
$ CFLAGS="-D__FreeBSD__=10" ./configure --prefix=/usr/local
```

You may also need `-D_FILE_OFFSET_BITS=64` in `CFLAGS` unless it's already there.

MacFUSE installs a `fuse.pc` file in `/usr/local/lib/pkgconfig/`. This, in conjunction with the `pkg-config` program, will automatically cause `-D__FreeBSD__=10 -D_FILE_OFFSET_BITS=64` to be included in `CFLAGS`. If your particular installation of `pkg-config` (say, through Fink or Mac Ports) doesn't look under `/usr/local/lib/pkgconfig/` by default, you can set the environment variable `PKG_CONFIG_PATH` to `/usr/local/lib/pkgconfig`.