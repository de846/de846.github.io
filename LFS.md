# de846.github.io

# Linux From Scratch
I'm starting a Linux From Scratch project, something I've always wanted to do for years.
Today is Independence Day, July 4th, 2020. Here we go..

## Day 1 (7/4/2020)
I am using a MacBook Pro (Retina, 13-inch, Mid 2014) with 2.6GHz Dual-Core Intel Core i5 and 8 GB 1600 MHz DDR3. Yes, this is an older laptop but I loathe the newer keyboards, and this machine got me through so much.

Already running into problems.

I upgraded to macOS Catalina a while ago, but my VMWare Fusion Pro 10 has bugs on this version and I don't want to upgrade. So, I installed VirtualBox and got the latest CentOS 8 image to use a host for my LFS build.

Apparently, the network interface doesn't come up automatically. I'm a Debian guy, but want to take the opportunity to get used to RHEL/CentOS, so trying something new. I had to run
```
nmcli connection up enp0s3
```

Additionally, I had issues getting the VBox Guest Utils installed. I had to run `dnf update` first to get the latest kernel so I could get the matching kernel-headers package. Reboot, reinstall, all good.

### Prereq packages
Missing some packages. I had to install a few prerequisites, such as
- gcc-c++
- byacc
- texinfo (from PowerTools)
- bison
- m4
- patch

The versions don't necessarily match up to the ones in the LFS doc, so I hope that doesn't bite me later. If parameter names have changed, then I might need to manually downgrade some packages. We'll see later.

### Partition
I need a partition (and a swap is recommended), so I shut down the CentOS VM to create a new virtual disk and attach it. I'll create a 20 GB disk, with 2GB for swap.

I used gdisk to create a gpt partition, created a 10MB partition for Grub BIOS (ef02), a 2GB swap (8200) and ~18GB root filesystem (8300). I decided to not make separate partitions for /boot, /tmp, etc. to keep it simple on my first pass.

### LFS packages
From section 3.2 and 3.3, I copied the text and used awk to export the URLs need to create a list to pass into wget using:
```
 awk '/Download/{print $2}' wget-list > wget-list-cleaned
 wget --input-file=wget-list-cleaned --continue --directory-prefix=$LFS/sources
 md5sum -c md5sums
```
All of the checksums matches. So far, so good.

Creating tools directory so it can be removed later.
```
mkdir -v $LFS/tools
ln -sv $LFS/tools /
```

### Creating lfs user
We are creating a group `lfs` and a user `lfs`.
```
groupadd lfs
useradd -s /bin/bash -g lfs -m -k /dev/null lfs
```
The `-g lfs` option adds the user to the new group. `-m` says to make a homedir, with the skeleton defined by `-k`.

```
chown -v lfs $LFS/tools
chown -v lfs $LFS/sources
```

### Creating toolchain
I made it to chapter 5 and was reading about the toolchain. I'll have to go back to check some things about GCC and binutils, but it seems very interesting. I am unpacking all tarballs in `$LFS/sources` right now.

Taking a break today to prepare for a Leetcode contest.

## Day 2 (7/5/2020)

I unpacked some libraries needed to compile gcc in the wrong directory, so I was missing `gmp`, `mpfr`, and `mpc`. I re-read the instructions and let the gcc build take off.

The guest VM I am using (as the host for LFS) was spending too much room on gnome-shell, so I installed xfce and rebooted. I also decided to remove `gdm` and replace it with `sddm`.

We built binutils, then gcc (without c++ support intentionally) so that we could compile the sources for the C library (glibc-2.31). Now that we have a C library, we can compile libstdc++ so that we can recompile gcc with c++ support.

Now that we have our target compiler with C and C++ support in `/tools`, we can recompile binutils for pass-2 with our versions of `cc` and `ar` by setting `CC=$LFS_TGT-gcc` and `AR=$LFS_TGT-ar`.

## Day 3 (7/6/2020)

I most recently have been following the compilation instructions for packages and got to the point where I am compiling `tcl8.6.10` to run its test suite. Tcl itself is going to be useful to run other tests later to ensure everything in the toolchain built properly.

I got an error running `make` for the expect package. Something with the tcl package integration, since you need to specify `--with-tcl=/tools/lib` and `--with-tclinclude=/tools/include`, but I am seeing:
```
(echo 'if {![package vsatisfies [package provide Tcl] 8.6]} {return}' ; \
 echo 'package ifneeded Expect 5.45.4 \
    [list load [file join $dir libexpect5.45.4.so]]'\
) > pkgIndex.tcl
```
It looks like no one really runs tests in chapter 5. Perhaps the above message isn't a proper error. I was able to install DejaGNU without issues and it relied on Tcl8.6, so /shrug. Moving on to m4... and ncurses..

In 5.15, the instruction to `ln -s libncursesw.so /tools/lib/libncurses.so` fails because the source library is actually in the `lib/` subdirectory, so I used that one instead (of the non-existent one). Uh-oh - my compilation of Bash 5.0 failed because it couldn't find `-lcurses`. Time to dig...

## Day 4 (7/7/2020)

Still digging into this failing curses library. What is going on? I wonder if I forgot to do something when I rebooted the other day. I remounted the block device and the swap partition, the lfs user's shell is already properly configured. The investigation continues...

I tried reading through `strace make ...` for bash, but I didn't see anything relevant to curses. I was hoping to see an attempt to open libncurses files, but nothing.

I just spent 10 minutes reading through the `Bash-5.0/configure` script. Yikes. Apparently, it needs `curses` which is _not_ `ncurses`, so I am trying to recompile after running `ln -sv /tools/lib/libncursesw.so /tools/lib/libcurses.so`.

It worked! I can move on to compile `bison-3.5.2` next.

The bison tests took over 30 minutes to complete. I think it ran something like 564 tests. I went back to Leetcode while I waited for the tests to pass.

I've spent the last hour just going through basic compilations that are nothing more than `./configure --prefix=/tools && make && make check && make install`, so just doing a lot of waiting. It's 8:54PM ET.
