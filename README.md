WHAT
====

This is a project that collects together several Open Source sub-projects from the Darwin (OS X) opensource repository (http://www.opensource.apple.com/darwinsource/)

**NOTE**
  This is code has been modified, although based on the posted sources.
  problem reports should be filed to **https://github.com/iains/darwin-xtools/issues**
  **Please do not** file problem reports for this code to Apple.

The sub-projects are
 cctools ; providing as, nm, ar, ranlib, strip, etc.
 ld64 ; providing the static linker and some test utils.
 dyld ; actually only used for headers
 several stub libraries used when support is needed on older systems.

The principal changes common to all versions are:

1. Build with CMAKE
 - I wanted to avoid a clash with the build stuff contained in the original sources
 - This is the same approach as used for other Darwin/OS X toolchain content (i.e. LLVM)

2. Support for PPC/PPC64.
 - This is a heavily modified merge of the XCode 3.2.6 ppc support + my own additions to re-introduce ppc64.
 - The branch islanding code has been almost completely re-written and it now supports multiple code sections (which GCC emits) and works for larger binaries (e.g. a Debug version of clang).

3. Bug-fixes for GCC-support.

4. Support for the content that LLVM/Clang emit on earlier Darwin.
 - XCode tools <= 3.2.6 do not support the macosx-version-min load command in "ar, ranlib" etc.
 - This means Darwin9/10 (OS X 10.5/6) need updated tools to handle LLVM.

RELEASE 1.0.0
=============

This is darwin-xtools 1.0.0

It has been built and tested (using gcc-5.3 and/or XCode clang, where that supports c++11) on:
  i686-apple-darwin9, powerpc-apple-darwin9, i686-apple-darwin10
  x86-64-apple-darwin{10,11,12,13,14}

NOTE: this is not really necessary on Darwin14+, since XCode 7.2 contains more recent tools; you might want some of the GCC compatibility features, but GCC also builds with XC7.2.

Changes to cctools (877.5):
 - as; **Default to using cctools 'as' for all targets** this means that you don't need to have a clang install in order to proceed (and is somewhat more GCC-compatible on most Darwin targets).
 - as; Fix ULEB128 handling where the MSB is set.
 - as; Allow SLEB128 to have 64b values on 32b hosts (needed for GCC support)
 - as; Fix .zerofill to handle up to 0xffffffff on 32b hosts and to check for out-of-range situations.
 - as; Restore PPC relocations.
 - general; cater for building with/without LTO support.
 - general; add --version, --help, --target-help where it helps with GCC compatibilty.
 - general; add a bugURL.
 - general; Various build fixes and sim libs for different Darwin versions (see commmit logs)

Changes to ld64 (253.3):
 - Add PPC/PPC64 support
 - Revised branch islands algorithm for PPC.
 - general; add --version, --help, --target-help where it helps with GCC compatibilty.
 - Various build fixes (see commit log)
 
PRE-REQUISITES
==============

A. cmake - I've been using 3.4.1 without issue on Darwin9...Darwin14 (OS X 10.5 +)

B. **A C++11 compiler**
 - on Darwin9, 10 and probably 11 that means building a compiler...
 - GCC-5.3 (https://github.com/iains/darwin-gcc-5) works fine and can be built using the default XCode toolchain.

HOW TO BUILD
============

1. Checkout the relevant composite branch.

2. I usually use a script file like this:

cmake -C /path/to/script/file.cmake /path/to/source

Example for 10.6 -- you will need to modify this as relevant to your case

darwin-xtools-10-6.cmake:
====== 8< =======::
    SET(CMAKE_INSTALL_PREFIX /where/you/want/it CACHE PATH "put it here")
    SET(CMAKE_BUILD_TYPE MinSizeRel CACHE STRING "build style")

    SET(CMAKE_OSX_DEPLOYMENT_TARGET "10.6" CACHE STRING "OS X Version")
    set(SDK_BASE /Developer/SDKs)
    SET(CMAKE_OSX_SYSROOT ${SDK_BASE}/MacOSX10.6.sdk CACHE PATH "system SDK 2")
    SET(LLVM_DEFAULT_TARGET_TRIPLE x86_64-apple-darwin10 CACHE STRING "system triple")

    SET(compilers /path/to/c++11/compiler/bin)
    SET(CMAKE_C_COMPILER   ${compilers}/gcc CACHE PATH "C")
    SET(CMAKE_CXX_COMPILER ${compilers}/g++ CACHE PATH "C++")

    SET(CMAKE_CXX_FLAGS  "-mmacosx-version-min=10.6 -pipe" CACHE STRING "cxx flags")
    SET(CMAKE_C_FLAGS_MINSIZEREL  "-Os" CACHE STRING "c   opt flags")
    SET(CMAKE_CXX_FLAGS_MINSIZEREL "-Os" CACHE STRING "cxx opt flags")

    # If we're building with my GCC, then avoid carrying around a shared lib.
    set(CMAKE_EXE_LINKER_FLAGS "-static-libstdc++ " CACHE STRING "toolchain exe ldflags")
    set(CMAKE_SHARED_LINKER_FLAGS "-static-libstdc++ " CACHE STRING "toolchain shlib ldflags")

    set(XTOOLS_HOST_IS_64B YES CACHE BOOL "host native bitwidth")
    set(XTOOLS_BUGURL "https://githb.com/iains/darwin-xtools/issues" CACHE STRING "bug url")
====== >8 =======

You can set these things on the cmake invocation line like::

    cmake -DCMAKE_C_COMPILER=/path/to/compiler ... etc.

See the cmake documentation for the version you're using.

3. make, make install.
 - at present there's not a usable test-suite (although using the resulting tools to build GCC/LLVM is a fairly good test).
