![http://macfuse.googlecode.com/svn/trunk/core/packaging/images/MacFUSE_Banner.png](http://macfuse.googlecode.com/svn/trunk/core/packaging/images/MacFUSE_Banner.png)

**Please Note: This project is no longer being maintained. We cannot currently help with any lion (or any large feline) related bugs or issues. The [MacFuse google group](http://groups.google.com/group/macfuse) is a decent resource for finding forks/replacement projects.**

MacFUSE allows you to extend Mac OS X's native file handling capabilities via 3rd-party file systems. It is used as a software building block by dozens of products.

As a user, installing the MacFUSE software package will let you use any 3rd-party file system written atop MacFUSE.

As a developer, you can use the MacFUSE SDK to write numerous types of new file systems as regular user-mode programs. The content of these file systems can come from anywhere: from the local disk, from across the network, from memory, or any other combination of sources. Writing a file system using MacFUSE is orders of magnitude easier and quicker than the traditional approach of writing in-kernel file systems. Since MacFUSE file systems are regular applications (as opposed to kernel extensions), you have just as much flexibility and choice in programming tools, debuggers, and libraries as you have if you were developing standard Mac OS X applications.

In more technical terms, MacFUSE implements a mechanism that makes it possible to implement a fully functional file system in a user-space program on Mac OS X (10.4 and above). It provides multiple APIs, one of which is a  superset of the FUSE (File-system in USEr space) API that originated on Linux. Therefore, many existing FUSE file systems become readily usable on Mac OS X.

The MacFUSE software consists of a kernel extension and various user-space libraries and tools. It comes with C-based and Objective-C based SDKs. If you prefer another language (say, Python or Java), you should be able to create file systems in those languages after you install the relevant language bindings yourself.

To see some examples of MacFUSE at work, see the videos linked on the right.

The MacFUSE source repository contains source code for several exciting and useful file systems for you to browse, compile, and build upon, such as sshfs, procfs, AccessibilityFS, GrabFS, LoopbackFS, SpotlightFS, and YouTubeFS.

**NB**: Please post all your MacFUSE-related questions and issues in the [MacFUSE discussion forum](http://groups.google.com/group/macfuse). Do search to see if your question or problem is already discussed. You should also look at the [MacFUSE wiki](http://code.google.com/p/macfuse/w/list) and the [archive of old issues](http://code.google.com/p/macfuse/issues/list).