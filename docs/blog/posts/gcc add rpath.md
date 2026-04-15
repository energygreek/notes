---
draft: false
comments: true
date: 2026-02-06
tags: [gcc,rpath]
---

# add rpath in cmake and make project
This doc shows how to add rpath for cmake and make
<!-- more -->

## problem

```sh
ldd ~/.local/bin/baresip
  linux-vdso.so.1 (0x0000700da349c000)
  libbaresip.so.25 => not found
  libre.so.41 => not found
  libresolv.so.2 => /usr/lib/libresolv.so.2 (0x0000700da344d000)
  libz.so.1 => /usr/lib/libz.so.1 (0x0000700da3434000)
  libssl.so.3 => /usr/lib/libssl.so.3 (0x0000700da334d000)
  libcrypto.so.3 => /usr/lib/libcrypto.so.3 (0x0000700da2c00000)
  libm.so.6 => /usr/lib/libm.so.6 (0x0000700da324d000)
  libc.so.6 => /usr/lib/libc.so.6 (0x0000700da2a0f000)
  /lib64/ld-linux-x86-64.so.2 => /usr/lib64/ld-linux-x86-64.so.2 (0x0000700da349e000)
```

check rpath
```sh
readelf -d  ~/.local/bin/baresip

Dynamic section at offset 0x3d60 contains 33 entries:
  Tag        Type                         Name/Value
 0x0000000000000001 (NEEDED)             Shared library: [libbaresip.so.25]
 0x0000000000000001 (NEEDED)             Shared library: [libre.so.41]
 0x0000000000000001 (NEEDED)             Shared library: [libresolv.so.2]
 0x0000000000000001 (NEEDED)             Shared library: [libz.so.1]
 0x0000000000000001 (NEEDED)             Shared library: [libssl.so.3]
 0x0000000000000001 (NEEDED)             Shared library: [libcrypto.so.3]
 0x0000000000000001 (NEEDED)             Shared library: [libm.so.6]
 0x0000000000000001 (NEEDED)             Shared library: [libc.so.6]
 0x000000000000000c (INIT)               0x2000
 0x000000000000000d (FINI)               0x2e18
 0x0000000000000019 (INIT_ARRAY)         0x4d50
 0x000000000000001b (INIT_ARRAYSZ)       8 (bytes)
 0x000000000000001a (FINI_ARRAY)         0x4d58
 0x000000000000001c (FINI_ARRAYSZ)       8 (bytes)
 0x000000006ffffef5 (GNU_HASH)           0x3d0
```


## modify CMakelist.txt

set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

## validate

```sh
readelf -d  ~/.local/bin/baresip

Dynamic section at offset 0x3d60 contains 34 entries:
  Tag        Type                         Name/Value
 0x0000000000000001 (NEEDED)             Shared library: [libbaresip.so.25]
 0x0000000000000001 (NEEDED)             Shared library: [libre.so.41]
 0x0000000000000001 (NEEDED)             Shared library: [libresolv.so.2]
 0x0000000000000001 (NEEDED)             Shared library: [libz.so.1]
 0x0000000000000001 (NEEDED)             Shared library: [libssl.so.3]
 0x0000000000000001 (NEEDED)             Shared library: [libcrypto.so.3]
 0x0000000000000001 (NEEDED)             Shared library: [libm.so.6]
 0x0000000000000001 (NEEDED)             Shared library: [libc.so.6]
 0x000000000000001d (RUNPATH)            Library runpath: [/home/hst/.local/lib]
```

```sh
ldd ~/.local/bin/baresip
  linux-vdso.so.1 (0x000077faa818d000)
  libbaresip.so.25 => /home/hst/.local/lib/libbaresip.so.25 (0x000077faa8115000)
  libre.so.41 => /home/hst/.local/lib/libre.so.41 (0x000077faa8034000)
  libresolv.so.2 => /usr/lib/libresolv.so.2 (0x000077faa7ff1000)
  libz.so.1 => /usr/lib/libz.so.1 (0x000077faa7fd8000)
  libssl.so.3 => /usr/lib/libssl.so.3 (0x000077faa7ef1000)
  libcrypto.so.3 => /usr/lib/libcrypto.so.3 (0x000077faa7800000)
  libm.so.6 => /usr/lib/libm.so.6 (0x000077faa7df1000)
  libc.so.6 => /usr/lib/libc.so.6 (0x000077faa760f000)
  /lib64/ld-linux-x86-64.so.2 => /usr/lib64/ld-linux-x86-64.so.2 (0x000077faa818f000)
```

## makefile 中的办法
LDFLAGS += -Wl,-rpath=\$$ORIGIN/lib

## reference

https://cmake.org/cmake/help/latest/prop_tgt/INSTALL_RPATH.html