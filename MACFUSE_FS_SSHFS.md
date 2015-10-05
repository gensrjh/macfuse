# The SSH File System for MacFUSE #

**NB**: You can get a precompiled `sshfs` binary for your platform from the MacFUSE subversion tree (these **require** MacFUSE 2.0 or above.)

[sshfs-static-leopard.gz](http://osxbook.com/download/sshfs/sshfs-static-leopard.gz)

[sshfs-static-tiger.gz](http://osxbook.com/download/sshfs/sshfs-static-tiger.gz)

Alternatively, here's how you can acquire these binaries using the `svn` command. (The command sequence shown will leave the binaries in a folder called `sshfs-binaries` on your Desktop.)

```
$ cd ~/Desktop
$ svn co http://macfuse.googlecode.com/svn/trunk/filesystems/sshfs/binary sshfs-binaries
```

These binaries, called `sshfs-static-leopard` (etc.), are statically linked against `glib` but dynamically linked against `libfuse`. If you wish to quickly try out `sshfs`, you can just install the MacFUSE Core package and use the precompiled statically linked binaries without having to compile anything. You can rename them to `sshfs` if you so prefer.

**NB: If you are running an older version of sshfs, please be sure to upgrade to the latest version available. The latest source patch for `sshfs` is available in the [filesystems/sshfs/](http://code.google.com/p/macfuse/source/browse/#svn/trunk/filesystems/sshfs/) directory of the MacFUSE source tree. Newer versions might have important and useful updates.**

## Using the Command Line `sshfs` Program ##

To use the command line `sshfs` program, you can do something like:

```
# Mounting the SSH file system
$ mkdir /some/mount/point # or use one that already exists
$ sshfs user@host:/some/directory /some/mount/point -oauto_cache,reconnect,volname=<volname>
```

`<volname>` should be a string you wish to use as the name of the newly mounted volume. If everything went fine, you should see the volume on your Desktop and in the Finder (unless you have disabled remote volumes from showing up in these places).

Once you are done using sshfs, you can unmount the volume either using the eject icon in the Finder or other common user-interface means of ejecting. Alternatively, you can do the following from the command line:

```
$ umount /some/mount/point
```

## Changelog ##

### sshfs(MacFUSE) 2.2.0 (October 22, 2008) ###

  * Numerous bugfixes. See `ChangeLog` file in `sshfs` source for details.
  * The GUI wrapper to `sshfs` (the `sshfs.app` application) is no longer available in precompiled form. It source is still available, of course.

### sshfs(MacFUSE) 0.3.0 (May 7, 2007) ###

  * No changes to the user interface, which remains, and will remain, an **unsupported demo GUI wrapper**.
  * Fix for an sshfs crash that could occur when doing heavy I/O to a single file (such as while creating a large tarball or running `mkfile`).
  * Fix for an sshfs crash that could occur when doing heavy directory I/O in a heavily populated directory hierarchy (such as running multiple `find` commands on an sshfs volume, possibly in conjunction with some file reads).
  * Increased the timeout for the "daemon not responding" alert panel to 20 seconds from 10 seconds. On a slow link, 10 seconds wasn't enough with some heavily populated directory hierarchies. Of course, even 20 seconds may not be enough, but you can always use the `daemon_timeout` mount-time option.
  * User and group ID mapping is now **turned on** by default. This means that user/group IDs of the user logging in to the remote (SSH) machine will be automatically translated to the user/group IDs of the local user.

### sshfs(MacFUSE) 0.2.0 (April 19, 2007) ###

  * Alert panel shown upon daemon timeout, giving the user the option to eject the sshfs volume.
  * Fixes for some "hangs" that users could experience under certain circumstances (such as saving a document in Microsoft Word).
  * `TCP_NODELAY` supported (and enabled by default) through the `sshnodelay.so` dynamic library that's preloaded into sshfs. This library is looked for in the same location as `sshfs-static`, in `/usr/local/lib/`, and in the current directory (in that order). If `sshnodelay.so` is not found, `TCP_NODELAY` is not enabled.
  * Fix for issue with pathname caching at the sshfs user daemon level. This could cause strange behavior after certain sequences of file system operations.
  * `ping_diskarb` is enabled by default in MacFUSE 0.2.5. If you don't want this option for some reason, you can turn it off through the `noping_diskarb` option.
  * The sshfs option `workaround=rename` is now enabled by default.
  * If the `kill_on_unmount` option (new in MacFUSE 0.2.5) is specified, the kernel will explicitly try to kill the sshfs daemon after the volume has been unmounted. This is to avoid the issue of the daemon "hanging around" even after a successful unmount.
  * Upon launching the application from its containing disk image, the user is given the option of copying the application to the Applications folder.

### sshfs(MacFUSE) 0.1.0 (January 2007) ###

  * Initial binary release.