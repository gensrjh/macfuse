Due to popular demand, I added support for custom volume icons beginning with MacFUSE 0.4.0. Before this, you _could_ have a custom volume icon for your MacFUSE file system, but it was painful and required the file system to do extra work, specifically, you need to do the following:

  * Implement a `/.VolumeIcon.icns` file within your file system. This file's contents need to be that of the icon file.
  * At mount time, suppose the volume is mounted on `/Volume/Foo/`, someone (some user interface code, or perhaps the file system itself) needs to create a "dot-underscore" file (`/Volume/._Foo`) containing metadata that designates the volume as having a custom icon.

Besides the extra work being, well, "extra", this approach is also somewhat inflexible because the user can't easily change the icon arbitrarily.

MacFUSE 0.4.0 adds a new mount-time option (sigh... another option) called `volicon`. You use it as follows:

```
$ sshfs user@host:/dir /some/mount/point -ovolicon=/path/to/somefile.icns
```

That's it. Just add `-ovolicon=/path/to/somefile.icns` and the volume will show up with the icon contained in `somefile.icns`. MacFUSE (the user-space library, specifically) will take care of serving the `/.VolumeIcon.icns` file. MacFUSE will also make a "best-effort" attempt at creating the dot-underscore file at mount time, and another best-effort attempt at deleting the dot-underscore file at unmount time.

File system developers, take note of the following:

  * MacFUSE automagically serves the file by fulfilling `access()`, `fgetattr()`, `getattr()`, `open()`, and `read()` requests itself (instead of passing them to your file system as it normally would) if the path in question happens to be `/.VolumeIcon.icns`. However, it doesn't do anything different in the case of `readdir()`. Therefore, you will not see the icon file when you simply list the contents of your file system's root directory. However, if you specifically stat the icon file or open it for reading, it will be there.
  * If the dot-underscore file for the mount point already exists (say, from a previous mount), MacFUSE will not overwrite it. This file, which should be 82 bytes in size, should be identical for all MacFUSE volumes. The file's creator code should be 'FUSE' and its type should be 'ROOT'.
  * At unmount time, MacFUSE will try to delete an existing dot-underscore file only if the creator code and type are 'FUSE' and 'ROOT', respectively, and only if the custom icon option is in effect. If you find any stale files, it's fine to delete them, and equally fine to leave them around if you'd be reusing that mount point later. Since these files always have the exact same content--neither icon-specific nor mount-point-specific--it doesn't matter either way.
  * As a sanity-check, the icon file must be no larger than 1MB. Of course, the icon file must also be readable by the user mounting the file system.
  * While developing, if you experiment with different icon files, know that the Finder's in-memory cache causes it to "remember" icon file contents a bit too much. Even if you pass another icon file through the `volicon` option, the Finder will keep using the old one. Relaunching the Finder (through `killall Finder`, for example) should "fix" this.

A substantial amount of new code was added to the user-space library to handle all of this. Please test enough. If you don't use the `volicon` option, the newly added code will all be bypassed.

**NB**: Custom volume icon support is implemented as a "stack module" within the MacFUSE user-space library. The `volicon=/path/to/icon/file` argument is actually a convenience shortcut for the following long form:

```
    ... -omodules=volicon -oiconpath=/path/to/icon/file
```

The shortcut form will work **only** if you do not have any other stack module enabled. If you do, the library will not parse module arguments correctly. For example, if you want to use custom volume icons together with the `subdir` stack module (which takes one argument called `subdir`, the same as the stack module's name), you have to do the following:

```
    ... -omodules=subdir:volicon -oiconpath=/path/to/icon/file -osubdir=/path/to/subdir
```