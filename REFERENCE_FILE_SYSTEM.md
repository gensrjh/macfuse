# Using the MacFUSE Reference File System #

File systems are intricate and complex beings. In modern operating systems, file systems are intimately tied to numerous other subsystems, and in particular, to the virtual memory subsystem. Information flows in and out of file systems through myriad channels residing in both kernel and user spaces.

Kernel programming can be rather challenging in itself, but programming in-kernel file systems can be extremely, insanely complex and frustrating. MacFUSE simplifies this task by orders of magnitude on Mac OS X, allowing developers to create file systems using a simple _user-mode_ API along with other familiar and rich programming libraries available to them. Still, any reasonably complete user-mode file system, especially one that is writable, will still be complex enough to require very careful design, implementation, and testing on part of the developer. Subtleties and caveats abound in the world of file systems in general, but on Mac OS X, subtleties have their own subtleties. Mac OS X has so many software layers and APIs that you can never be too sure that your file system will work as expected by the end user under all circumstances. Therefore, it is paramount to test your user-space file system the best you can.

The MacFUSE source tree contains a _reference_ file system that can be an important tool in helping you understand MacFUSE behavior and use that knowledge to test your file system better. The reference file system,  called [loopback](http://code.google.com/p/macfuse/source/browse/trunk/filesystems/loopback/loopback.c), can be found in the `filesystems/` subdirectory in the source tree. The file system is trivial to compile. We will assume that you have the MacFUSE source tree checked out under `/work/macfuse/`.

```
$ cd /work/macfuse/filesystems/loopback
$ make
...
Compiled. The following is a typical way to run the loopback file system. In
this example, /tmp/dir is an existing directory whose contents will become
available in the existing mount point /Volumes/loop:

sudo ./loopback /Volumes/loop -omodules=threadid:subdir,subdir=/tmp/dir -oallow_other,native_xattr,volname=LoopbackFS
```

As the compilation output mentions, you can use the loopback file system to perform a, well, "loopback" mount of an existing local directory through MacFUSE. It is a low-footprint file system that will reroute file system activity to a given directory's contents through MacFUSE. You can take an existing directory, say `/d` and mount it on top of another directory, say `/Volumes/loopback`. Then, as you access and manipulate files and folders in `/Volumes/loopback`, your accesses will go from the kernel (MacFUSE) to the user-mode `loopback` program. `loopback` serves file system requests by simply forwarding them to the appropriate C library calls (which in turn will forward to system calls). For example, its implementation of `read` and `write` simply forward to the `read` and `write` system calls, respectively. Since `loopback` does so little itself, it is valuable as a tool that lets you focus on MacFUSE behavior rather than the specifics of the user-mode file system. If you are having difficulty in getting some behavior right in your file system, try the same tests with `loopback` and see what it does. You might be able to determine what you are doing (or not doing) that's causing the issue. You might also find a bug in `loopback` and/or MacFUSE.

Note that `loopback` is also a great way to isolate MacFUSE's overhead. Often, people expect user-space file systems to be "very slow". If you compare file system activity benchmarks for I/O to a local HFS+ directory vs I/O to a loopback mounted directory, you _could_ be surprised at how well MacFUSE fares in terms of being low-overhead.

To run `loopback` in high-fidelity mode (that is, as close to a "real" Mac OS X file system as possible), make sure that your backing directory (the one that you remount through `loopback`) resides on an HFS+ volume. You can create a directory called `/d` for this purpose and then mount it as follows:

```
$ mkdir /d # This is the backing store directory; the volume's data will actually reside in here
$ mkdir /Volumes/loopback # This is where we'll mount
$ sudo ./loopback /Volumes/loopback -omodules=threadid:subdir -osubdir=/d -oallow_other,native_xattr,volname=LoopbackFS
```

The `threadid` stacking module in conjunction with the `allow_other` mount-time option (and the fact that you run `loopback` as root) lets you run the loopback file system in multi-user mode. The `threadid` module automatically sets the user/group ID of each thread that calls into your file system to that of the process that made the original request.

The `subdir` stacking module in conjunction with the `subdir=/d` module option lets you limit `loopback` to a single directory. Normally, `loopback` would actually remount the root file system. It's nicer and cleaner to contain experimentation to one directory. Note that if you conduct file system tests that involve long path names, the `loopback` program will have to work with path names that are longer on the back-end side: for example, if a call comes in to create a file `/foo`, `loopback` will attempt to create `/d/foo` on the underlying file system. Thus, the back-end will hit the path name length limit earlier than the front-end.

The `native_xattr` mount-time option tells the kernel that the user-space file system supports _true_ native extended attributes. Unless you really, **really** understand what this means and entails, you shouldn't enable this option. In any case, this is an undocumented, unsupported option.

Note that the MacFUSE source tree also contains an Objective-C based file system called [LoopbackFS](http://code.google.com/p/macfuse/source/browse/#svn/trunk/filesystems-objc/LoopbackFS). Although similar in nature, LoopbackFS sits higher in the MacFUSE software stack as compared to the C-based `loopback`. If you are writing a MacFUSE file system in Objective-C, you _will_ greatly benefit from studying LoopbackFS. In fact, it will likely serve as a great starting point. However, as a _reference_ file system that represents MacFUSE's true behavior, `loopback` is a more appropriate choice.