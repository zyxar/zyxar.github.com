---
layout: post
title: "install_name on OS X"
date: 2012-03-10 20:29
comments: true
categories: 
---
On OS X, the loader `dyld` does have a search path, defined in the `DYLD_FRAMEWORK_PATH` and `DYLD_LIBRARY_PATH` variables. However, these are empty on OS X by default, so they rarely matter. 

Sometimes, we want to install a third-party library to a location, which is not system-defined, not */usr/local/lib* nor */usr/lib*, for some personal reasons or when you did not have root privilege. Suppose it was *libdummy*. If *libdummy*'s **install_name** was just libdummy.1.dylib or so, and you were building a program which links against libdummy. After the compilation, you checked the shared libraries your program used:

    otool -L program

then you could see libdummy.1.dylib in the output, just *libdummy.1.dylib* in that line. Ja, the linker stored *install_name* there, not the location of the library.
Then you let the program run, but the `dyld` said it could not find the proper libdummy.1.dylib. That's the story a lot of people would experience on OS X.

On Linux, we could modify */etc/ld.so.conf* in order to include some other directories when the loader searches for a library. However, on OS X things are different. Also, we would not like to set *DYLD\_* variables every time launching the program, nor add those variables in *.zshrc*, *.bashrc*, ...

De facto, as we know **install\_name** matters, we could simply employ it.

On Darwin platform, `gcc` has some platform dependent options, such as `-dynamic`, `-arch`, `-bundle`, and `-install_name` plays an important role here.

    gcc -o libdummy.dylib -install_name ${PREFIX}/lib/libdummy.dylib ...

would set **install\_name** for libdummy.dylib to a well-defined path. Next time when linking your program against libdummy, the linker would store that path. Use `otool -D` to print the **install\_nam** for specified library.

Besides absolute paths, we could use other techniques as well: 
**@executable\_path**, **@loader\_path**, **@rpath**. This [article](https://wincent.com/wiki/@executable_path,_@load_path_and_@rpath) describes these very well.

For already built libraries and programs, there is no need to rebuild them. On OS X, there is a very useful tool: `install_name_tool`.

- change **install\_name** for a library:
    `install_name_tool -id "new_install_name" libdummy.dylib`
- change linked **install\_name** in a program:
    `install_name_tool -change "old_install_name" "new_install_name" program`


One more thing. If you use `cmake` to generate Makefile for you project, you could solve the **install_name** issue like [this](http://www.cmake.org/pipermail/cmake/2011-April/043826.html):

    SET(CMAKE_INSTALL_NAME_DIR @executable_path)

Replace *@executable\_path* with your own choice.
