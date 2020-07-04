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
