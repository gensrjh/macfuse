# MacFUSE Core Version Number Format #

When a new MacFUSE Core release occurs, separate installer disk images are available for Mac OS X 10.4 ("Tiger") and 10.5 ("Leopard"). MacFUSE Core releases have version numbers of the form `x.y.z`, where you can think of `x.y` as the release number, and `z` as the "Tiger"/"Leopard" differentiator. `z` is always `0` (smaller) for "Tiger" and `1` (greater) for "Leopard".

For example, MacFUSE Core release `1.1` would have a "full" version number `1.1.0` for "Tiger" and `1.1.1` for "Leopard".

You might be curious as to why I'm doing this differentiation. Perhaps I'll explain some other time. However, I'd strongly advise both developers and users of MacFUSE to **not** depend on this "format" being permanent.

Note that regardless of the version numbers, the installer packages wouldn't install on the "wrong" operating system.

# Determining MacFUSE Core Version #

There are several ways to know which MacFUSE version you have available, both as a file system developer and an end user.

## Call a Function in the User-Space Library ##

If you are a file system developer, you can call the following function that's provided by the user-space MacFUSE library. It will return a version string of the aforementioned form `x.y.z`.

```
#include <fuse_darwin.h>

const char *macfuse_version(void);
```

## Run the `mount_fusefs` Program ##

You can run the `mount_fusefs` command-line program, a MacFUSE Core component, to determine the version of the MacFUSE Core suite it's part of. `mount_fusefs` resides in the `Support` subdirectory of the `fusefs.fs` file system bundle, which lives under `/Library/Filesystems/` on "Leopard" and `/System/Library/Filesystems` on "Tiger". Note the use of the environment variable `MOUNT_FUSEFS_CALL_BY_LIB`.

```
$ MOUNT_FUSEFS_CALL_BY_LIB=1 /Library/Filesystems/fusefs.fs/Support/mount_fusefs --version 
MacFUSE mount version 1.1.1
```

## Run `defaults` ##

The MacFUSE Core file system bundle contains the version number in its `Info.plist` file. So does the MacFUSE kernel extension. You can retrieve the version number from there using, say, the `defaults` command, or through an equivalent Core Foundation API function.

```
# Command line
$ defaults read /Library/Filesystems/fusefs.fs/Contents/Info CFBundleVersion 
1.1.1
```

## Use `sysctl` ##

You can also query the version of the MacFUSE kernel extension, both within your own code and through a command-line program. Note that both the user-space library and the kernel extension share the same version number. However, this approach works only if the kernel extension is loaded at the time you query. Since the kernel extension is dynamically loaded as needed, this is not a foolproof technique to determine if you have a certain version of MacFUSE Core available (or if you have it even installed). The following is an example of using the `sysctl` command to retrieve this information.

```
# Command line
$ sysctl macfuse.version
macfuse.version.api_major: 7
macfuse.version.api_minor: 8
macfuse.version.number: 1.1.1
macfuse.version.string: 1.1.1, Nov  6 2007, 15:37:11
...
```

You could also use `sysctlbyname(3)` to retrieve this information at runtime (not recommended).