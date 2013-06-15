---
author: michael
title: Building android AOSP on Mac OS X Mountain Lion
layout: blog
number: 3
tags:
- android
- aosp
- brew
- macosx
- mountain lion
- 10.8
---
<p>
I have been working with the Android AOSP source code for some time now and spent endless hours searching the internet for problems that arose on different occasions. Today I decided to note them down, on the one hand it is convenient for me to just look it up on my own blog, on the other hand you might find some of these notes useful. 
</p>

<p>
What I ran into today was upgrading from Android 4.0.3 to 4.2.2. I didn't expect any difficulties in compiling the new source as compiling Android 4.0.3 on my Mac OS X (Mountain Lion) worked like a charm for me. But of course I discovered something disturbing - when I ran <code>lunch 12</code>, I got an error:
</p>

<p>
{% highlight bash %}
xcrun: Error: failed to exec real xcrun. (No such file or directory)
build/core/combo/HOST_darwin-x86.mk:62: *****************************************************
build/core/combo/HOST_darwin-x86.mk:63: * Cannot find SDK 10.6 at /Developer/SDKs/MacOSX10.6.sdk
build/core/combo/HOST_darwin-x86.mk:65: * If you wish to build using higher version of SDK, 
build/core/combo/HOST_darwin-x86.mk:66: * try setting BUILD_MAC_SDK_EXPERIMENTAL=1 before 
build/core/combo/HOST_darwin-x86.mk:67: * rerunning this command 
build/core/combo/HOST_darwin-x86.mk:69: *****************************************************
build/core/combo/HOST_darwin-x86.mk:70: *** Stop..  Stop.

** Don't have a product spec for: 'full_maguro'
** Do you have the right repo manifest?
{% endhighlight %}
</p>

<h2> What the hell is it complaining about?</h2>
<p>
Quickly looking up what xcrun is doing I discovered it's some helper program do select the correct XCode version/sdk etc. It didn't suprise me that it failed as I don't have XCode installed. But why would it want somthing from XCode?
</p>

<p>
I don't really have an answer to that, but I discovered that some genius decided that everyone building on Mac OS X has XCode installed. There's no option nor flag nor anything else to disable the use of these XCode specific tools - at least I didn't find them. However, I don't want to complain about it (too much) so without further ado I'll tell you how to compile Android 4.2.2 on Mac OS X Mountain Lion. I'll even try to do it in a way that you can do it when starting from scratch. Please keep in mind that I had my system running for a long time so I might have missed some steps when trying to backtrack the essentials.
</p>

<h2> Setting up the build environment </h2>

<p>
At this point I should point out that there is an excellent documentation for setting up the environment <a href="http://source.android.com/source/initializing.html">here</a> and if anything goes wrong you should check it and it might solve your problem.
</p>

<h3>Homebrew and Prequesites</h3>

<p>
I am using <a href="http://mxcl.github.io/homebrew/">brew</a> to install packages that are not shipped with the os and will setup the build environment using it. Of course you are free to use whatever you prefer, but I can highly recommend brew. After your system is ready to brew (check it with <code>brew doctor</code>), execute the following commands:
</p>

{% highlight bash %}

brew install automake
brew tap homebrew/dupes
brew install apple-gcc42

{% endhighlight %}

<p>
Moreover you have to install Java 1.6, for that just open the Terminal and type <code>java -version</code>. If you haven't installed it the system will prompt you whether you want to install it.
</p>

<h3>Prepare a virtual disk to hold the android source</h3>

<p>
The default file system on Mac OS X is case-insensitive, meaning that <i><b>Test</b></i> and <i><b>test</b></i> would refer to the same file or directory. For some reason there are files in the linux kernel that differ only in the case of a letter - this is something you'd probably do after 72 hours of not sleeping and having a few drinks, at least that's the situation I imagine when trying to grasp why someone would do that... Anyhow, either you have a disk with a case-sensitive filesystem or you create a virtual disk and mount it. To create the virtual disk, mount it and enter the directory run:
</p>

{% highlight bash %}

hdiutil create -type SPARSE -fs 'Case-sensitive Journaled HFS+' -size 40g ~/android.dmg
hdiutil attach ~/android.dmg.sparseimage -mountpoint /Volumes/android
cd /Volumes/android

{% endhighlight %}


<h3>Downloading the source</h3>

<p>
First we need to install <code>repo</code> which is a program that helps to organize the git repositories.
</p>

{% highlight bash %}

mkdir ~/bin
PATH=~/bin:$PATH
curl https://dl-ssl.google.com/dl/googlesource/git-repo/repo > ~/bin/repo
chmod a+x ~/bin/repo

{% endhighlight %}

<p>
If you have followed along, you're still in the root directory of the mounted disk. If not you'll have to <code>cd /Volumes/android</code>. Finally to download the code run:
</p>

{% highlight bash %}

repo init -u https://android.googlesource.com/platform/manifest -b android-4.2.2_r1.2 --config-name
repo sync

{% endhighlight %}

<p>
This will take a while, go do something else until it finishes.
</p>

<h3>Building for a device</h3>

<p>
These steps depend on the device you're building for, I will give you the steps for <b>Galaxy Nexus</b> (maguro) and you have to adapt them accordingly. First download the necessary drivers from <a href="https://developers.google.com/android/nexus/drivers">the nexus driver page</a> and extract them to <code>/Volumes/android</code>. Now you should have a directory listing similar to this one:
</p>

{% highlight bash %}

$ cd /Volumes/android
$ ls
Makefile			external			libcore
abi				extract-broadcom-maguro.sh	libnativehelper
bionic				extract-imgtec-maguro.sh	ndk
bootable			extract-invensense-maguro.sh	out
build				extract-nxp-maguro.sh		packages
cts				extract-samsung-maguro.sh	pdk
dalvik				extract-widevine-maguro.sh	prebuilts
development			frameworks			sdk
device				gdk				system
docs				hardware			vendor

{% endhighlight %}

<p>
Now run each of the <b>extract-...</b> shell scripts. You will be asked to accept its license and you should (well, otherwise the drivers won't be extracted). Also you might have to unlock your bootloader etc. but I'll leave that to you; it's a well documented procedure.
</p>

<h3> Building the source code </h3>

<p> Just to be on the safe side, the android build scripts depend on being executed by <code>bash</code> therefore run it if you don't already. Moreover I had problems with <code>perlbrew</code>, if you have it installed it'd be wise to switch to the perl version that is shipped with Mac OS X now. </p>

<p>
Now there comes the time for the hack, download this <a href="http://tryge.com/downloads/HOST_darwin-x86.mk">file</a> (<b> android-4.2.2_r1.2 only !!! </b>) and overwrite the file in <code>build/core/combos/HOST_darwin-x86.mk</code>. I changed the file to allow for manually overriding the host compilers <code>HOST_CC</code> and <code>HOST_CXX</code> and skip the xcode test if they are both present. I don't use makefiles at all so there might be a more elegant solution to the problem but this works. After you have replaced the files you can execute:
</p>

{% highlight bash %}

. build/envsetup.sh
HOST_CC=gcc-4.2 HOST_CXX=g++-4.2 lunch 
make HOST_CC=gcc-4.2 HOST_CXX=g++-4.2 -j8

{% endhighlight %}

<p>
This will again take a while and if you followed all the steps the compilation succeeds.
</p>

<h2> Final Notes </h2>

<p>
Most of the steps I presented here are from the <a href="">official android documentation</a> and I recommend reading these docs for further information or troubleshooting. I hope I could help some of you :)
</p>


