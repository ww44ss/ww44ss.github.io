---
layout: post
title: Install RWeka
excerpt: "Getting Java Dependencies Right"
modified: 2017-01-26
categories: articles
tags: [R, code, Java, package, install.packages]
image:
  feature: so-simple-sample-image-1.jpg
  credit: WeGraphics
  creditlink: http://wegraphics.net/downloads/free-ultimate-blurred-background-pack/
comments: true
share: true
---

I had to install the R package RWeka but kept getting an error associated with the version of the JavaVM available on my Mac. Since it'd been a while since I updated it, I had to resurrect how that could be done. Here it is for posterity. Hopefully Ill recall this blog post when I have to go thru this again...

`
> install.packages("RWeka")

There is a binary version available but the source version is later:
binary source needs_compilation
RWeka 0.4-26 0.4-31             FALSE

installing the source package ‘RWeka’

trying URL 'https://cran.rstudio.com/src/contrib/RWeka_0.4-31.tar.gz'
Content type 'application/x-gzip' length 409473 bytes (399 KB)
==================================================
downloaded 399 KB

* installing *source* package ‘RWeka’ ...
** package ‘RWeka’ successfully unpacked and MD5 sums checked
** R
** inst
** preparing package for lazy loading
JavaVM: requested Java version ((null)) not available. Using Java at "" instead.
JavaVM: Failed to load JVM: /bundle/Libraries/libserver.dylib
JavaVM FATAL: Failed to load the jvm library.
Error : .onLoad failed in loadNamespace() for 'RWekajars', details:
...
`

It turns out I needed to do three things:

## 1. Update my version of Java. 

I downloaded from [java.com](https://java.com/en/download/mac_download.jsp)

__ Recommended Version 8 Update 121 (filesize: 62.28 MB)__
_Release date January 17, 2017_

## 2. Reconfigure Java on my Mac

I did this by opening the terminal and running on the command line:
` sudo R CMD javareconf`

`
Winstons-Air:~ winstonsaunders$ sudo R CMD javareconf
Password:

Java interpreter : /usr/bin/java
Java version     : 1.8.0_101
Java home path   : /Library/Java/JavaVirtualMachines/jdk1.8.0_101.jdk/Contents/Home/jre
Java compiler    : /usr/bin/javac
Java headers gen.: /usr/bin/javah
Java archive tool: /usr/bin/jar
Non-system Java on macOS

trying to compile and link a JNI program 
detected JNI cpp flags    : -I$(JAVA_HOME)/../include -I$(JAVA_HOME)/../include/darwin
detected JNI linker flags : -L$(JAVA_HOME)/lib/server -ljvm
clang -I/Library/Frameworks/R.framework/Resources/include -DNDEBUG -I/Library/Java/JavaVirtualMachines/jdk1.8.0_101.jdk/Contents/Home/jre/../include -I/Library/Java/JavaVirtualMachines/jdk1.8.0_101.jdk/Contents/Home/jre/../include/darwin -I/usr/local/include -I/usr/local/include/freetype2 -I/opt/X11/include    -fPIC  -Wall -mtune=core2 -g -O2  -c conftest.c -o conftest.o
clang -dynamiclib -Wl,-headerpad_max_install_names -undefined dynamic_lookup -single_module -multiply_defined suppress -L/Library/Frameworks/R.framework/Resources/lib -L/usr/local/lib -o conftest.so conftest.o -L/Library/Java/JavaVirtualMachines/jdk1.8.0_101.jdk/Contents/Home/jre/lib/server -ljvm -F/Library/Frameworks/R.framework/.. -framework R -Wl,-framework -Wl,CoreFoundation


JAVA_HOME        : /Library/Java/JavaVirtualMachines/jdk1.8.0_101.jdk/Contents/Home/jre
Java library path: $(JAVA_HOME)/lib/server
JNI cpp flags    : -I$(JAVA_HOME)/../include -I$(JAVA_HOME)/../include/darwin
JNI linker flags : -L$(JAVA_HOME)/lib/server -ljvm
Updating Java configuration in /Library/Frameworks/R.framework/Resources
Done.
`

## 3. Install the package 'rJava'

`
> install.packages("rJava",type='source')
trying URL 'https://cran.rstudio.com/src/contrib/rJava_0.9-8.tar.gz'
Content type 'application/x-gzip' length 656615 bytes (641 KB)
==================================================
downloaded 641 KB

* installing *source* package ‘rJava’ ...
`

(this generated a ton of text output)

## 4. Install 'Rweka'


`
> install.packages("RWeka")

There is a binary version available but the source version is later:
binary source needs_compilation
RWeka 0.4-26 0.4-31             FALSE

installing the source package ‘RWeka’

trying URL 'https://cran.rstudio.com/src/contrib/RWeka_0.4-31.tar.gz'
Content type 'application/x-gzip' length 409473 bytes (399 KB)
==================================================
downloaded 399 KB

* installing *source* package ‘RWeka’ ...
** package ‘RWeka’ successfully unpacked and MD5 sums checked
** R
** inst
** preparing package for lazy loading
** help
*** installing help indices
** building package indices
** installing vignettes
** testing if installed package can be loaded
* DONE (RWeka)

The downloaded source packages are in
‘/private/var/folders/9z/44by6fj17mx2wvczwr6q42r80000gn/T/RtmpPIoUju/downloaded_packages’
`


Works like a charm!


