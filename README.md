# rcraid-dkms

AMD RAIDXpert driver as DKMS package

## Foreword

Many AMD mainboards with RAID support are available across different sockets and platforms:

### AM4 Socket

- X370 / B350
- X470 / B450
- X570 / B550

### AM5 Socket

- X670 / B650
- X870 / B850

### TR4 Socket (Threadripper)

- X399

### sTRX4 Socket (Threadripper)

- TRX40

### sWRX8 Socket (Workstation Threadripper)

- WRX80

> **Note:** RAID functionality requires enabling RAID mode in BIOS and using the appropriate driver for your operating system.

On Linux, AMD provides the driver either as a binary blob or as full source code.
Without automation you would need to recompile the module manually after each kernel update.
Thanks to DKMS this process becomes automatic.

---

# Building From Source (Debian/Ubuntu)

This section explains how to compile the driver and create a Debian package using `debuild -us -uc`.

## 1. Install prerequisites

```bash
sudo apt-get update
sudo apt-get install \
    build-essential \
    dkms \
    devscripts \
    debhelper \
    dpkg-dev \
    fakeroot \
    git \
    linux-headers-$(uname -r)
```

## 2. Clone the repository

```bash
git clone https://github.com/marcofortina/rcraid-dkms.git
cd rcraid-dkms
```

## 3. Build the DKMS Debian package

```bash
debuild -us -uc
```

This will generate a `.deb` package in the parent directory, e.g.:

```
../rcraid-dkms_<version>_amd64.deb
```

## 4. Install the generated package

```bash
sudo dpkg -i ../rcraid-dkms_*_amd64.deb
```

DKMS will automatically compile and install the module for your running kernel.

---

# Installing rcraid During Linux Installation

This section describes how to embed the AMD RAID driver *during the OS installation process*, ensuring that the installer detects RAID volumes correctly.

Documentation is based on AMD’s official quick start guides.

---

## Ubuntu Installation

Source reference: *AMD RAID Quick Start Guide for Ubuntu Operating System – rev 0.5.1*

### 1. Create RAID array in BIOS

Enable RAID mode → create array → save and reboot.

### 2. Boot the Ubuntu installer

### 3. Load the AMD RAID driver into the installer environment

1. Switch to a TTY (Ctrl+Alt+F2).
2. Mount your driver media:

```bash
mount /dev/sdX1 /mnt
```

3. Install the driver:

```bash
dpkg -i /mnt/rcraid-dkms*.deb
modprobe rcraid
```

4. Switch back to the installer (Ctrl+Alt+F1).

### 4. Continue installation and update initramfs

```bash
echo "rcraid" | sudo tee -a /etc/initramfs-tools/modules
sudo update-initramfs -u
```

---

## RHEL Installation

Source reference: *AMD RAID Quick Start Guide for RHEL Operating System – rev 1*

### 1. Boot installer and enable driver disk mode

At the GRUB screen press **e** and append:

```
inst.dd
```

Boot with **Ctrl+X**.

### 2. Provide the driver disk

Select:

```
Driver Disk → Local disk / USB
```

### 3. After installation ensure module is loaded

```bash
echo "rcraid" > /etc/modules-load.d/rcraid.conf
dracut --force
```

---

# Arch Linux Integration

On Arch, the kernel installer does not auto-load DKMS modules.
You must inject the driver into the installation ISO or load it manually.

## 1. Install prerequisites

```bash
sudo pacman -S --needed base-devel dkms linux-headers git
```

## 2. Build the package

```bash
git clone https://github.com/marcofortina/rcraid-dkms.git
cd rcraid-dkms/arch
makepkg -si
```

## 3. Add rcraid to initramfs

Edit `/etc/mkinitcpio.conf`:

```
MODULES=(rcraid)
```

Rebuild:

```bash
sudo mkinitcpio -P
```

---

# Additional Notes

- AMD RAID support on Linux is limited.
- mdadm, LVM or ZFS provide far more reliable alternatives.

---

# Further Information

- [AMD RAID Quick Start Guide for the Ubuntu Desktop Operating System](https://docs.amd.com/v/u/en-US/56966_1.01)
- [AMD RAID Quick Start Guide for the Red Hat® (RHEL) Operating System](https://docs.amd.com/v/u/en-US/56963_1.20)

---

# Thanks to…

- [**AMD**](https://www.amd.com/en/support/downloads/drivers.html/chipsets/swrx8/wrx80.html), for providing source code
- [**Thomas Karl Pietrowski (@thopiekar)**](https://github.com/thopiekar/rcraid-dkms), for the original rcraid-dkms repository
- [**Martin Weber (@martinkarlweber)**](https://github.com/martinkarlweber/rcraid-patches), for patches supporting Linux ≥ 4.15
- [**LENOVO**](https://pcsupport.lenovo.com/us/en/products/workstations/thinkstation-p-series-workstations/thinkstation-p620/downloads/ds568883-amd-raid-driver-for-rhel-93-thinkstation-p8), for providing modified source code for RHEL 9.3