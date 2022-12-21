---
layout: post
title:  "Understanding DWARF - Variables"
tags: [debug, DWARF]
---
If you are involved in developing a debugger you need to understand at least some basics of the [DWARF](https://dwarfstd.org) standard.
During the build process a toolchain generates debug information to be used later in the debug phase. This information consists on names, positions, size of variables and functions, etc etc. Anything needed to identifying what every bit of a running program.
This information is put in the final ELF file, and has a standard format, whose name is DWARF.
DWARF is not the only one, there are others. But in AURIX world you will be dealing only with DWARF.



The following steps assume MSYS2 is already installed and working.
To build [libdwarf][libdwarf] for usage with Windows, I performed the following steps:

- Open a MSYS2 MINGW64 session
- Install all required dependencies: 

        pacman -S mingw-w64-x86_64-binutils mingw-w64-x86_64-brotli mingw-w64-x86_64-bzip2 mingw-w64-x86_64-c-ares mingw-w64-x86_64-ca-certificates mingw-w64-x86_64-cmake mingw-w64-x86_64-crt-git mingw-w64-x86_64-curl mingw-w64-x86_64-expat mingw-w64-x86_64-gcc mingw-w64-x86_64-gcc-ada mingw-w64-x86_64-gcc-fortran mingw-w64-x86_64-gcc-libgfortran mingw-w64-x86_64-gcc-libs mingw-w64-x86_64-gcc-objc mingw-w64-x86_64-gdb mingw-w64-x86_64-gdb-multiarch mingw-w64-x86_64-gettext mingw-w64-x86_64-gmp mingw-w64-x86_64-headers-git mingw-w64-x86_64-isl mingw-w64-x86_64-jansson mingw-w64-x86_64-jemalloc mingw-w64-x86_64-jsoncpp mingw-w64-x86_64-libarchive mingw-w64-x86_64-libb2 mingw-w64-x86_64-libffi mingw-w64-x86_64-libgccjit mingw-w64-x86_64-libiconv mingw-w64-x86_64-libidn2 mingw-w64-x86_64-libmangle-git mingw-w64-x86_64-libpsl mingw-w64-x86_64-libssh2 mingw-w64-x86_64-libsystre mingw-w64-x86_64-libtasn1 mingw-w64-x86_64-libtre-git mingw-w64-x86_64-libunistring mingw-w64-x86_64-libuv mingw-w64-x86_64-libwinpthread-git mingw-w64-x86_64-libxml2 mingw-w64-x86_64-lz4 mingw-w64-x86_64-make mingw-w64-x86_64-meson mingw-w64-x86_64-mpc mingw-w64-x86_64-mpdecimal mingw-w64-x86_64-mpfr mingw-w64-x86_64-ncurses mingw-w64-x86_64-nghttp2 mingw-w64-x86_64-ninja mingw-w64-x86_64-openssl mingw-w64-x86_64-p11-kit mingw-w64-x86_64-pkgconf mingw-w64-x86_64-python mingw-w64-x86_64-readline mingw-w64-x86_64-rhash mingw-w64-x86_64-sqlite3 mingw-w64-x86_64-tcl mingw-w64-x86_64-termcap mingw-w64-x86_64-tk mingw-w64-x86_64-tools-git mingw-w64-x86_64-tzdata mingw-w64-x86_64-windows-default-manifest mingw-w64-x86_64-winpthreads-git mingw-w64-x86_64-winstorecompat-git mingw-w64-x86_64-xxhash mingw-w64-x86_64-xz mingw-w64-x86_64-zlib mingw-w64-x86_64-zstd mingw-w64-x86_64-libelf

- Download a libdwarf release, and prepare the build directories

        rm -rf /tmp/build
        mkdir /tmp/build
        cd /tmp
        wget https://github.com/davea42/libdwarf-code/releases/download/v0.5.0/libdwarf-0.5.0.tar.xz
        tar -xvf libdwarf-0.5.0.tar.xz
        cd  /tmp/build

- Build

        meson /tmp/libdwarf-0.5.0
        ninja



[libdwarf]: https://github.com/davea42/libdwarf-code
