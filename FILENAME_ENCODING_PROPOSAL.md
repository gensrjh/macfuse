# Objective #

MacFUSE should support filesystems whose filenames are arbitrarily encoded. Currently the assumption MacFUSE makes is that all filenames are UTF-8 encoded, which is sometimes wrong. This can lead to confusing errors and behavior for which there is currently no workaround. The natural fix is to add support for arbitrary filename encodings.

# Background #

Some FUSE filesystems (e.g. sshfs) do not attempt to convert filename encodings at all. Filenames are read in which ever format, treated as a byte-array, and passed verbatim to MacFUSE. In the general English/7-bit ASCII case, filenames are treated fine, but when non-ASCII characters are used, problems can occur, since MacFUSE treats filenames as UTF-8.

A related problem is that copying a file to an HFS+ volume requires the name to be decomposed UTF-8 (normalization form D, or NFD). This has been noted before at http://code.google.com/p/macfuse/issues/detail?id=139 .

By adding support for the fs daemon to declare that it is giving and accepting filenames in a given encoding, MacFUSE can convert filenames to make both Mac OS X/HFS+ and the fs daemon cooperate.

# Overview #

A command-line flag will be added to allow an encoding to be specified. When specified, all filenames from the fs daemon en route to MacFUSE will be converted to UTF-8 NFD. Filenames from MacFUSE en route to the fs daemon will be converted to the encoding specified. Filenames coming from the daemon will be verified to ensure they are, in fact, in the encoding specified.

When conversion fails in a name from the fs daemon, the name will be ignored. This means it will be absent from directory listings. An error may be given to the user.

When conversion fails in a name from MacFUSE, failure will be reported back to MacFUSE. The user much choose a different filename.

# Detailed Design #

The specific component of MacFUSE being modified here is libfuse.

Two new options will be added to to libfuse: `list_encodings` and `encoding=xxx`. The first will print out a list of all possible encoding options in both a human and machine readable format (IANA encoding name, if available). The second option will start libfuse with the encoding specified as an IANA name (or backup name if none is available). If neither is specified, libfuse behaves as before, reading the bytes directly from the fs daemon.

The encodings will be listed by calling the CoreFoundation function `CFStringGetListOfAvailableEncodings()`. Each line of output will specify the IANA encoding name and a human-friendly encoding name. Should an IANA encoding not be available, a substitute of the form `MACFUSE-ENCODING-n` for some number _n_. There will be changes with the Unicode encodings, tho. Since we require a conversion to NFD, the encoding must specify NFC or NFD. Thus the Unicode encodings will need to specify "-NFC" or "-NFD" (e.g., "UTF-8-NFD"). We will also remove UTF-16 and UTF-32 when endianness is not specified. If in the future we allow the encoding to be changed while a filesystem is running, we may be able to support UTF-8 Best-Guess. In this case, the first filename that is invalid in either NFC or NFD will determine that NFD or NFC is the given encoding, respectively.

If an encoding is specified with `encoding=xxx`, that encoding is used to convert all incoming and outgoing filenames. The only way a filename travels from the fs daemon to Mac OS X is via directory listing. If, in a directory listing, a name is given by the fs daemon is invalid for the given encoding, it is omitted in the directory listing. One way to test if the encoding is valid or not is to convert it to NFD and back to the encoding specified. If either conversion fails or this filename doesn't exactly match the original filename, the filename is invalid.

Filenames come from Mac OS X to the fs daemon as part of a filesystem request. If an encoding is specified, the name will be converted to the proper encoding. If the encoding fails, an error (perhaps ENOENT) will be returned to the user. Whichever application triggered this request will see an error, and the user should choose a new name before trying again. MacFUSE has the ability to pop a dialog box to the user (this can happen, for example, when anfs daemon is unresponsive); perhaps MacFUSE could pop up a dialog informing the user that the filename is invalid.

# Caveats #

Users may have difficulty choosing an encoding, since they may not understand what the encodings mean. Even advanced users may be confused by having two choices for UTF-8. This may be alleviated by documentation, but most likely, the user will try one encoding and if things don't work, they will try another. Also, the potential future feature "UTF-8 Best-Guess" may help for the UTF-8 case.

This also does not address the possible case that files may exist in the fs daemon in the same directory in multiple encodings. The case of this that I have personally seen is with sshfs to a Linux box. Linux (rather, ext-3) doesn't care if a filename is normalized to NFC or NFD. Thus two files may exist named "otoño" ("otoño" and "oton~o", where ~ represents COMBINING TILDE=U+0303), but only one will be visible when it is remote mounted over sshfs. I hope that this case is rare. In any event, I don't have a good way to handle it in fuse for all filesystems.

This is a proposed Mac-only feature. It will add a fair amount of code to libfuse that is #ifdef'd out for Linux/FreeBSD/etc. The Linux mount command supports various encodings already, tho, so it may not be so urgent for Linux users. Still, we could consider porting this functionality to Linux.

