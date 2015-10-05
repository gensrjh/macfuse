# NOTE #

This page is currently being updated to reflect the MacFUSE 2.0 release. The update is almost done, so feel free to try it out. This message will go away when the update is complete.

# Introduction #

MacFUSE comes with a framework to assist in developing user-space file systems written in Objective-C.  The framework resides in _/Library/Frameworks/MacFUSE.framework_.  This HOWTO walks through an example of creating a simple Cocoa-app file system in Objective-C that uses _MacFUSE.framework_. The author of this HOWTO gave a presentation at CocoaHeads that may be another useful source of information on writing file system using Objective-C. The video for that presentation can be found [here](http://theocacao.com/document.page/543), but keep in mind that this HOWTO has been revised since the presentation.

# Requirements #

In order to go through this HOWTO, you'll need to:
  1. Be running XCode 3.1.1+ on Leopard+.
  1. If you haven't already, then install [MacFUSE](http://code.google.com/p/macfuse/downloads) 2.0+.
  1. Check out the helper files that will be used in our tutorial project (you can do all the work in your _/tmp_ directory if you'd like):
```
cd /tmp
svn checkout http://macfuse.googlecode.com/svn/trunk/filesystems-objc/Support/ Support
```

# The Tutorial Project #

We'll create a simple read-only file system that downloads a feed of top-rated [YouTube](http://www.youtube.com/) videos and presents them in the root directory as a clickable webloc to the video. A completed version of this tutorial can be found [here](http://macfuse.googlecode.com/svn/trunk/filesystems-objc/YTFS/), but it is highly recommended that you follow along with this tutorial to get an understanding of how things work. It should be fun and painless!

# Project Setup #

Start Xcode and choose _File -> New Project_. From here select _MacFUSE_ under _User Templates_ on the left and then choose the _Objective-C File System (Read-Only)_ project template to create your project. We'll refer to this project as YTFS from now on.

At this point you should have a working read-only file system! If you _Build and Go_ from Xcode then your YTFS file system should mount and a Finder window will appear showing an empty root directory. If you quit the YTFS application then the file system will unmount; conversely if you unmount the file system the application will terminate.

If you inspect the files in your project then you'll see the `YTFS_FileSystem` and `YTFS_Controller` classes. The `YTFS_Controller` class will control the life cycle of both our app and the file system, while the `YTFS_FileSystem` class will serve the contents of our file system.

We still need a few more classes in order to complete YTFS. From the _Support_ directory that you checked out earlier, add the _NSImage+IconData.{h,m}_ and _YTVideo.{h,m}_ files to the project. The former will be used for our thumbnails; it provides handy code to turn an `NSImage` into `NSData` for the _.icns_ file that represents the image. The latter is a small class that fetches an xml feed of top videos from YouTube and provides a class to access information about the videos.

Finally, in order to use `NSImage+IconData` you'll need to add _Accelerate.framework_ to the project.

# Time To Code #

I recommend that as you follow along in this you take the time to compile and run the app at each step. This iterative process will give you the best idea about what each step is accomplishing.

## Part 1: YTFS\_Controller.m ##

For the first part, we'll make changes to the _YTFS\_Controller.m_ file that handles the life cycle of our file system.

### Include YTVideo.h Header File ###

```
#import YTVideo.h
```

### Initialize the File System With Videos ###

Change the following:

```
fs_delegate_ = [[YTFS_Filesystem alloc] init];
```

to:

```
fs_delegate_ = [[YTFS_Filesystem alloc] initWithVideos:[YTVideo fetchTopRatedVideos]];
```

Note: The code won't compile cleanly any more until we do a bit of work in _YTFS\_Filesystem.m_.

### GMUserFileSystem, Mounting, and Notifications ###

If you look at the template-generated controller code in `applicationDidFinishLaunching:`, you will see that it starts off by registering for MacFUSE file system notifications. This is how it handles quitting the application on unmount and showing you a Finder window when the file system finishes mounting.

It then allocates a `GMUserFileSystem` instance to handle the file system and gives it a `YTFS_Filesystem` instance as the delegate that serves file system contents. By the way, since our file system is thread-safe, you may optionally change `isThreadSafe:NO` to `isThreadSafe:YES` in the line that initializes `fs_`.

Finally, it sets up the MacFUSE options and instructs the `GMUserFileSystem` instance to mount. You always need to carefully consider what [options](http://code.google.com/p/macfuse/wiki/OPTIONS) to use when mounting your file system. By default, with this project template the file system is mounted read-only (`rdonly`) and uses a custom name and icon so that things look nice.

## Part 2: YTFS\_Filesystem.h ##

Add a member variable to hold our video dictionary:

```
@interface YTFS_Filesystem : NSObject  {
  NSDictionary* videos_;
}
```

We'll also need an iniitalizer:

```
- (id)initWithVideos:(NSDictionary *)videos;
```

## Part 3: YTFS\_Filesystem.m ##

Now we'll be fleshing out the code that serves the file system contents. Open _YTFS\_Filesystem.m_ and take a look around. You should see that the complete set of [delegate methods](http://macfuse.googlecode.com/svn/trunk/core/sdk-objc/Documentation/GMUserFileSystem/Categories/NSObject_GMUserFileSystemOperations_/index.html) that you can implement to serve read-only file system data are already stubbed out for you.

### Add Headers For Utility Classes ###

```
#import "YTVideo.h"
#import "NSImage+IconData.h"
```

### Remove Unused Delegate Selectors ###

We actually don't need to use all of the delegate methods for this tutorial, so feel free to remove these selectors if you'd like:

  * `attributesOfFileSystemForPath:error:`
  * `openFileAtPath:mode:userData:error:`
  * `releaseFileAtPath:userData:`
  * `readFileAtPath:userData:buffer:size:offset:error:`
  * `destinationOfSymbolicLinkAtPath:`
  * `extendedAttributesOfItemAtPath:error:`
  * `valueOfExtendedAttribute:ofItemAtPath:position:error:`

### Add Init and Helper Methods ###

We'll add our initialization and cleanup methods as well as a helper method that looks up a video based on a given path. The should go near the top of the `YTFS_Filesystem` implementation, just above `contentsOfDirectoryAtPath:error:`.

```
- (id)initWithVideos:(NSDictionary *)videos {
  if ((self = [super init])) {
    videos_ = [videos retain];
  }
  return self;
}
- (void)dealloc {
  [videos_ release];
  [super dealloc];
}
- (YTVideo *)videoAtPath:(NSString *)path {
  NSArray* components = [path pathComponents];
  if ([components count] != 2) {
    return nil;
  }
  YTVideo* video = [videos_ objectForKey:[components objectAtIndex:1]];
  return video;
}
```

Things should compile cleanly again at this point.

### Directory Contents ###

Edit `contentsOfDirectoryAtPath:error` to show our root directory contents:

```
- (NSArray *)contentsOfDirectoryAtPath:(NSString *)path 
                                 error:(NSError **)error {
  return [videos_ allKeys];
}
```

If you run it now you should see a bunch of .webloc files. If you click on them then you'll be disappointed.

### File Attributes ###

Since the Finder will ask about a lot of files that probably don't exist, every file system should implement `attributesOfItemAtPath:userData:error:`. Edit it to be as follows:

```
- (NSDictionary *)attributesOfItemAtPath:(NSString *)path 
                                   error:(NSError **)error {
 if ([self videoAtPath:path]) {
    return [NSDictionary dictionary];
  }
  return nil;
}
```

The framework will translate a `nil` return value into `ENOENT`. Another option would be to explicitly set the error out parameter to an `NSError` in the `NSPOSIXErrorDomain`. You can only return POSIX errors from your delegate methods in the `NSError` out parameter. The good news is that the project template provides a category on `NSError` to make this easier.

### File Contents ###

Just to show you how to do it, let's return the xml data for the video as the file contents. A _.webloc_ file can actually be 0 bytes, since the url is stored in the resource fork. If you prefer you can return an empty `NSData` as the contents or remove the `contentsAtPath:` delegate method altogether. However, let's implement it so that you can drag the file to _TextEdit.app_ and see the xml data that we've been using:

```
- (NSData *)contentsAtPath:(NSString *)path {
  YTVideo* video = [self videoAtPath:path];
  if (video) {
    return [video xmlData];
  }
  return nil;
}
```

### Webloc URLs ###

What good are weblocs if you can't click on them? Change `resourceAttributesAtPath:error:` to be the following:

```
- (NSDictionary *)resourceAttributesAtPath:(NSString *)path
                                     error:(NSError **)error {
  NSMutableDictionary* attribs = nil;
  YTVideo* video = [self videoAtPath:path];
  if (video) {
    attribs = [NSMutableDictionary dictionary];
    NSURL* url = [video playerURL];
    if (url) {
      [attribs setObject:url forKey:kGMUserFileSystemWeblocURLKey];
    }
  }
  return attribs;  
}
```

This looks up the video in our list of videos and, if found, returns the url of the video as a resource fork attribute. _MacFUSE.framework_ will automatically create a resource fork for us so that clicking on the video via the Finder will open up the URL in the default browser. If you run the app now you should be able to double-click on a video and watch it!

### Custom Icons ###

Now that we can view our videos, we'll need some help in figuring out which ones look interesting. We can display a custom icon thumbnail for each of our videos. First change `finderAttributesAtPath:error:` to notify that we have a custom icon in our resource fork:

```
- (NSDictionary *)finderAttributesAtPath:(NSString *)path 
                                   error:(NSError **)error {
  NSDictionary* attribs = nil;
  if ([self videoAtPath:path]) {
    NSNumber* finderFlags = [NSNumber numberWithLong:kHasCustomIcon];
    attribs = [NSDictionary dictionaryWithObject:finderFlags
                                          forKey:kGMUserFileSystemFinderFlagsKey];
  }
  return attribs;
}
```

If we implement this method then _MacFUSE.framework_ will populate _Finder Info_ data with the attributes we give it. We still need to add the custom icon to the file's resource fork, so update `resourceAttributesAtPath:error:` to look like:

```
- (NSDictionary *)resourceAttributesAtPath:(NSString *)path
                                     error:(NSError **)error {
  NSMutableDictionary* attribs = nil;
  YTVideo* video = [self videoAtPath:path];
  if (video) {
    attribs = [NSMutableDictionary dictionary];
    NSURL* url = [video playerURL];
    if (url) {
      [attribs setObject:url forKey:kGMUserFileSystemWeblocURLKey];
    }
    url = [video thumbnailURL];
    if (url) {
      NSImage* image = [[[NSImage alloc] initWithContentsOfURL:url] autorelease];
      NSData* icnsData = [image icnsDataWithWidth:256];
      [attribs setObject:icnsData forKey:kGMUserFileSystemCustomIconDataKey];
    }
  }
  return attribs;  
}
```

You'll notice that we download the thumbnail every time this routine is called. In practice this is not a very good idea and caching the _.icns_ data would be preferred. For this tutorial the system url cache should be sufficient in preventing us from actually downloading from the server repeatedly.

We take advantage of a category on `NSImage` (see _NSImage+IconData.{h,m}_ in the project) to create _.icns_ data for us from an `NSImage` on the fly.

## Conclusion ##

If you've gone through this HOWTO then you've created a working and marginally useful user-space file system! Don't forget to try dragging a video that you like onto the desktop to see what happens.

There is a lot more to using _MacFUSE.framework_, so please read the [documentation](http://macfuse.googlecode.com/svn/trunk/core/sdk-objc/Documentation/index.html) and feel free to ask questions on the [MacFUSE mailing list](http://groups.google.com/group/macfuse).