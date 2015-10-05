# MacFUSE Mount-Time Options: A Brief Guide #

This page documents several user-accessible mount-time options in MacFUSE. Please note that:

  * This page assumes MacFUSE version 1.0.0 or higher, although many options listed here will work with older MacFUSE versions.
  * Unless otherwise noted, options should work the same on any Mac OS X version that MacFUSE itself is supported on.
  * MacFUSE has other options besides the ones listed here. I've listed the ones that I feel would be useful to most users.

As I get more time, I may expand this document.

## Options ##

  * `allow_other`

By default, MacFUSE limits volume access to the user that mounted the volume. Nobody, not even the superuser, can access another user's MacFUSE volume. This blanket denial is a first line of defense against misbehaving (whether inadvertently or otherwise) user-space file systems that could, for one, "hang" system programs. If you do trust a file system or are otherwise confident, you can use `allow_other` to turn the blanket denial off. For example, if you wish to use Spotlight on a MacFUSE volume, you'll need to use `allow_other`.

With `allow_other`, the volume will be accessible normally to all users, but usual permission checks will of course apply. Note that `allow_other` is a privileged option in that it can only be used by either the superuser or by a user belonging to the "MacFUSE Admin" group. When MacFUSE loads in the kernel, it sets this group's ID to be that of the `admin` group on Mac OS X. The superuser can view or change this ID through the `sysctl` interface (note that MacFUSE allows you to set any group ID you specify, even those that do not exist):

```
$ sudo sysctl macfuse.tunables.admin_group # get
macfuse.tunables.admin_group: 80
$ sudo sysctl -w macfuse.tunables.admin_group=81 # set
macfuse.tunables.admin_group: 80 -> 81
```

  * `allow_recursion`

By default, MacFUSE does not allow you to mount a MacFUSE volume on a directory that itself resides on a MacFUSE volume. Such recursion can create interesting unmount-time issues in some cases. Unlike most other MacFUSE restrictions, this is a "soft" check that's done only by the mount program. `allow_recursion` disables this restriction.

  * `allow_root`

Please read the description for `allow_other` first. `allow_root` is similar, except that the set of "other" users contains only the superuser. You need to be a member of the MacFUSE Admin group to use `allow_root`.

  * `auto_cache`

By default, if MacFUSE detects a change in a file's size during `getattr()`, it will purge that file's buffer cache. When `auto_cache` is enabled,  MacFUSE will additionally detect modification time changes during `getattr()` and `open()` and will automatically purge the buffer cache and/or attributes of the file if necessary. It will also generate the relevant `kqueue` messages. All this is subject to the attribute timeout. That is, up to one purge can occur per attribute timeout window. As long as the user-space file system's `getattr()` callback returns up-to-date size and modification time information, this should work as intended. For user-space file systems that wish the kernel to keep up with "remote" changes, this should obviate the need for explicit purging. `auto_cache` is not enabled by default: it's opt-in.

  * `auto_xattr`

By default, MacFUSE provides a flexible and adaptive mechanism to handle extended attributes (including things such as Finder Info, Resource Forks, and ACLs). It will initially forward extended attributes calls up to the user-space file system. If the latter does not implement extended attribute functions, MacFUSE will remember this and will not forward subsequent calls. It will store extended attributes as Apple Double (`._`) files. If the user-space file system does implement extended attribute functions, it can choose to handle all or only some extended attributes. If there are certain extended attributes that the user-space file system wants MacFUSE (the kernel) to handle through `._` files,  it should return `ENOTSUP` for such attributes. The `auto_xattr` option tells MacFUSE to not bother with sending any extended attributes calls up to user-space, regardless of whether the user-space file system implements the relevant functions or not. With `auto_xattr`, the kernel will /always/ use `._` files.

  * `daemon_timeout=N`, where `N` specifies the timeout in seconds

When the kernel sends calls to a user-space file system, the latter must reply within some reasonably short time limit, otherwise those calls, and in turn the application that made those calls, will "hang". User programs cannot be completely trusted as they could inadvertently or maliciously fail to respond forever. A typical example is when the file system has a bug that has caused it to crash. In this case, the file system will never respond. Another typical example is that of a network operation performed by the file system taking too long. The kernel can't tell what is really going on, but it does have to guard against hangs. Such hangs may have limited harmful impact on other operating systems, but they are typically deadly on Mac OS X: many parts of the system, including the Finder, can throw a major fit. Often, end users will fail to see a difference between trivially recoverable hangs and unrecoverable system lockup. Therefore, such defensive measures are imperative on Mac OS X.

Still, if you want more control, you can tweak things on a per-mount basis. The `daemon_timeout` option lets you specify the aforementioned time limit in seconds. The default timeout is set by the `FUSE_DEFAULT_DAEMON_TIMEOUT` define in `fusefs/common/fuse_param.h` in the MacFUSE kernel source tree. A good question is regarding what happens when there /is/ a timeout. There are several possibilities. Currently, on Mac OS X "Tiger" (default timeout 30 seconds), you will see an alert dialog asking you to choose between force ejecting the volume immediately, continuing to wait until the next timeout, or continue to wait forever (the latter disables the timeout altogether for the given mount instance). On Mac OS X "Leopard" (default timeout 60 seconds), there is no alert dialog--the volume will be automatically ejected (data in any open files may be lost). This is because the kernel APIs to show the alert dialog are deprecated in Leopard. There may be more fine-grained and more fault-tolerant behavior in future versions of MacFUSE.

  * `debug`

The `debug` (or simply `-d`) option causes the user-space file system daemon to print debugging messages for requests and response. The daemon will also run in foregrounded mode with this option.

  * `default_permissions`
  * `defer_permissions`

As with extended attributes, MacFUSE provides a flexible and adaptive mechanism for dealing with permission checks. By default, MacFUSE will involve the user-space file system in permission checks by sending it `access` messages (see `access(2)`). If the file system doesn't implement the `access` function, MacFUSE will perform permission checks /entirely/ itself based on the attributes of file system objects. The `default_permissions` option also enables this behavior, wherein the kernel will attempt to do the "right thing" based on what it can "see" (the permission bits as reported by the file system). The file system may not always do the right thing itself though--for example, if the file system retrieves file information from another computer and reports user/group IDs /as is/, the kernel will not see any alien IDs as belonging to the user that mounted the volume. (This can happen if user ID translation is not enabled or sometimes doesn't work with sshfs.) The `defer_permissions` (formerly `defer_auth`) option is useful in such cases. It causes MacFUSE to assume that all accesses are allowed--it will forward all operations to the file system, and it is up to somebody else to eventually allow or deny the operations. In the case of sshfs, it would be the SFTP server eventually making the decision about what to allow or disallow.

  * `direct_io`

Some file systems may not know the sizes of files that they provide. This could be because a file's content is being streamed so it's difficult or impossible to know the "size" of the file. The content could be dynamically changing so it may not make sense to advertise a size at `getattr` time only to find that the size has changed at `read` or `write` time. `procfs` is a good example of a MacFUSE file system with such needs. What these file systems would like is to be able to allow reads and writes without the file size mattering. This isn't normally possible in the normal I/O paths in the kernel. In particular, short reads from a user-space file system will be zero filled by MacFUSE. The `direct_io` option causes MacFUSE to use an alternative "direct" I/O path between the kernel and the user-space file system. This path makes the file size irrelevant--a read will go on until the file system keeps returning data. There is also no automatic zero filling. In particular, as an implementation side effect, the I/O path bypasses the unified buffer cache altogether.

`direct_io` is a rather abnormal mode of operation from Mac OS X's standpoint. Unless your file system requires this mode, I wouldn't recommend using this option.

  * `extended_security`

MacFUSE supports Mac OS X access control lists (ACLs). The `extended_security` option will enable it for a file system. To properly support ACLs, a user-space file system must correctly handle the `com.apple.system.Security` extended attribute. It /is/ possible to get by with storing ACLs in Apple Double (`._`) files, although that wouldn't be a very secure implementation of ACLs because such files can be manipulated directly. Still, it would work for experimental purposes.

  * `-f`

The `-f` option causes a user-space file system to run in foregrounded mode.

  * `fsid=N`, where `N` is a 24-bit integer (`0` and `0xFFFFFF` **not** valid)

Normally, MacFUSE will use a newly generated file system ID for every mount. This is fine except in a case where you /do/ want the volume, when mounted again, to have a persistent file system ID (think aliases, for example). Since MacFUSE volumes do not have "real" devices backing them, this isn't possible without extra help. The `fsid` option is that extra help: you use it to specify a 24-bit number that will be used to generate the file system ID.

  * `fsname=NAME`, where `NAME` is a string

This option can be used to specify the "file system name", analogous to the device in the case of a disk-backed "real" file system. For example, the fsname in the following mount information is `sshfs#singh@127.0.0.1`.

```
$ mount
...
sshfs#singh@127.0.0.1:/tmp/dir on /private/tmp/ssh (fusefs, nodev, nosuid, synchronous, mounted by singh)
...
```

Note that a file system daemon can choose to specify its own fsname and can override (by strategically inserting its argument) a user-provided fsname.

  * `fssubtype=N`, where `N` is an integer

You can use this option to specify the file system subtype identifier for MacFUSE volume being mounted. The Finder uses this subtype to retrieve the string description (if any) for the file system. The description is what you see next to the `Format:` label in window that you get when you perform a "Get Info" Finder operation on the volume. This only works for subtype-string pairs that are already present in the MacFUSE file system bundle (`fusefs.fs`) that's installed as part of MacFUSE. You can add your own additional mappings to the bundle, or send me an email if you think your file system is useful enough to have an "official" entry.

  * `fstypename=NAME`, where `NAME` is a string

You can use this option to specify the file system's /type/ name. `NAME` can be at most 7 character (and if specified, must be at least one character). This would cause the in-kernel file system type to be `fusefs_NAME`. That is, the `fusefs_` prefix will be implied and automatically added by the kernel. Thereafter, a user-space file system that uses this option must name its on-disk file system bundle `fusefs_NAME.fs`. The Finder and other programs/libraries that look into file system bundles will now look inside this bundle (instead of the usual `fusefs.fs`) for things such as subtypes and description strings. The `fssubtype=N` option will continue to exist and work. A file system that does not need or use `fssubtypename=NAME` will work as in the past. Please note that this option is doing something the side effects of which cannot be entirely predicted because arbitrary programs can depend on the /true/ file system type--therefore, the use of this option is subject to a general "this might break things you don't know about" warning.

  * `iosize=N`, where N is the I/O size in bytes

You can use this option to specify the I/O size MacFUSE should use while accessing the hypothetical storage device corresponding to a MacFUSE volume. The minimum possible I/O size is 512 bytes, whereas the largest is 1MB. The size must also be a power of 2.

  * `jail_symlinks`

This option can be used to "jail" symbolic links within the MacFUSE volume itself. If an absolute symbolic link is encountered, MacFUSE will prefix the mount path to it.

  * `kill_on_unmount`

On Mac OS X 10.4.x, a user-space file system may not exit upon unmount. This is because it may be blocked in one or more operations whose cancellation is not supported by the operating system. The `kill_on_unmount` option can be used as a workaround to make the kernel send a `SIGKILL` to the file system daemon /after/ the `unmount` call has been processed. Although I have not tested exhaustively, this should not be necessary on Mac OS X 10.5.x.

  * `local`

This option marks the volume being mounted as "local". By default, MacFUSE volumes are marked as "nonlocal", which technically isn't necessarily the same as a "server" or "network" volume, but is treated as such by the Finder in some cases. For example, the Finder may not show "connected servers" on the Desktop or in the sidebar in some cases. If you use this option, you can get around this "limitation". However, **wait**! Don't be too tempted and think `local` is a magic pill that will solve all your problems. In fact, it may mess things up more than you realize. The operating system can be more aggressive in dealing with "local" volumes (a `.Trashes.` directory will be created, for one). You could run into mysterious problems with Disk Arbitration and other system components. I don't know (and possibly can't know--Mac OS X isn't all open source!) the side effects of using this option. Therefore, treat this as experimental and use with caution. Moreover, please do not file bug reports that involve this option--reproduce your issue **without** this option and then file a bug report.

  * `modules=M1[:M2...]`, where Mi is the name of a module to pushed onto the file system stack

The MacFUSE user-space library supports /stackable/ modules--entire file systems or file system shims that can be stacked atop other file systems or shims. For example, filename character set conversion can be handled through such a module. Custom volume icon support in MacFUSE is handled through the `volicon` module. In fact, the `volicon=PATH` option is a shortcut for `modules=volicon,iconpath=PATH` option combination. Note that each module can have one or more of its own arguments (`volicon` has one option: `iconpath`).

  * `negative_vncache`

This option enables negative vnode name caching in the kernel. That is, when a file system object name does not exist, MacFUSE will remember this upon the first lookup of that name. Thereafter, MacFUSE will not call the user-space file system upon subsequent lookups for that name--until an object of that name does get created. The negative cache is LRU managed by the kernel. This is an optimization. If you have a case where file system objects can appear "outside of MacFUSE's knowledge" (say, on a remote server in the case of a MacFUSE-based network file system), then you should NOT enable this option.

  * `noappledouble`

This option makes MacFUSE deny all types of access to Apple Double (`._`) files and `.DS_Store` files. Any existing files will become apparently non-existent. New files that match the criteria will be disallowed from being created.

  * `noapplexattr`

This option makes MacFUSE deny all types of access to extended attributes that begin with the "com.apple." prefix. On Mac OS X 10.5.x, this is the preferred option if you wish to disallow entities such as resource forks and Finder information.

  * `nobrowse`

This marks a volume as non-browsable in that it indicates that the file system is not an appropriate path to user data. The Finder wouldn't automatically browse into such a volume.

  * `nolocalcaches`

The `nolocalcaches` option disables (in the kernel) the unified buffer cache (UBC), vnode name caching, attribute caching, and readaheads. This means the kernel will have to call the user-space file system upon every operation. The purported "benefit" is that the kernel would be able to "pick up" file changes that occur unbeknownst to the kernel. For example, in the case of SSHFS, if a file was changed on the server, MacFUSE has no way of knowing about it--with `nolocalcaches`, MacFUSE will have no cached information and will be forced to talk to the `sshfs` program. The SSHFS program has its own cache, which you can also disable using the `cache=no` option. If all caching is thus disabled, all operations would end up going to go to the SFTP server, resulting in an /apparently/ up-to-date view of the remote file system. There are major caveats, however. `nolocalcaches` makes the operating system work in an abnormal mode, so the relevant code paths may not have been well tested. The option also creates significant overhead--file operations could end up being /much/ slower. Besides, since SFTP provides no synchronization or locking, there are no consistency or ordering guarantees if multiple clients write to the same file concurrently. My strong suggestion is to /realize/ that SFTP is merely a utility to access remote data, and not a means of distributed file sharing in the same vein as NFS, AFS, Coda, and so on. Of course, you are welcome to create a true distributed file system based on MacFUSE.

  * `noubc`

This turns off the unified buffer cache (UBC) for the entire MacFUSE volume.

  * `novncache`

This turns off vnode name caching in the kernel.

  * `ping_diskarb`
  * `noping_diskarb`

Beginning with version 1.0.0, MacFUSE always performs synchronous (foreground) mounting, so there's no race between the mounting process finishing and the Finder discovering a MacFUSE volume. On Mac OS X 10.4.x, MacFUSE still needs to tell the Finder that a MacFUSE volume has appeared. The `ping_diskarb` option tells MacFUSE to do that (`ping_diskarb` is enabled by default). The `noping_diskarb` turns the default behavior off. On Mac OS X 10.5.x, both these options are no-ops (they are silently ignored).

  * `quiet`

The MacFUSE user-space library ensures at runtime that it is talking to a compatible version of the MacFUSE kernel component. If there is a mismatch, MacFUSE will show an alert dialog and will also send a notification to the Distributed Notification Center. The `quiet` option will suppress any such alert dialogs.

  * `rdonly`

The `rdonly` (or simply, `-r`) option can be used to mount a MacFUSE file system in read-only mode.

  * `-s`

By default, the MacFUSE user-space library runs a file system in multithreaded mode. You can use the `-s` option to make a user-space file system run in single-threaded mode.

  * `volicon=PATH`, where `PATH` is path to an icon (`.icns`) file

You can use the `volicon` option to specify an icon file that would be used as the Desktop icon for the mounted MacFUSE volume. MacFUSE will then do the needful (simulating Finder information for the root folder, simulating a `/.VolumeIcon.icns` file, etc.) to make the custom volume icon work. Note that MacFUSE uses library-level stacking for this--it generates a shim file system atop the regular user-space file system, with the shim's only purpose being to simulate what's necessary for the icon to work. **NB**: the `volicon` argument WILL NOT WORK correctly if you use any other library stack module along with the volicon module. To make custom volume icons work alongside other stack modules, you have to specify the modules and their arguments using the "long form" (`volicon=PATH` is a convenience shortcut). For details, see the note at the end of the [CUSTOM\_VOLUME\_ICON](http://code.google.com/p/macfuse/wiki/CUSTOM_VOLUME_ICON) wiki page.

  * `volname=NAME`, where `NAME` is a string

You can use the `volname` option to specify a name for the MacFUSE volume being mounted. This is the name that would show up on the Desktop. In the absence of this option, MacFUSE will automatically generate a name that would incorporate the MacFUSE device index and the user-space file system being used. For example, an SSHFS mount might have an automatically assigned name "MacFUSE Volume 0 (sshfs)".

