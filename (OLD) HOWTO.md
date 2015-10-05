# Introduction #

This document briefly describes the steps you need to take to get a MacFUSE development environment set up starting from scratch (that is, compiling everything yourself).

**NB** If you don't wish to compile everything yourself (and you **really shouldn't** any more, unless you want to do development of MacFUSE itself), you have the option of using a precompiled version available under the Downloads tab of this project's Google Code page (http://code.google.com/p/macfuse/downloads/list). Please read the [QUICKER\_START\_GUIDE](QUICKER_START_GUIDE.md) document to get started with MacFUSE the fastest way.

# Details #

The MacFUSE subversion repository is located at:

http://code.google.com/p/macfuse/

You can anonymously check out the latest project source code as follows:

`svn checkout http://macfuse.googlecode.com/svn/trunk/ macfuse`

If you have developer access to the repository, you can do the following:

`svn checkout https://macfuse.googlecode.com/svn/trunk macfuse --username <user>`

The following are some of the key components of the repository:

```
    filesystems/
            |_ sshfs/
                   |_ sshfs-fuse-<version>-macosx.patch  # Mac OS X patches for sshfs
    fusefs/                                                                   # Mac OS X FUSE kernel extension
    libfuse/
            |_ fuse-<version>-macosx.patch                   # Mac OS X patches for libfuse
    util/                                                                       # Miscellaneous utilities and scripts
```

## 0. Upgrade to Xcode 2.4 ##

The current MacFUSE source tree has only been compiled with Xcode 2.4.x. In fact, it may not even compile with older Xcode versions. To avoid mysterious compilation and runtime failures, please upgrade to Xcode 2.4, which is a free download from Apple.

## 1. Install the Kernel Extension and Associated Components ##

The core and the most critical component of MacFUSE is the kernel extension (`fusefs.kext`). The `fusefs/` subdirectory in the repository contains the source for this kext. Moreover, it contains the sources for the `mount_fusefs` and `load_fusefs` command-line programs, as well as the `fusefs.fs` bundle. Here's what you do with these things:

**NB** By default, all compilations are Universal.

### Copy the fusefs.fs bundle to its destination ###

```
$ xcodebuild -target fusefs.fs -configuration Release
$ sudo cp -pR build/Release/fusefs.fs /System/Library/Filesystems/
$ sudo chown -R root:wheel /System/Library/Filesystems/fusefs.fs
```

### Compile fusefs.kext ###

```
$ cd fusefs # Go to the kernel extension source directory
$ xcodebuild -target fusefs -configuration Release
```

This will build the kext as `build/Release/fusefs.kext`. The kext should go in `/System/Library/Filesystems/fusefs.fs/Support/` and must have the appropriate permissions.

```
$ sudo mkdir -p /System/Library/Filesystems/fusefs.fs/Support
$ sudo cp -pR build/Release/fusefs.kext /System/Library/Filesystems/fusefs.fs/Support
$ sudo chown -R root:wheel /System/Library/Filesystems/fusefs.fs/Support/fusefs.kext
```

### Compile load\_fusefs ###

```
$ xcodebuild -target load_fusefs -configuration Release
```

`load_fusefs` is a small helper program that the MacFUSE user library calls to load `fusefs.kext` if the latter isn't already loaded. Note that `load_fusefs` needs to be installed **setuid root**.

```
$ sudo cp build/Release/load_fusefs /System/Library/Filesystems/fusefs.fs/Support
$ sudo chown root:wheel /System/Library/Filesystems/fusefs.fs/Support/load_fusefs
$ sudo chmod u+s /System/Library/Filesystems/fusefs.fs/Support/load_fusefs
```

### Compile mount\_fusefs ###

```
$ xcodebuild -target mount_fusefs -configuration Release
```

`mount_fusefs` is called by the MacFUSE user library to perform the `mount()` system call. Note that this doesn't need to be installed setuid root.

```
$ sudo cp build/Release/mount_fusefs /System/Library/Filesystems/fusefs.fs/Support/
$ sudo chown root:wheel /System/Library/Filesystems/fusefs.fs/Support/mount_fusefs
```

**NB**: The `sshfs` binary package available from this project's website contains an `sshfs-static` binary that is statically linked against `glib` (but dynamically linked against `libfuse`). If you wish to quickly try out `sshfs`, you can just install the MacFUSE Core package and use `sshfs-static` without having to compile anything.

## 2. Install pkg-config ##

`pkg-config` is an open source helper tool used when compiling applications and libraries. You don't **need** pkg-config to run MacFUSE, but if you're going to be compiling existing FUSE file systems (those that were written for Linux), having `pkg-config` will be helpful in many cases. To download its source and learn more about it, go here:

http://pkgconfig.freedesktop.org/wiki/

To compile and install `pkg-config` as a set of Universal binaries, do the following:

```
$ tar -xzvf pkg-config-<version>.gz
$ cd pkg-config-<version>/
$ CFLAGS="-O -g -arch i386 -arch ppc -isysroot /Developer/SDKs/MacOSX10.4u.sdk" LDFLAGS="-arch i386 -arch ppc" ./configure --prefix=/usr/local --disable-dependency-tracking
$ make
$ sudo make install
```

This installs `pkg-config` under the `/usr/local/` hierarchy.

## 3. Install the MacFUSE User-Space Library ##

You need the MacFUSE user-space library to develop your own FUSE-based file systems and also to run precompiled file systems that are dynamically linked with the library.

You can download the library source (the original "fuse" package) from:

http://fuse.sourceforge.net

Suppose you downloaded `fuse-2.6.5.tar.gz` (`2.6.5` is used here just as an example--you should download the most recent version for which the MacFUSE source repository has a patch). You'll need to patch this source using an appropriate patch file available in the `libfuse/` subdirectory in the MacFUSE subversion repository. Be sure to use the `CFLAGS` and `LDFLAGS` specified below! For your convenience, the file `README.MacFUSE` in the library source tree contains the command-line you should use.

```
$ tar -xzvf fuse-2.6.5.tar.gz
$ cd fuse-2.6.5
$ patch -p1 < /path/to/fuse-2.6.5-macosx.patch
...
$ CFLAGS="-D__FreeBSD__=10 -D_POSIX_C_SOURCE=200112L -O -g -arch i386 -arch ppc -isysroot /Developer/SDKs/MacOSX10.4u.sdk" LDFLAGS="-arch i386 -arch ppc -framework CoreFoundation" ./configure --prefix=/usr/local --disable-dependency-tracking
$ make
$ sudo make install
```

This installs the MacFUSE user library and headers under the `/usr/local/` hierarchy.

## 4. Install glib ##

Some FUSE-based file systems depend on `glib`, which is not available by default on Mac OS X.  You don't **need** `glib` to run MacFUSE, but if you're going to be compiling existing FUSE file systems (those that were written for Linux), having `glib` will be required in some cases.To compile `glib` without pain, you will also need the GNU `gettext` package installed.

You can download `gettext` from:

[ftp://ftp.gnu.org/pub/gnu/gettext/](ftp://ftp.gnu.org/pub/gnu/gettext/)

You can compile gettext (Universally) as follows:

```
$ tar -xzvf gettext-0.16.1.tar.gz
$ cd gettext-0.16.1
$ CFLAGS="-O0 -g -arch i386 -arch ppc -isysroot /Developer/SDKs/MacOSX10.4u.sdk" \
  LDFLAGS="-Wl,-syslibroot,/Developer/SDKs/MacOSX10.4u.sdk -arch i386 -arch ppc -fno-common" \
  ./configure --prefix=/usr/local --disable-dependency-tracking \
  --with-libiconv-prefix=/Developer/SDKs/MacOSX10.4u.sdk/usr
$ make
$ sudo make install
```

You can download `glib` from:

[ftp://ftp.gtk.org/pub/glib/](ftp://ftp.gtk.org/pub/glib/)

You can compile `glib` (Universally) as follows:

```
$ tar -xzvf glib-2.13.1.tar.gz
$ cd glib-2.13.1
$ CFLAGS="-O0 -g -D_POSIX_C_SOURCE=200112L -arch i386 -arch ppc \
  -isysroot /Developer/SDKs/MacOSX10.4u.sdk -I/usr/local/include" \
  LDFLAGS="-Wl,-syslibroot,/Developer/SDKs/MacOSX10.4u.sdk -arch i386 -arch ppc -L/usr/local/lib" \
  ./configure --prefix=/usr/local --disable-dependency-tracking --enable-static
```

If you are on a **PowerPC** Macintosh, comment out one line in `config.h`: `// #define G_ATOMIC_POWERPC 1`

Continuing on any Macintosh (x86 or PowerPC -- not tested on ARM yet :-)), open glibconfig.h and locate the line that begins with `#define G_BYTE_ORDER`... replace it with this code block:

```
#if (4 != sizeof(long))
#error no support for 64-bit compiles
#endif
#ifdef __LITTLE_ENDIAN__
#define G_BYTE_ORDER G_LITTLE_ENDIAN
#elif defined(__BIG_ENDIAN__)
#define G_BYTE_ORDER G_BIG_ENDIAN
#else
#error neither __LITTLE_ENDIAN__ nor __BIG_ENDIAN__ #defined
#endif
```

Now open the file `libtool` and locate the lines (I see two of them) that begin with `sys_lib_search_path_spec=`... replace the entire line (in both cases) with:

```
sys_lib_search_path_spec=" /Developer/SDKs/MacOSX10.4u.sdk/usr/lib/ /usr/local/lib"
```

Now, back in the terminal:
```
$ make
$ sudo make install
```

At this point, you should be able to do the following:

```
$ pkg-config --list-all
...
gmodule-export-2.0    GModule - Dynamic module loader for GLib
gmodule-2.0           GModule - Dynamic module loader for GLib
glib-2.0              GLib - C Utility Library
gobject-2.0           GObject - GLib Type, Object, Parameter and Signal Library
gthread-2.0           GThread - Thread support for GLib
gmodule-no-export-2.0 GModule - Dynamic module loader for GLib
fuse                  fuse - File System in User Space (MacFUSE)
```

Now you should also be able to compile many generally available FUSE file systems. Let us look at `sshfs`.

## 5. Install sshfs ##

`sshfs` can be downloaded from:

http://fuse.sourceforge.net/sshfs.html

Suppose you downloaded `sshfs-fuse-1.7.tar.gz`. You'll need to patch this source using an appropriate patch file available in the `filesystems/sshfs/` subdirectory in the MacFUSE subversion repository.

**NB**: The `sshfs` binary package available from this project's website contains an `sshfs-static` binary that is statically linked against `glib` (but dynamically linked against `libfuse`). If you wish to quickly try out `sshfs`, you can just install the MacFUSE Core package and use `sshfs-static` without having to compile anything.

```
$ tar -xzvf sshfs-fuse-1.7.tar.gz
$ cd sshfs-fuse-1.7
$ patch -p1 < /path/to/sshfs-fuse-1.7-macosx.patch
...
$ CFLAGS="-D__FreeBSD__=10 -DSSH_NODELAY_WORKAROUND -O -g -arch i386 -arch ppc -isysroot /Developer/SDKs/MacOSX10.4u.sdk" LDFLAGS="-arch i386 -arch ppc" ./configure --prefix=/usr/local --disable-dependency-tracking
$ sudo make install
```

This will install the `sshfs` command-line program under `/usr/local/bin/`.

# Compiling FUSE File Systems #

Many existing FUSE file systems are readily usable on Mac OS X with MacFUSE. Some might have additional dependencies (typically open source libraries that might not be already installed on your Mac OS X machine), which you have to satisfy before compiling and using the file system in question.

When running the configure script for a file system, you **need** to have `-D__FreeBSD__=10` in `CFLAGS`. This is critical!