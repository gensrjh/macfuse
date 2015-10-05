Mac OS X, like many other Unix-like operating systems, includes the “autofs” file system layer that make automatic on-demand mounting of remote resources possible. See the man page for `automount(8)` for more details.

Such automatic mounting is orthogonal to and possible with MacFUSE. Consider `sshfs`, a user-space SFTP file system implementation that works with MacFUSE. The following is a quick-and-dirty example of how you could set up an autofs mount for `sshfs`. (There are other ways to set up autofs mounts.)

Create an `/etc/fstab` file (or add to an existing one) with the following entry. We will create what’s called a "static map" in autofs parlance.

```
 $ cat /etc/fstab
dummy:user@host:/remotedir /Network/name sshfs volname=volname,allow_other 0 0
```

You will have to customize some components of this specification. `user`, `host`, and `remotedir` are the SFTP user name, the SFTP server host name, and the remote directory on the SFTP server, respectively. You can choose some reasonable value for `name` and `volname`. The local mount point will be `/Network/name`.

Next, to keep things simple, configure key-based authentication to the SFTP server so you can log in without having to type your password.

The keyword “sshfs” in the `/etc/fstab` entry is the type of the file system. Given a file system type `foo`, the automounter will expect a `mount_foo` file-system-specific mounting program to exist. In our case, we don’t have a separate mounting program for `sshfs`. However, because of the format of the entry and how the automounter passes arguments to the mounting program, it will work if you simply copy the command-line `sshfs` program to `/sbin/mount_sshfs`. Alternatively, you can create a symbolic link as follows.

```
$ which sshfs
$ /usr/local/bin/sshfs
$ sudo ln -s /usr/local/bin/sshfs /sbin/mount_sshfs
```

That should be it. Run the `automount` program to update the state of things. The `-c` argument tells the automount daemon to flush any cached information.

```
 $ sudo automount -c
```

If everything went well, the new mount should appear in the output of the `mount` command. In the following example, we used SSH as the name component of the mount point.

```
$ mount
...
map -static on /Network/SSH (autofs, automounted)
```

Now, if you simply access `/Network/SSH`, the SFTP file system should be automatically mounted.

```
$ ls /Network/SSH
Applications		Volumes			work
Desktop DB		bin			private
...
```

If there is an error in mounting (say, the remote host is not reachable), you will not be permitted to access the `/Network/SSH` directory.

```
$ ls /Network/SSH
ls: SSH: Operation not permitted
```

You can specify a timeout period after which an automounted file system will be unmounted if it has not been accessed within that period. Either use the `-t` argument of `automount` or see the `/etc/autofs.conf` file.