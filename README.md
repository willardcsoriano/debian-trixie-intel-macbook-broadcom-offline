Here's the updated README with all three platforms referenced properly:

---

# Debian Trixie (13) on Intel MacBook — Complete Offline WiFi Bootstrap Kit

**Status:** Verified Working — April 2026
**Maintained by:** Willard
**Target Hardware:** MacBook Air (Mid-2015, Intel Core i5 dual-core, 8GB RAM)

---

## Why This Repository Exists

This repository serves as the documentation and instructions hub for getting a freshly installed Debian 13 (Trixie) system connected to the internet on an Intel MacBook — entirely offline, using nothing but a USB stick. The actual package files are hosted across three platforms due to GitHub's file size limitations, all linked in the Download section below.

The machine used to develop and verify this guide is a **2015 Intel MacBook Air** with a dual-core Intel Core i5 and 8GB of RAM. The migration to Debian was driven by a straightforward reason: macOS Monterey — the last version of macOS Apple supports on this hardware — had become increasingly unusable due to high idle RAM consumption, leaving very little headroom for actual work. With Apple having officially ended support for this machine, the only path forward was a full OS migration.

Debian Trixie was chosen for its stability, package freshness, and long-term support. This repository documents everything that was needed to bridge the gap between a working Debian install and a working internet connection — a gap that turns out to be surprisingly wide on MacBook hardware due to Broadcom's proprietary WiFi chips.

If you found this repository by searching for how to install Debian Trixie on an Intel MacBook, or how to get WiFi working on Debian without internet access, you are in exactly the right place.

---

## Understanding the Problem Before You Start

### The Netinst Trap

The standard way to install Debian on any machine is to use the `netinst` ISO — a small installer image that downloads the rest of the system from the internet during installation. This works perfectly on hardware with open-source-compatible network cards. MacBooks are not that hardware.

MacBooks from roughly 2010 onwards use Broadcom WiFi chipsets from the BCM43xx family. These chips are not supported by any open-source driver that ships with the Linux kernel by default. The only driver that reliably works is `broadcom-sta-dkms`, a proprietary driver distributed separately that must be **compiled from source** against your specific running kernel at install time.

This compilation requires:
- The GCC compiler toolchain
- Your kernel's header files
- The DKMS framework and its own dependencies
- Dozens of supporting libraries

None of these are present on a minimal Debian install. And you cannot download them because you have no WiFi. And you cannot get WiFi because you do not have them.

This is the **Netinst Trap** — a classic chicken-and-egg problem that has frustrated MacBook Linux users for years. This repository is the bridge.

### What This Repository Does

Every single `.deb` file needed to break this dependency loop has been manually identified, downloaded, and verified. You download them from any of the mirrors listed below, transfer them to a USB stick on any machine that has internet access, carry them into your new Debian install, and install them all at once. DKMS then compiles the Broadcom driver against your kernel, and you are online.

Once you are online, `apt` handles everything from that point forward and you will never need to do this manually again.

---

## Compatibility

| | |
|---|---|
| **Tested Hardware** | MacBook Air Mid-2015 (Intel Core i5, 8GB RAM) |
| **Broadcom Chip** | BCM43xx series (BCM4360, BCM43224 and similar) |
| **Operating System** | Debian 13 Trixie (stable) |
| **Architecture** | amd64 (Intel/AMD 64-bit) |
| **Kernel** | `6.12.73+deb13-amd64` |
| **Driver** | `broadcom-sta-dkms` v6.30.223.271-26 |

This package set should work on any Intel MacBook running Debian Trixie amd64 with a Broadcom BCM43xx WiFi chip. If you are on a different kernel version, see the Kernel Mismatch section under Troubleshooting.

---

## Download the Package Collection

The full package set is hosted on three platforms. All three contain identical files — use whichever is fastest or most accessible for you.

| Platform | Link | Notes |
|---|---|---|
| **Internet Archive** | https://archive.org/details/debian-trixie-macbook-broadcom-wifi-offline | Recommended. No size limits, permanent hosting, no account needed to download. |
| **Hugging Face** | https://huggingface.co/datasets/unicornfacemask/debian-trixie-macbook-broadcom-wifi-offline | Fast CDN, good for large file downloads. |
| **Zenodo** | *(add your Zenodo DOI link here once published)* | Hosted by CERN. Assigns a permanent DOI, ideal for citation. |

> **Note on `linux-image-6.12.73+deb13-amd64`:** This file is 107MB and may be absent or separately listed on some platforms. If it is not included in a download, get it directly from the official Debian mirror:
> ```
> http://ftp.debian.org/debian/pool/main/l/linux/linux-image-6.12.73+deb13-amd64_6.12.73-1_amd64.deb
> ```
> If your system already boots into kernel `6.12.73+deb13-amd64`, you may not need to reinstall the image — but having it on the USB as a fallback is recommended.

Once downloaded, organize all `.deb` files into a single flat folder on your USB stick. Do not use subfolders — the install command targets everything in the current directory with a wildcard.

---

## What Is in the Package Collection

The packages fall into several logical groups. Understanding what each group does will help you troubleshoot if something goes wrong.

### The Driver Itself
- `broadcom-sta-dkms` — the proprietary Broadcom STA wireless driver source
- `dkms` — the Dynamic Kernel Module Support framework that compiles and manages the driver

### Kernel Build Requirements
- `linux-headers-6.12.73+deb13-amd64` — architecture-specific kernel headers
- `linux-headers-6.12.73+deb13-common` — common kernel headers shared across architectures
- `linux-kbuild-6.12.73+deb13` — kernel build scripts required by DKMS
- `pahole` and `dwarves` — debug info tools required by the kernel build process

### Compiler Toolchain
- `gcc-14`, `gcc-14-x86-64-linux-gnu` — the GNU C compiler
- `cpp-14`, `cpp-14-x86-64-linux-gnu` — the C preprocessor
- `binutils`, `binutils-x86-64-linux-gnu`, `binutils-common` — assembler, linker and binary utilities
- `make` — build automation tool
- `patch` — required by DKMS to apply source patches during compilation

### Package Management Tools
- `dpkg-dev` — Debian package development tools, required by DKMS
- `libdpkg-perl` — Perl library that dpkg-dev depends on
- `xz-utils` and `liblzma5` — compression tools required by dpkg-dev

### Core Libraries
A large collection of libraries that the compiler toolchain and kernel build system depend on, including:
- `libc6`, `libc6-dev`, `libc-dev-bin`, `linux-libc-dev` — C standard library and headers
- `libgcc-14-dev`, `libgcc-s1` — GCC runtime libraries
- `libasan8`, `liblsan0`, `libtsan2`, `libubsan1`, `libhwasan0` — GCC sanitizer libraries
- `libgomp1`, `libitm1`, `libatomic1`, `libquadmath0` — GCC support libraries
- `libcc1-0` — GCC plugin library
- `libbinutils`, `libctf0`, `libctf-nobfd0`, `libgprofng0`, `libsframe1` — binutils libraries
- `libisl23`, `libmpc3`, `libmpfr6`, `libgmp10` — math libraries for GCC
- `libelf1t64`, `libdw1t64` — ELF and DWARF library support
- `libzstd1`, `zlib1g` — compression libraries
- `libnsl2`, `libnsl-dev`, `libtirpc-dev`, `rpcsvc-proto` — network service libraries
- `libcrypt-dev` — cryptography development library
- `libfakeroot`, `fakeroot` — tool for simulating root privileges during builds
- `libidn2-0`, `libunistring5` — internationalization libraries
- `libjansson4` — JSON library
- `libgcc-14-dev` — GCC 14 development files

### WiFi Connection Tools
These are needed to actually connect to a network after the driver is built:
- `wpasupplicant` — handles WPA/WPA2 authentication
- `iw` — wireless interface configuration tool
- `libnl-3-200`, `libnl-genl-3-200` — netlink libraries required by iw and wpasupplicant
- `libssl3t64` — SSL library required by wpasupplicant

---

## Step-by-Step Installation Guide

### Before You Begin

You will need:
- A USB stick with at least 2GB of free space, formatted as FAT32, exFAT, or ext4
- A machine with internet access to download the packages — any OS works, Windows, macOS, and Linux all handle this equally well
- Your target MacBook booted into a fresh Debian Trixie minimal install
- Your WiFi network name (SSID) and password

---

### Step 1 — Download and Prepare the Packages

Download the full package collection from any of the mirrors listed in the Download section above. Place all `.deb` files into a single flat folder — not inside any subfolders. Copy that folder to your USB stick.

---

### Step 2 — Boot Into Debian and Become Root

On your MacBook, boot into Debian. At the login prompt, log in with your user account, then switch to root:

```
su -
```

Enter your root password when prompted. All commands from this point forward require root privileges. Note that a minimal Debian install does not include `sudo` by default — use `su -` throughout this guide instead of prefixing commands with `sudo`.

---

### Step 3 — Identify Your USB Partition

Plug in the USB stick. The device name Linux assigns to it depends on what other storage devices are present and the order in which they were detected at boot — it is not always the same between sessions. Always confirm the correct device name before attempting to mount anything.

Run:

```
lsblk
```

This displays all block devices and their partitions in a tree format. Look for a device whose size matches your USB stick — it will appear as `sdb`, `sdc`, `sdd`, or similar. Its partitions will be listed underneath as `sdb1`, `sdb2`, etc.

Your files will be on the data partition, which is typically the largest one. On USB sticks formatted in Windows it is often the second partition. If you need more detail to identify the right partition, run:

```
lsblk -o NAME,SIZE,FSTYPE,LABEL
```

This adds filesystem type and volume label to the output, which usually makes identification straightforward.

---

### Step 4 — Mount the USB

Once you have identified the correct partition, mount it:

```
mkdir -p /mnt/usb
mount /dev/sdb2 /mnt/usb
```

Replace `sdb2` with the actual partition name from your `lsblk` output. Then verify your files are present:

```
ls /mnt/usb
```

You should see all your `.deb` files listed. If the directory is empty or shows unexpected contents, unmount and try a different partition:

```
umount /mnt/usb
mount /dev/sdb1 /mnt/usb
```

---

### Step 5 — Install All Packages

Navigate into the mounted USB and install everything at once:

```
cd /mnt/usb
dpkg -i *.deb
```

During this pass you will see a significant number of warnings and dependency error messages. This is completely expected and does not indicate that anything has gone wrong. What is happening is that `dpkg` is unpacking all 90+ packages onto your filesystem simultaneously, but some packages cannot finish configuring yet because other packages they depend on have not been configured in the correct order. Everything is being written to disk — the configuration step that follows will resolve the ordering.

---

### Step 6 — Configure All Packages

```
dpkg --configure -a
```

This is the most critical step in the entire process. The `-a` flag instructs dpkg to iterate through every package in an unconfigured state and complete its setup, now that the full dependency set is available on disk.

This step is also where DKMS will automatically detect the newly installed kernel headers and compile the Broadcom driver against your kernel. Watch the output for this line:

```
Building initial module for 6.12.73+deb13-amd64
```

When this appears, the compilation is underway. It typically takes between 2 and 4 minutes on mid-2015 MacBook Air hardware. Do not interrupt the process. Successful completion will produce:

```
DKMS: build completed.
DKMS: install completed.
```

---

### Step 7 — Audit for Remaining Issues

```
dpkg --audit
```

This command inspects the package database for any packages that are broken, partially installed, or not properly configured. If the command returns no output at all, the system is clean and you can proceed. If it lists any packages, run the following sequence and audit again:

```
dpkg -i *.deb
dpkg --configure -a
dpkg --audit
```

Repeat until the audit produces no output.

---

### Step 8 — Verify the Driver Was Built Successfully

```
dkms status
```

The output should read:

```
broadcom-sta/6.30.223.271, 6.12.73+deb13-amd64, x86_64: installed
```

If the status shows `built` but not `installed`, run:

```
modprobe wl
```

If it shows neither built nor installed, the compilation failed — refer to the Troubleshooting section.

---

### Step 9 — Load the Driver

The open-source Broadcom drivers (`b43`, `bcma`, `ssb`) may already be loaded and will conflict with the proprietary `wl` driver. Remove them first:

```
modprobe -r b43 bcma ssb
```

Then load the proprietary driver:

```
modprobe wl
```

Confirm it loaded successfully:

```
lsmod | grep wl
```

You should see:

```
wl                   6459392  0
cfg80211             1404928  1 wl
```

Both lines together confirm the driver is running correctly.

---

### Step 10 — Bring Up the WiFi Interface

Check your wireless interface name:

```
ip link
```

Look for an interface starting with `wlp` — on the mid-2015 MacBook Air it is typically `wlp3s0`, but this can vary. Bring it up:

```
ip link set wlp3s0 up
```

Replace `wlp3s0` with your actual interface name throughout the remaining steps.

---

### Step 11 — Scan for Networks

```
iw wlp3s0 scan | grep SSID
```

You should see a list of nearby WiFi networks. Confirm your network name appears before proceeding.

---

### Step 12 — Connect to Your Network

Create a WPA configuration file for your network:

```
wpa_passphrase YOUR_NETWORK_NAME YOUR_PASSWORD > /etc/wpa_supplicant/wpa_supplicant.conf
```

Then start the WPA supplicant in the background:

```
wpa_supplicant -B -i wlp3s0 -c /etc/wpa_supplicant/wpa_supplicant.conf
```

---

### Step 13 — Obtain an IP Address

```
dhcpcd wlp3s0
```

This requests an IP address from your router via DHCP. You should see confirmation that an address was assigned within a few seconds.

---

### Step 14 — Verify Connectivity

```
ping -c 3 google.com
```

Three successful replies confirm you are online. You have successfully bridged the Netinst Trap.

---

### Step 15 — Make WiFi Persistent Across Reboots

The connection established above will not survive a reboot on its own. To make WiFi connect automatically on every boot, install NetworkManager now that you have internet access:

```
apt install network-manager
```

Enable and start it:

```
systemctl enable NetworkManager
systemctl start NetworkManager
```

Register your network:

```
nmcli device wifi connect YOUR_NETWORK_NAME password YOUR_PASSWORD
```

From this point forward your MacBook will connect to WiFi automatically on every boot, and `apt` is fully available for installing any additional software.

---

## Troubleshooting

### The b43 Error at Boot
You may see the following message during boot:
```
b43 bcma0:1 probe with driver b43 failed with error -95
```
This is harmless. It means the open-source b43 driver attempted to claim your Broadcom card and was rejected. This is expected — your card requires the proprietary `wl` driver. The message will appear at every boot and can be safely ignored.

### DKMS Build Failed
Run `dkms status`. If it does not show `installed`, the compilation failed. The most common causes are a missing `patch` package or a missing `pahole` package. Verify both:
```
dpkg -l | grep patch
dpkg -l | grep pahole
```
If either is missing, ensure it was included in your USB package collection and run `dpkg --configure -a` again.

### Kernel Mismatch
This package set targets kernel `6.12.73+deb13-amd64`. If your system booted into a different kernel version, the headers will not match and the DKMS build will fail silently. Check your running kernel:
```
uname -r
```
If the version differs, install the kernel image from this package set first, reboot into it, then follow this guide from Step 5.

### USB Device Name Changes Between Sessions
Linux assigns device names like `sdb` and `sdc` dynamically at boot based on detection order. The same USB stick may appear as `sdb` one session and `sdc` the next. Always run `lsblk` after plugging in the USB to confirm the current device name before mounting.

### sudo Not Found
A minimal Debian install does not include sudo. Use `su -` to operate as root instead. Once you are online, sudo can be installed with `apt install sudo`.

### dpkg-dev Depends on xz-utils Error
This means xz-utils was not copied to the USB. It is included in the package collection — verify all files were transferred. Check with:
```
ls /mnt/usb | grep xz
```

### Interface Name Is Not wlp3s0
Wireless interface names vary by hardware. Always check your actual interface name with `ip link` and substitute it wherever `wlp3s0` appears in this guide.

### wpa_supplicant Fails to Connect
SSIDs and passwords are case-sensitive. Inspect the generated config to verify correctness:
```
cat /etc/wpa_supplicant/wpa_supplicant.conf
```
The password appears in plain text — confirm it matches exactly.

---

## The Broader Context

This guide was written in April 2026, at a time when a significant number of Intel MacBooks from the 2013–2017 era are reaching end of support from Apple. macOS Monterey is the last version available for many of these machines, and Apple has confirmed there will be no further security updates.

For users who want to continue using this hardware securely and productively, Linux is the most viable path forward. Debian Trixie in particular offers an excellent balance of stability and modernity on older Intel hardware — it is lightweight, fast, and will receive security support for years to come.

The single biggest obstacle to this migration is the WiFi problem documented in this repository. Once it is solved, Debian runs exceptionally well on mid-2015 MacBook Air hardware. Idle RAM usage is a fraction of what macOS Monterey consumed, the system is responsive, and the full Debian package ecosystem is available.

It is hoped that this repository saves other MacBook users the hours of trial and error that went into building it.

---

## Contributing

If you are running a newer Trixie kernel than `6.12.73`, the kernel-specific packages in this set will not match. The fix is to download the matching `linux-headers`, `linux-image`, and `linux-kbuild` packages for your kernel version from:

```
http://ftp.debian.org/debian/pool/main/l/linux/
```

And substitute them for the ones in this collection. If you verify that a updated package set works on a newer kernel, please open a pull request with the updated files and a note about what changed.

---

## Citation

If you reference this work, you can cite the Zenodo archive which carries a permanent DOI:

*(add your Zenodo DOI here once published)*

---

*Born out of a real offline install session on a MacBook Air in April 2026. Every package in this collection was manually identified and verified through trial and error — sometimes a lot of error.*
