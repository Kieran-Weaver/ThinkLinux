# ThinkLinux

Linux for IBM Thinkpads, from the 486 to the Pentium 4

## Minimum requirements

Floppy image bin/floppy.144

RAM: 24 MB
CPU: Intel 486SX

CD Image / HDD Install

RAM: 32 MB
CPU: Intel 486SX

## Recommended specs

CPU: Pentium MMX or higher
RAM: 64 MB or higher

## Building

Requires a cross compiler for whichever target you want. Examples are:
- i486-musl
- i686-musl

### Bootable CDROM

The bootable CD is built with [buildroot](https://buildroot.org/). Place buildroot.config into the buildroot root dir as .config, and copy the other files from bootcd/ into the buildroot root dir.

### Floppy Image

Download the most recent versions of [toybox](https://landley.net/toybox/) and [the linux kernel](https://cdn.kernel.org/pub/linux/kernel/v6.x/) and extract them to toybox/ and linux/ respectively.

First, compile toybox.

```sh
cp toybox.config toybox/.config
cd toybox
CROSS_COMPILE=i486-buildroot-linux-musl- LDFLAGS="--static -static-libgcc" make root
cd ..
```
Then remake the initramfs image with the correct init script and compression
```
cp init toybox/root/i486/fs/
cd toybox/root/i486/fs
find . -printf '%P\n' | cpio -o -H newc -R +0:+0 > ../../../../initramfs.cpio
cd ../../../..
zstd -9 initramfs.cpio
```

Next, compile the kernel with the initramfs already included
```
cp initramfs.cpio linux/
cp linux.config linux/.config
cd linux
CROSS_COMPILE=i486-buildroot-linux-musl- make -j12 bzImage
```

Now, use [w98qi-tiny-floppy-bootloader](https://github.com/oerg866/w98qi-tiny-floppy-bootloader) to create a floppy image from the bzImage

```sh
git clone https://github.com/oerg866/w98qi-tiny-floppy-bootloader
cp linux/arch/x86/boot/bzImage w98qi-tiny-floppy-bootloader/
cd w98qi-tiny-floppy-bootloader
./build.sh bzImage floppy.144 1474560
```

The new floppy.144 image should be available in the current directory.
