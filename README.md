# ssbagpcmOS

```
         _                                      _____ _____
        | |                                    |  _  /  ___|
 ___ ___| |__   __ _  __ _ _ __   ___ _ __ ___ | | | \ `--.
/ __/ __| '_ \ / _` |/ _` | '_ \ / __| '_ ` _ \| | | |`--. \
\__ \__ \ |_) | (_| | (_| | |_) | (__| | | | | \ \_/ /\__/ /
|___/___/_.__/ \__,_|\__, | .__/ \___|_| |_| |_|\___/\____/
                      __/ | |
                     |___/|_|
```

A minimal Linux-based operating system built from scratch using a custom kernel and BusyBox.

## Features

- 🐧 Custom Linux kernel 6.19.8
- 📦 BusyBox-based userland with Alpine package manager (apk)
- 🌐 Networking with DHCP out of the box
- 💾 Live ISO boot (runs in RAM) or install to disk
- ⚡ KVM hardware acceleration support

## Project Structure

```
.
├── iso_root/
│   ├── bzImage              # Linux kernel
│   ├── initramfs.cpio.gz    # Root filesystem (compressed)
│   ├── isolinux.bin         # ISOLINUX bootloader
│   ├── isolinux.cfg         # Bootloader configuration
│   └── ldlinux.c32          # SYSLINUX module
├── rootfs/                  # Root filesystem source (not shipped)
├── ssbagpcmOS.iso           # Bootable ISO image
└── README.md
```

## Requirements

- `qemu-system-x86_64` — Virtual machine emulator
- `xorriso` — ISO creation tool
- `cpio` — Archive tool for initramfs
- `gzip` — Compression

### Install dependencies (Debian/Ubuntu)

```bash
sudo apt install qemu-system-x86 xorriso cpio gzip
```

### Install dependencies (Arch Linux)

```bash
sudo pacman -S qemu-full xorriso cpio gzip
```

## Quick Start

### 1. Boot the Live ISO

```bash
qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -cdrom ssbagpcmOS.iso \
  -nographic \
  -m 2048M \
  -netdev user,id=net0 -device virtio-net-pci,netdev=net0
```

> **Note:** Remove `-enable-kvm -cpu host` if your system does not support KVM.

### 2. Exit QEMU

Press `Ctrl+A` then `X` to kill the VM.

## Install to Disk

### 1. Create a virtual disk

```bash
qemu-img create -f qcow2 ssbagpcmOS-disk.qcow2 10G
```

### 2. Boot the ISO with the disk attached

```bash
qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -cdrom ssbagpcmOS.iso \
  -hda ssbagpcmOS-disk.qcow2 \
  -nographic \
  -m 2G \
  -netdev user,id=net0 -device virtio-net-pci,netdev=net0
```

### 3. Run the installer

```bash
sh /install.sh
```

The installer will automatically:
- Partition the disk
- Format the partitions
- Copy the system
- Install the bootloader
- Shutdown when done

### 4. Boot from disk (no ISO needed)

```bash
qemu-system-x86_64 \
  -enable-kvm \
  -cpu host \
  -hda ssbagpcmOS-disk.qcow2 \
  -nographic \
  -m 2G \
  -netdev user,id=net0 -device virtio-net-pci,netdev=net0
```

## Building from Source

### Rebuild the initramfs

If you modify the rootfs, rebuild the initramfs:

```bash
cd rootfs
find . | cpio -H newc -o | gzip > ../iso_root/initramfs.cpio.gz
```

### Rebuild the ISO

```bash
xorriso -as mkisofs \
  -o ssbagpcmOS.iso \
  -b isolinux.bin \
  -c boot.cat \
  -no-emul-boot \
  -boot-load-size 4 \
  -boot-info-table \
  iso_root/
```

### Full rebuild (initramfs + ISO)

```bash
cd rootfs
find . | cpio -H newc -o | gzip > ../iso_root/initramfs.cpio.gz
cd ..
xorriso -as mkisofs \
  -o ssbagpcmOS.iso \
  -b isolinux.bin \
  -c boot.cat \
  -no-emul-boot \
  -boot-load-size 4 \
  -boot-info-table \
  iso_root/
```

## Commands Reference

| Action | Command |
|---|---|
| Boot live ISO | `qemu-system-x86_64 -enable-kvm -cpu host -cdrom ssbagpcmOS.iso -nographic -m 2048M -netdev user,id=net0 -device virtio-net-pci,netdev=net0` |
| Create virtual disk | `qemu-img create -f qcow2 ssbagpcmOS-disk.qcow2 10G` |
| Boot ISO + disk | `qemu-system-x86_64 -enable-kvm -cpu host -cdrom ssbagpcmOS.iso -hda ssbagpcmOS-disk.qcow2 -nographic -m 2G -netdev user,id=net0 -device virtio-net-pci,netdev=net0` |
| Boot from disk | `qemu-system-x86_64 -enable-kvm -cpu host -hda ssbagpcmOS-disk.qcow2 -nographic -m 2G -netdev user,id=net0 -device virtio-net-pci,netdev=net0` |
| Rebuild initramfs | `cd rootfs && find . \| cpio -H newc -o \| gzip > ../iso_root/initramfs.cpio.gz` |
| Rebuild ISO | `xorriso -as mkisofs -o ssbagpcmOS.iso -b isolinux.bin -c boot.cat -no-emul-boot -boot-load-size 4 -boot-info-table iso_root/` |
| Exit QEMU | `Ctrl+A` then `X` |



