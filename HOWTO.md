# Introduction #

This document briefly describes the steps you need to take to get a MacFUSE development environment set up starting from scratch.

**NB** If you don't wish to compile everything yourself (you shouldn't have to, unless you want to do development of MacFUSE itself), you have the option of using a precompiled version available under the Downloads tab of this project's Google Code page (http://code.google.com/p/macfuse/downloads/list). Please read the [QUICKER\_START\_GUIDE](QUICKER_START_GUIDE.md) document to get started with MacFUSE the fastest way.

# Setting Up MacFUSE Core #

The MacFUSE subversion repository is located at:

http://code.google.com/p/macfuse/source

You can anonymously check out the latest project source code as follows:

`svn checkout http://macfuse.googlecode.com/svn/trunk/ macfuse`

If you have developer access to the repository, you can do the following:

`svn checkout https://macfuse.googlecode.com/svn/trunk macfuse --username <user>`

**NB**: The current MacFUSE source tree has only been compiled with Xcode 3.x (Leopard) and Xcode 2.4.x/2.5.x (Tiger). To avoid mysterious compilation and runtime failures, please upgrade to an appropriate Xcode version, which should be a free download from Apple.

## 1. Compile MacFUSE Core ##

Compiling MacFUSE Core, the software atop which all MacFUSE file systems run, is no longer a complex process. Once you have the source tree checked out, you just need to run a single command from the appropriate directory, for example:

```
$ /path/to/macfuse/source/core/macfuse_buildtool.sh -t smalldist
...
```

This will compile the kernel extension, the user-space library (`libfuse`), the mount utility (`mount_fusefs`), the load utility (`load_fusefs`), and will create the file system bundle (`fusefs.fs`). If everything goes fine, a ready-to-install Universal package can be found in `/tmp/macfuse-core-<OS version>-<MacFUSE version>`. For example, if you compile MacFUSE version 2.0 on Mac OS X 10.5, the package will be generated at:

```
/tmp/macfuse-core-10.5-2.0.1/MacFUSE Core.pkg
```

You can install this package by double clicking on it or through the command line:

```
$ sudo installer -pkg "MacFUSE Core.pkg" -target /
...
```

# Setting Up Other MacFUSE-Related Software #

If you wish to write a new MacFUSE-based file system on Mac OS X, you should be good to go. If you wish to compile `sshfs` or other FUSE-based file systems originally from the Linux world, you'll find that several of them depend upon `glib`. Therefore, it might be useful to just install a few other packages that are typically available on Linux but not on Mac OS X by default.

**NB**: The `sshfs` binary package available from this project's website contains an `sshfs-static` binary that is statically linked against `glib` (but dynamically linked against `libfuse`). If you wish to quickly try out `sshfs`, you can just install the MacFUSE Core package and use `sshfs-static` without having to compile anything.

**NB**: The version numbers in examples below aren't necessarily the ones you should be using--in general, the latest version should be good.

## 1. Install pkg-config ##

`pkg-config` is an open source helper tool used when compiling applications and libraries. You don't **need** pkg-config to run MacFUSE, but if you're going to be compiling existing FUSE file systems (those that were written for Linux), having `pkg-config` will be helpful in many cases. To download its source and learn more about it, go here:

http://pkgconfig.freedesktop.org/wiki/

To compile and install `pkg-config` as a set of Universal binaries, do the following:

```
$ tar -xzvf pkg-config-0.23.tar.gz
...
$ cd pkg-config-0.23
$ /path/to/macfuse/source/core/macfuse_buildtool.sh -t swconfigure
...
$ make
...
$ sudo make install
...
```

This installs `pkg-config` under the `/usr/local/` hierarchy.

## 2. Install glib ##

Some FUSE-based file systems depend on `glib`, which is not available by default on Mac OS X.  You don't **need** `glib` to run MacFUSE, but if you're going to be compiling existing FUSE file systems (those that were written for Linux), having `glib` will be required in some cases. To compile `glib` without pain, you should also have the GNU `gettext` package installed.

You can download `gettext` from:

[ftp://ftp.gnu.org/pub/gnu/gettext/](ftp://ftp.gnu.org/pub/gnu/gettext/)

You can compile gettext (Universally) as follows:

```
$ tar -xzvf gettext-0.17.tar.gz
...
$ cd gettext-0.17
$ /path/to/macfuse/source/core/macfuse_buildtool.sh -t swconfigure
...
$ make
...
$ sudo make install
...
```

You can download `glib` from:

[ftp://ftp.gtk.org/pub/glib/](ftp://ftp.gtk.org/pub/glib/)

You can compile `glib` (Universally) as follows:

```
$ tar -xzvf glib-2.19.0.tar.gz
...
$ cd glib-2.19.0
$ /path/to/macfuse/source/core/macfuse_buildtool.sh -t swconfigure
...
```

Only if you are on a **PowerPC** Macintosh, comment out one line in `config.h`: `// #define G_ATOMIC_POWERPC 1`

Additionally (again, only on **PowerPC**), open the file `libtool` and locate the lines (there should be two) that begin with `sys_lib_search_path_spec=`... replace the entire line (in both cases) with the following (choose between the 10.4 one and the 10.5 one depending on whether you are on "Tiger" or "Leopard", respectively):

```
sys_lib_search_path_spec=" /Developer/SDKs/MacOSX10.4u.sdk/usr/lib/ /usr/local/lib/"
```

```
sys_lib_search_path_spec=" /Developer/SDKs/MacOSX10.5.sdk/usr/lib/ /usr/local/lib/"
```

Now, back in the Terminal:
```
$ make
...
$ sudo make install
...
```

## 3. Install sshfs ##

`sshfs` source can be downloaded from:

http://fuse.sourceforge.net/sshfs.html

You'll need to patch the source using an appropriate Mac OS X patch file available in the `filesystems/sshfs/` subdirectory in the MacFUSE subversion repository.

```
$ tar -xzvf sshfs-fuse-2.2.tar.gz # unpack sshfs source
$ cd sshfs-fuse-2.2
$ patch -p1 < /path/to/sshfs-fuse-2.2-macosx.patch # apply patch with a matching version
...
$ /path/to/macfuse/source/core/macfuse_buildtool.sh -t swconfigure
...
$ make
...
$ sudo make install
```

This will install the `sshfs` command-line program under `/usr/local/bin/`.

**NB**: You can also get a precompiled `sshfs` binary for your platform from the [filesystems/sshfs/binary/](http://code.google.com/p/macfuse/source/browse/#svn/trunk/filesystems/sshfs/binary) directory in the source tree. These binaries, called `sshfs-static-leopard` (etc.), are statically linked against `glib` but dynamically linked against `libfuse`. If you wish to quickly try out `sshfs`, you can just install the MacFUSE Core package and use the precompiled statically linked binaries without having to compile anything.

# Compiling Other FUSE File Systems Written for Linux #

Many existing FUSE file systems are readily usable on Mac OS X with MacFUSE. Some might have additional dependencies (typically open source libraries that might not be already installed on your Mac OS X machine), which you have to satisfy before compiling and using the file system in question.

When running the configure script for a file system, you **need** to have `-D__FreeBSD__=10` in `CFLAGS`. This is critical!

Often, using the `macfuse_buildtool.sh` script to configure the software for compilation will work. As in the case of `sshfs` above, go to the top-level compilation source directory (the one that contains a `configure` script) of the software in question and run `macfuse_buildtool.sh`.

```
$ cd /path/to/source/you/want/to/compile/
$ /path/to/macfuse/source/core/macfuse_buildtool.sh -t swconfigure
...
$ make
...
```

Of course, if this doesn't work, you'll have to tweak things in minor or major ways. There's no panacea.