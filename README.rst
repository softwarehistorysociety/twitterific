MobileTwitterrific.app
======================

Craig Hockenberry - 19.Aug.2007 - Initial version
Craig Hockenberry - 01.Sep.2007 - Added svn to get headers
Craig Hockenberry - 11.Sep.2007 - Updated to use MacPorts (arm-apple-darwin)


Introduction
============

This project is used to create a native iPhone application for using Twitter.

It's secondary purpose is to act as a "best practices" document for creating
native iPhone applications.


Environment
===========

It is assumed that you are using MacPorts to maintain your build tools and
Subversion to maintain your UIKit headers. With this setup, the paths are:

/opt/local/arm-apple-darwin
	/bin		-- build tools: cc, ld, as and ar
	/etc		-- build tools configuration
	/heavenly	-- the frameworks extracted from decrypted disk image
	/include	-- the framework include files for UIKit
	/csu		-- the folder for the mysterious crt1.o
	
We don't recommend using Kroo's Toolchain installer. In spite of it being an
excellent way to get up and running quickly, the installer places older
versions of the UIKit header files and screws up Subversion. You'll be
updating the UIKit headers often -- they change daily as people discover new
things. Keeping Subversion happy is very important.


Setup Toolchain
===============

Create /heavenly
----------------

Locate the latest iPhone restore image that is located in your Home > Library >
iPhone Software Updates. Copy this file to somewhere convenient and change the
name from .ipsw to .zip. Then uncompress the archive using BOMArchiveHelper.

Now find the decryption key in the archive. This command will search for all
hex strings in the disk image and print out the one that is 72 characters long:

% strings 009-7698-4.dmg | grep '^[0-9a-fA-F]*$' | awk '{ if (length($1) == 72) print; }'

Use the key to decrypt the disk image with vfdecrypt (replacing XXXXX with the
72 character key):

% ./vfdecrypt -i iPhone1,1_1.0.2_1C28_Restore/694-5298-5.dmg -k XXXXX -o heavenly.dmg

After you have a decrypted disk image, mount it and copy the files into your
toolchain:

% sudo mkdir -p /opt/local/arm-apple-darwin/heavenly
% sudo cp -Rn /Volumes/SUHeavenlyJuly1C28.UserBundle/ /opt/local/arm-apple-darwin/heavenly

(Note: the names of the disk images and the mounted UserBundle will change with
each release -- figure it out. If you need help getting the disk image mounted,
see these instructions: <http://landonf.bikemonkey.org/code/iphone>)

Create /bin and /etc
--------------------

Now install the toolchain with MacPorts:
 
% sudo port install arm-apple-darwin-binutils
% sudo port install arm-apple-darwin-cc

(Note: if you are having problems building with MacPorts after using Kroo's
installer, make sure to remove the iPhone and heavenly directories in
/Developer/SDKs and to remove "/Developer/SDKs/iPhone/bin" from your PATH.)

(Note: be sure to have the latest version of Xcode installed: v2.4.1. It can
be downloaded from ADC.)

Create /include
---------------

Get the latest header files with:

% sudo svn checkout \
	http://svn.berlios.de/svnroot/repos/iphone-binutils/trunk/include \
	/opt/local/arm-apple-darwin/include

Then symlink CoreGraphics:

% sudo ln -s \
	/System/Library/Frameworks/ApplicationServices.framework/Frameworks/CoreGraphics.framework/Headers/ \
	/opt/local/arm-apple-darwin/include/CoreGraphics

If you have problems compiling, make sure that you have include/WebCore and
that include/CarbonCore has headers (and not .diff files.) Finally, you may
need to comment out some #include "NSObject.h" in Celestial and WebCore.

Create /csu
-----------

To be honest, I have no idea what crt1.o is all about, I just know that you
need it. Here's how to pull the object file out of the Berlios.de repository:

% sudo svn checkout \
	http://svn.berlios.de/svnroot/repos/iphone-binutils/trunk/Csu-71 \
	/opt/local/arm-apple-darwin/csu


Building
========

Open the MobileTwitterrificApp.xcodeproj and build. The project is built using
the Makefile in the project's root directory.

If you add files to the project, you need to update the SOURCES in the
the Makefile. They will not automatically be added to the build. Until there
is better Xcode integration with the iPhone toolchain, that's the way it
has to be.

Note also that there aren't any precompiled headers and there is isn't any
code index. So builds will get slower as the project size increases and you'll
be sorely disappointed by what you see when you hit the ESC key while editing
(no code completion.)

After the build completes, you'll find the iPhone application package in
the build/Release folder. The final build step uses scp to get the application
onto the iPhone in the /Applications folder: you'll need to update the command
in the Makefile to suit your local environment.


Debugging
=========

NSLog. Learn to love it, since printf is all you have at the moment.

otool can also be handy for what's in the executable.

If you have a bug that causes a crash, you can find the crash log in
/var/logs/CrashReporter/ -- the latest crash is conveniently named
LatestCrash.plist

