# How to build the xPack GNU RISC-V Embedded GCC

## Introduction

This project includes the scripts and additional files required to
build and publish the
[xPack GNU RISC-V Embedded GCC](https://xpack.github.io/riscv-none-embed-gcc/) binaries.

The build scripts use the
[xPack Build Box (XBB)](https://github.com/xpack/xpack-build-box),
a set of elaborate build environments based on GCC 7.4 (Docker containers
for GNU/Linux and Windows or a custom folder for MacOS).

## Repository URLs

The build scripts use a set of local repositories, to accommodate the
small changes required by the `riscv-none-embed-` prefix:

- [xpack-dev-tools/riscv-binutils-gdb](https://github.com/xpack-dev-tools/riscv-binutils-gdb)
- [xpack-dev-tools/riscv-gcc](https://github.com/xpack-dev-tools/riscv-gcc)
- [xpack-dev-tools/riscv-newlib](https://github.com/xpack-dev-tools/riscv-newlib)

These repositories were forked from the SiFive repositories:

- [sifive/riscv-binutils-gdb](https://github.com/sifive/riscv-binutils-gdb)
- [sifive/riscv-gcc](https://github.com/sifive/riscv-gcc)
- [sifive/riscv-newlib](https://github.com/sifive/riscv-newlib)

GDB was upstreamed and does not require SiFive specific patches, so the
original FSF repository is used:

- `git://sourceware.org/git/binutils-gdb.git`

However, to accommodate the custom prefix, a separate branch is created
in `xpack-dev-tools/riscv-binutils-gdb`.

## Download the build scripts repo

The build scripts are available in the `scripts` folder of the
[`xpack-dev-tools/riscv-none-embed-gcc-xpack`](https://github.com/xpack-dev-tools/riscv-none-embed-gcc-xpack)
Git repo.

To download them, the following shortcut is available:

```console
$ curl --fail -L https://github.com/xpack-dev-tools/riscv-none-embed-gcc-xpack/raw/xpack/scripts/git-clone.sh | bash
```

This small script issues the following two commands:

```console
$ rm -rf ~/Downloads/riscv-none-embed-gcc-xpack.git
$ git clone --recurse-submodules https://github.com/xpack-dev-tools/riscv-none-embed-gcc-xpack.git \
  ~/Downloads/riscv-none-embed-gcc-xpack.git
```

> Note: the repository uses submodules; for a successful build it is
> mandatory to recurse the submodules.

## The `Work` folder

The script creates a temporary build `Work/riscv-none-embed-gcc-${version}`
folder in the user home. Although not recommended, if for any reasons
you need to change the location of the `Work` folder,
you can redefine `WORK_FOLDER_PATH` variable before invoking the script.

## Customizations

There are many other settings that can be redefined via
environment variables. If necessary,
place them in a file and pass it via `--env-file`. This file is
either passed to Docker or sourced to shell. The Docker syntax
**is not** identical to shell, so some files may
not be accepted by bash.

### Prerequisites

The prerequisites are common to all binary builds. Please follow the
instructions from the separate
[Prerequisites for building xPack binaries](https://xpack.github.io/xbb/prerequisites/)
page and return when ready.

## Update git repos

The xPack GNU RISC-V Embedded GCC distribution follows the official
[SiFive](https://github.com/sifive/freedom-tools)
releases, and it is planned to make a new release after each future
SiFive release.

## Prepare release

### Check `README.md`

Normally `README.md` should not need changes, but better check.
Information related to the new version should not be included here,
but in the version specific file (below).

### Identify SiFive commit IDs

- check the [SiFive Releases](https://github.com/sifive/freedom-tools/releases)
for new releases
- identify the tag of the latest release (like `v2019.08.0`)
- switch to Code view and select the tag
- open the `src` folder to identify the commit IDs used for the linked repos.

### Identify the main GCC version

Determine the GCC version (like `8.3.0`) and update the `scripts/VERSION`
  file; the format is `8.3.0-1.1`. The fourth digit is the number of the
  SiFive release of the same GCC version, and the fifth digit is the xPack
  GNU RISC-V Embedded GCC release number of this version.

### Create `README-<version>.md`

In the `scripts` folder create a copy of the previous version one.

From the the `src` folder, follow the linked module links for 
binutils/gcc/newlib and copy/paste them.
Also update the short IDs and dates.

Check the GDB commit ID (not linked, since it points to external repo).

### Update binutils

With Sourcetree in
[xpack-dev-tools/riscv-binutils-gdb](https://github.com/xpack-dev-tools/riscv-binutils-gdb)

Check if there is a `sifive` remote pointing to
https://github.com/sifive/riscv-binutils-gdb.

If the changes apply to an existing branch:

- checkout the current SiFive branch (like `sifive-binutils-2.32`)
- merge the commit with the desired ID from the corresponding branch in
the `sifive` remote
- checkout the corresponding xpack branch (like `sifive-binutils-2.32-xpack`)
- merge the previous merge
- check the differences from the non-xpack branch; it should be only the
addition of `embed)` in `config.sub`

- from the `sifive` remote
checkout the new SiFive branch (like `sifive-binutils-2.32`) into a new local
branch
- identify the commit ID and switch to it
- create a new branch with a similar name, but suffixed by `-xpack`
(like `sifive-binutils-2.32-xpack`)
- identify the commit which adds the xPack specific changes
- cherry pick it; do not commit immediately
- check the uncommited changes; there should be one file `config.sub` 
which adds `-embed)`
- commit as **add support for riscv-none-embed-***

In both cases:

- push the two modified branches (like `sifive-binutils-2.32` and
`sifive-binutils-2.32-xpack`)
- add a tag with the current version (like `v8.3.0-1.1`), and push 
it to `origin`.

### Update gcc

With Sourcetree in
[xpack-dev-tools/riscv-gcc](https://github.com/xpack-dev-tools/riscv-gcc)

Check if there is a `sifive` remote pointing to
https://github.com/sifive/riscv-gcc.

If the changes apply to an existing branch:

- checkout the current SiFive branch (like `sifive-gcc-8.3.0`)
- merge the commit with the desired ID from the corresponding branch in
the `sifive` remote
- checkout the corresponding xpack branch (like `sifive-gcc-8.3.0-xpack`)
- merge the previous merge
- check the differences from the non-xpack branch; there should be tree files:
  - `elf-embed.h` with the `LIB_SPEC` definitions without libgloss
  - `config.gcc` with `tm_file` definition that uses `elf-embed.h`
  - `config.sub` which adds `riscv0*)` and `-embed)`

If this is a new branch:

- from the `sifive` remote
checkout the new SiFive branch (like `sifive-gcc-8.3.0`) into a new local
branch
- identify the commit ID and switch to it
- create a new branch with a similar name, but suffixed by `-xpack` 
(like `sifive-gcc-8.3.0-xpack`)
- identify the commit which adds the xPaack specific changes
- cherry pick it; do not commit immediately
- check the uncommited changes; there should be tree files:
  - `elf-embed.h` with the `LIB_SPEC` definitions without libgloss
  - `config.gcc` with `tm_file` definition that uses `elf-embed.h`
  - `config.sub` which adds `riscv0*)` and `-embed)`
- commit as **add support for riscv-none-embed-***

In both cases:

- push the two modified branches (like `sifive-gcc-8.3.0` and
`sifive-gcc-8.3.0-xpack`)
- add a tag with the current version (like `v8.3.0-1.1`), and push
it to `origin`.

### Update newlib

With Sourcetree in
[xpack-dev-tools/riscv-newlib](https://github.com/xpack-dev-tools/riscv-newlib)

Check if there is a `sifive` remote pointing to
https://github.com/sifive/riscv-newlib.

If the changes apply to an existing branch:

- checkout the current SiFive branch (like `master`)
- merge the commit with the desired ID from the corresponding branch in
the `sifive` remote
- checkout the corresponding xpack branch (like `sifive-binutils-2.32-xpack`)
- merge the previous merge
- check the differences from the non-xpack branch; it should be only the
addition of `embed)` in `config.sub`

- from the `sifive` remote
checkout the new SiFive branch (like `master`) into a new local
branch (like `sifive-master`)
- identify the commit ID and switch to it
- create a new branch with a similar name, but suffixed by `-xpack`
(like `sifive-master-xpack`)
- identify the commit which adds the xPack specific changes
- cherry pick it; do not commit immediately
- check the uncommited changes; there should be one file `config.sub` 
which adds `-embed)`
- commit as **add support for riscv-none-embed-***

In both cases:

- push the two modified branches (like `sifive--master` and
`sifive--master-xpack`)
- add a tag with the current version (like `v8.3.0-1.1`), and push 
it to `origin`.

### Update gdb

With Sourcetree in
[xpack-dev-tools/riscv-binutils-gdb](https://github.com/xpack-dev-tools/riscv-binutils-gdb)

Check if there is a `gdb` remote pointing to
git://sourceware.org/git/binutils-gdb.git.

For GDB, it is a bit tricky, since it must identify the GNU code in line
with what was used by SiFive; create a branch like `sifive-gdb-8.3`

- branch it into `sifive-gdb-8.3-xpack` and edit the prefix code
- add a tag like `v8.3.0-1.1-gdb`

### Update container-build.sh

- add a new set of definitions in the `scripts/container-build.sh`, with
  the versions of various components;
- update `GH_RELEASE` to the new version (like `8.3.0-1.1`, without `v`)
- in [SiFive Releases](https://github.com/sifive/freedom-tools/releases)
for new releases
- identify the tag of the latest release (like `v2019.08.0`)
- switch to Code view and select the tag
- open the `Makefile` file to identify the `MULTILIBS_GEN` definition;
- copy/paste it into `GCC_MULTILIB`
- add `rv32imaf-ilp32f--` after `rv32imf-`

## Update `CHANGELOG.md`

Check `CHANGELOG.md` and add the new release.

## Build

Although it is perfectly possible to build all binaries in a single step
on a macOS system, due to Docker specifics, it is faster to build the
GNU/Linux and Windows binaries on a GNU/Linux system and the macOS binary
separately.

### Build the GNU/Linux and Windows binaries

The current platform for GNU/Linux and Windows production builds is an
Ubuntu Server 18 LTS, running on an Intel NUC8i7BEH mini PC with 32 GB of RAM
and 512 GB of fast M.2 SSD.

```console
$ ssh ilg-xbb-linux.local
```

Before starting a multi-platform build, check if Docker is started:

```console
$ docker info
```

Eventually run the test image:

```console
$ docker run hello-world
```

Before running a build for the first time, it is recommended to preload the
docker images.

```console
$ bash ~/Downloads/riscv-none-embed-gcc-xpack.git/scripts/build.sh preload-images
```

The result should look similar to:

```console
$ docker images
REPOSITORY TAG IMAGE ID CREATED SIZE
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ilegeul/centos32    6-xbb-v2.2          956eb2963946        5 weeks ago         3.03GB
ilegeul/centos      6-xbb-v2.2          6b1234f2ac44        5 weeks ago         3.12GB
hello-world         latest              fce289e99eb9        5 months ago        1.84kB
```

It is also recommended to Remove unused Docker space. This is mostly useful
after failed builds, during development, when dangling images may be left
by Docker.

To check the content of a Docker image:

```console
$ docker run --interactive --tty ilegeul/centos:6-xbb-v2.2
```

To remove unused files:

```console
$ docker system prune --force
```

To download the build scripts:

```console
$ curl --fail -L https://github.com/xpack-dev-tools/riscv-none-embed-gcc-xpack/raw/xpack/scripts/git-clone.sh | bash
```

To build both the 32/64-bit Windows and GNU/Linux versions, use `--all`; to
build selectively, use `--linux64 --win64` or `--linux32 --win32` (GNU/Linux
can be built alone; Windows also requires the GNU/Linux build).

Since the build takes a while, use `screen` to isolate the build session
from unexpected events, like a broken
network connection or a computer entering sleep.

```console
$ screen -S riscv

$ sudo rm -rf ~/Work/riscv-none-embed-gcc-*
$ bash ~/Downloads/riscv-none-embed-gcc-xpack.git/scripts/build.sh --all --jobs 4
```

To detach from the session, use `Ctrl-a` `Ctrl-d`; to reattach use
`screen -r riscv`; to kill the session use `Ctrl-a` `Ctrl-\` and confirm.

Several hours later, the output of the build script is a set of 4 files and
their SHA signatures, created in the `deploy` folder:

```console
$ ls -l deploy
total 350108
-rw-r--r-- 1 ilg ilg  61981364 Apr  1 08:27 xpack-riscv-none-embed-gcc-8.2.0-3.1-linux-x32.tar.xz
-rw-r--r-- 1 ilg ilg       140 Apr  1 08:27 xpack-riscv-none-embed-gcc-8.2.0-3.1-linux-x32.tar.xz.sha
-rw-r--r-- 1 ilg ilg  61144048 Apr  1 08:19 xpack-riscv-none-embed-gcc-8.2.0-3.1-linux-x64.tar.xz
-rw-r--r-- 1 ilg ilg       140 Apr  1 08:19 xpack-riscv-none-embed-gcc-8.2.0-3.1-linux-x64.tar.xz.sha
-rw-r--r-- 1 ilg ilg 112105889 Apr  1 08:29 xpack-riscv-none-embed-gcc-8.2.0-3.1-win32-x32.zip
-rw-r--r-- 1 ilg ilg       134 Apr  1 08:29 xpack-riscv-none-embed-gcc-8.2.0-3.1-win32-x32.zip.sha
-rw-r--r-- 1 ilg ilg 123181226 Apr  1 08:21 xpack-riscv-none-embed-gcc-8.2.0-3.1-win32-x64.zip
-rw-r--r-- 1 ilg ilg       134 Apr  1 08:21 xpack-riscv-none-embed-gcc-8.2.0-3.1-win32-x64.zip.sha
```

To copy the files from the build machine to the current development
machine, either use NFS to mount the entire folder, or open the `deploy`
folder in a terminal and use `scp`:

```console
$ cd ~/Work/riscv-none-embed-gcc-*/deploy
$ scp * ilg@ilg-wks.local:Downloads/xpack-binaries/riscv
```

### Build the macOS binary

The current platform for macOS production builds is a macOS 10.10.5
VirtualBox image running on the same macMini with 16 GB of RAM and a
fast SSD.

```console
$ ssh ilg-xbb-mac.local
```

To download them, the following shortcut is available:

```console
$ curl --fail -L https://github.com/xpack-dev-tools/riscv-none-embed-gcc-xpack/raw/xpack/scripts/git-clone.sh | bash
```

To build the latest macOS version:

```console
$ screen -S arm

$ sudo rm -rf ~/Work/riscv-none-embed-gcc-*
$ caffeinate bash ~/Downloads/riscv-none-embed-gcc-xpack.git/scripts/build.sh --osx --jobs 8
```

To detach from the session, use `Ctrl-a` `Ctrl-d`; to reattach use
`screen -r openocd`; to kill the session use `Ctrl-a` `Ctrl-k` and confirm.

Several hours later, the output of the build script is a compressed archive
and its SHA signature, created in the `deploy` folder:

```console
$ ls -l deploy
total 216064
-rw-r--r--  1 ilg  staff  110620198 Jul 24 16:35 xpack-riscv-none-embed-gcc-8.2.0-3.1-darwin-x64.tgz
-rw-r--r--  1 ilg  staff        134 Jul 24 16:35 xpack-riscv-none-embed-gcc-8.2.0-3.1-darwin-x64.tgz.sha
```

To copy the files from the build machine to the current development
machine, either use NFS to mount the entire folder, or open the `deploy`
folder in a terminal and use `scp`:

```console
$ cd ~/Work/riscv-none-embed-gcc-*/deploy
$ scp * ilg@ilg-wks.local:Downloads/xpack-binaries/riscv
```

## Run a test build on macOS

Before starting the builds on the dedicated machines, run a quick test on
the local development workstation.

```console
$ caffeinate bash ~/Downloads/riscv-none-embed-gcc-xpack.git/scripts/build.sh --osx --disable-multilib --develop --jobs 12
```

This should check the commit IDs and the tag names in all the refered
repositories, and the build scripts.

It is _quick_ because it does not build the multilibs. Even so, on a
fast machine, it takes about one hour.

## Subsequent runs

### Separate platform specific builds

Instead of `--all`, you can use any combination of:

```
--win32 --win64 --linux32 --linux64
```

Please note that, due to the specifics of the GCC build process, the
Windows build requires the corresponding GNU/Linux build, so `--win32`
alone is equivalent to `--linux32 --win32` and `--win64` alone is
equivalent to `--linux64 --win64`.

### clean

To remove most build files, use:

```console
$ bash ~/Downloads/riscv-none-embed-gcc-xpack.git/scripts/build.sh clean
```

To also remove the repository and the output files, use:

```console
$ bash ~/Downloads/riscv-none-embed-gcc-xpack.git/scripts/build.sh cleanall
```

For production builds it is recommended to completely remove the build folder.

### --develop

For performance reasons, the actual build folders are internal to each
Docker run, and are not persistent. This gives the best speed, but has
the disadvantage that interrupted builds cannot be resumed.

For development builds, it is possible to define the build folders in the
host file system, and resume an interrupted build.

### --debug

For development builds, it is also possible to create everything
with `-g -O0` and be able to run debug sessions.

### --disable-multilib

For development builds, to save time, it is recommended to build the
toolchain without multilib.

### --jobs

By default, the build steps use a single job at a time, but for
recent CPUs with multiple cores it is possible to run multiple jobs
in parallel.

The setting applies to all steps.

Warning: Parallel builds require significant system resources and occasionally
may crash the build.

### Interrupted builds

The Docker scripts run with root privileges. This is generally not a
problem, since at the end of the script the output files are reassigned
to the actual user.

However, for an interrupted build, this step is skipped, and files in
the install folder will remain owned by root. Thus, before removing the
build folder, it might be necessary to run a recursive `chown`.

### Rebuild previous releases

Set the release explicitly in the environment:

```console
$ RELEASE_VERSION=8.2.0-3.1 bash ~/Downloads/riscv-none-embed-gcc-xpack.git/scripts/build.sh --all --jobs 8
$ RELEASE_VERSION=8.3.0-1.1 bash ~/Downloads/riscv-none-embed-gcc-xpack.git/scripts/build.sh --all --jobs 8
```

> Note: This procedure builds a previous release but in the context of
the latest build environment, which might be different from the one used
at the time of the release, so the result may be slightly different; to
obtain exactly the same result, use the commit tagged with the desired
release.

## Test

A simple test is performed by the script at the end, by launching the
executables to check if all shared/dynamic libraries are correctly used.

For a true test you need to unpack the archive in a temporary location
(like `~/Downloads`) and then run the
program from there. For example on macOS the output should
look like:

```console
$ /Users/ilg/Downloads/xPacks/riscv-none-embed-gcc/8.2.0-3.1/bin/riscv-none-embed-gcc --version
riscv-none-embed-gcc (xPack RISC-V Embedded GCC, 64-bit) 8.2.0
```

## Install

The procedure to install xPack GNU RISC-V Embedded GCC is platform
specific, but relatively straight forward (a .zip archive on Windows,
a compressed tar archive on macOS and GNU/Linux).

A portable method is to use [`xpm`](https://www.npmjs.com/package/xpm):

```console
$ xpm install --global @xpack-dev-tools/riscv-none-embed-gcc@latest
```

More details are available on the
[How to install the RISC-V toolchain?](https://xpack.github.io/riscv-none-embed-gcc/install/)
page.

After install, the package should create a structure like this (only the
first two depth levels are shown):

```console
$ tree -L 2 /Users/ilg/opt/xPacks/riscv-none-embed-gcc/8.2.1-1.1
/Users/ilg/opt/gnu-mcu-eclipse/riscv-none-embed-gcc/8.2.1-1.1
├── README.md
├── riscv-none-embed
│   ├── bin
│   ├── include
│   ├── lib
│   └── share
├── bin
│   ├── riscv-none-embed-addr2line
│   ├── riscv-none-embed-ar
│   ├── riscv-none-embed-as
│   ├── riscv-none-embed-c++
│   ├── riscv-none-embed-c++filt
│   ├── riscv-none-embed-cpp
│   ├── riscv-none-embed-elfedit
│   ├── riscv-none-embed-g++
│   ├── riscv-none-embed-gcc
│   ├── riscv-none-embed-gcc-8.2.1
│   ├── riscv-none-embed-gcc-ar
│   ├── riscv-none-embed-gcc-nm
│   ├── riscv-none-embed-gcc-ranlib
│   ├── riscv-none-embed-gcov
│   ├── riscv-none-embed-gcov-dump
│   ├── riscv-none-embed-gcov-tool
│   ├── riscv-none-embed-gdb
│   ├── riscv-none-embed-gdb-add-index
│   ├── riscv-none-embed-gdb-add-index-py
│   ├── riscv-none-embed-gdb-py
│   ├── riscv-none-embed-gprof
│   ├── riscv-none-embed-ld
│   ├── riscv-none-embed-ld.bfd
│   ├── riscv-none-embed-nm
│   ├── riscv-none-embed-objcopy
│   ├── riscv-none-embed-objdump
│   ├── riscv-none-embed-ranlib
│   ├── riscv-none-embed-readelf
│   ├── riscv-none-embed-size
│   ├── riscv-none-embed-strings
│   └── riscv-none-embed-strip
├── distro-info
│   ├── CHANGELOG.md
│   ├── licenses
│   ├── patches
│   └── scripts
├── include
│   └── gdb
├── lib
│   ├── bfd-plugins
│   ├── gcc
│   ├── libcc1.0.so
│   └── libcc1.so -> libcc1.0.so
├── libexec
│   └── gcc
└── share
    ├── doc
    └── gcc-riscv-none-embed

19 directories, 37 files
```

No other files are installed in any system folders or other locations.

## Uninstall

The binaries are distributed as portable archives; thus they do not
need to run a setup and do not require an uninstall.

## Files cache

The XBB build scripts use a local cache such that files are downloaded only
during the first run, later runs being able to use the cached files.

However, occasionally some servers may not be available, and the builds
may fail.

The workaround is to manually download the files from an alternate
location (like
https://github.com/xpack-dev-tools/files-cache/tree/master/libs),
place them in the XBB cache (`Work/cache`) and restart the build.

## Pitfalls

### Parallel build

Parallel builds are not only supported, but encouraged. Use a number of
jobs based on the number of cores, preferably without counting the 
hyperthreads.

Please note that, for optimum performances, parallel builds require ample
amounts of RAM (8 GB are know to work, 16 GB are recommended for 4 jobs).

### Building GDB on macOS

GDB uses a complex and custom logic to unwind the stack when processing
exceptions; macOS also uses a custom logic to organize memory and process
exceptions; the result is that when compiling GDB with GCC on older macOS
systems (like 10.10), some details do not match and the resulting GDB
crashes with an assertion on the first `set language` command (most
probably many other commands).

The workaround was to compile GDB with Apple clang, which resulted in
functional binaries, even on the old macOS 10.10.

## More build details

The build process is split into several scripts. The build starts on the
host, with `build.sh`, which runs `container-build.sh` several times,
once for each target, in one of the two docker containers. Both scripts
include several other helper scripts. The entire process is quite complex,
and an attempt to explain its functionality in a few words would not
be realistic. Thus, the authoritative source of details remains the source
code.
