---
layout: post
title:  "Compile FastDownward on Mac OSX"
categories: AI
tags: classical-planning, fast-downward
---

[FD] as a platform stands out in [IPC-2014] as "29 planners out of 67 built on top of [FD]".

So I decided to try it out. However, the document seems out of date for compiling the codebase on Mac OSX. Here is my process to get it compiled on my laptop Mac OSX 10.10.4.

First, we need GNU C++ compiler.
I installed it from Homebrew (the latest version you can install with Homebrew is gcc 4.9, but [FD] document mentioned gcc 4.8, so I tried it out first):

    brew install homebrew/versions/gcc48

Then we need to setup command line to use GNU gcc instead of Apple's gcc compiler (I have clang installed). I also need a clean way that won't mass up my existing XCode projects, so I created a file gnu-gcc-aliases with the following aliases:

    alias gcc='gcc-4.8'
    alias cc='gcc-4.8'
    alias g++='g++-4.8'
    alias c++='c++-4.8

Then source the file in terminal:

    source gnu-gcc-aliases

Next step, get codebase. I checked out from here: https://github.com/danfis/fast-downward, as I don't want to install hg in my laptop.
Then goto src directory in the codebase, and run:

    ./build_all

It failed on my laptop with the following error:


    utilities.cc:174:12: error: use of undeclared identifier 'getpid'
        return getpid();
               ^
    1 error generated.
    make: *** [.obj/utilities.release.o] Error 1

Did Google with the error. It seems missing `#include <unistd.h>` somewhere. So I openned file utilities.cc, and found the following code:

     #if OPERATING_SYSTEM == LINUX
     #include <unistd.h>
     static void exit_handler(int exit_code, void *hint);
     #elif OPERATING_SYSTEM == OSX

Supprise! `#include <unistd.h>` is only included for compiling on Linux. Change it to:

     #include <unistd.h>
     #if OPERATING_SYSTEM == LINUX
     static void exit_handler(int exit_code, void *hint);
     #elif OPERATING_SYSTEM == OSX

Then run `./build_all` again. Compile successed.

Tested with some PDDL domain and problems, everything seems good.
Also successfully built with 64bit mode.

     ./build_all DOWNWARD_BITWIDTH=64


[FD]:              http://www.fast-downward.org/
[previous post]:   /ai/2015/07/05/classical-planning-3-international-planning-competition.html
[IPC-2014]:        https://helios.hud.ac.uk/scommv/IPC-14/index.html
