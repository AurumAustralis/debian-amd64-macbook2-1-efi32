<div align="center">
  <img src="macbook21.png" alt="Apple MacBook2,1" width="650">
</div>

<p align="center">
  <a href="README.es.md">Español</a> · <strong>English</strong>
</p>

---

# Debian 64-bit on MacBook2,1 with 32-bit EFI using Ventoy

A practical guide to installing **64-bit Debian GNU/Linux (`amd64`)** on an **Apple MacBook2,1** from 2006/2007, equipped with a 64-bit Intel Core 2 Duo processor but a **32-bit EFI firmware**.

This procedure uses **Ventoy** to create a USB drive capable of booting in **IA32 UEFI** mode and loading the standard Debian `amd64` image. It also documents the post-install fixes needed on this machine: an **intermittent black screen at boot** and a **login loop with LXQt + SDDM**.

> [!IMPORTANT]
> This guide is based on an installation performed and tested on a **MacBook2,1**. Ventoy considers IA32 UEFI support experimental, so behavior may vary on other old Intel-based Apple computers.

---

## Contents

- [The problem](#the-problem)
- [Tested hardware](#tested-hardware)
- [Requirements](#requirements)
- [Method 1: prepare the USB drive from Linux](#method-1-prepare-the-usb-drive-from-linux)
- [Method 2: prepare the USB drive from Windows](#method-2-prepare-the-usb-drive-from-windows)
- [Booting the MacBook2,1](#booting-the-macbook21)
- [Installing Debian](#installing-debian)
- [Fix the intermittent black screen at boot](#fix-the-intermittent-black-screen-at-boot)
- [Desktop environment](#desktop-environment)
  - [Option A: Xfce](#option-a-xfce)
  - [Option B: LXQt with SDDM](#option-b-lxqt-with-sddm)
- [Verify that Debian was installed as 64-bit](#verify-that-debian-was-installed-as-64-bit)
- [Troubleshooting](#troubleshooting)
- [Why this works](#why-this-works)
- [References](#references)
- [Result](#result)

---

## The problem

The MacBook2,1 has an unusual combination:

| Component | Architecture |
|---|---|
| Intel Core 2 Duo processor | 64-bit |
| Operating system it can run | 64-bit (`amd64`) |
| Mac EFI firmware | 32-bit (`IA32`) |

Because of this, a USB drive prepared in the usual way may not appear in the Mac startup manager or may fail before the installer starts.

Ventoy includes **IA32 UEFI** support and makes it possible to use a Debian `amd64` ISO without manually building a 32-bit EFI GRUB loader.

---

## Tested hardware

| Item | Value |
|---|---|
| Computer | Apple MacBook2,1 (2006) |
| CPU | Intel Core 2 Duo T7200 @ 2.00 GHz (x86-64) |
| GPU | Intel GMA 950 (945GM) |
| Firmware | Apple EFI 1.1, 32-bit (IA32) |
| System | Debian 13 (trixie) `amd64`, kernel 6.12 |
| Bootloader | `grub-efi-ia32` |
| Installation media | USB 2.0 flash drive with Ventoy |
| Desktops tested | Xfce (works out of the box) and LXQt + SDDM (works after the fixes in this guide) |

Resulting system, as reported by `fastfetch`:

<div align="center">
  <img src="fastfetch.png" alt="fastfetch output on the MacBook2,1 running Debian 13 amd64" width="850">
</div>

> [!NOTE]
> `fastfetch` lists the GMA 950 twice ("Discrete" and "Integrated"). This is expected: the 945GM exposes two PCI functions for the same chip. There is only one GPU.

---

## Requirements

1. A MacBook2,1.
2. A USB flash drive. **USB 2.0** is recommended for this old hardware.
3. Another computer running:
   - Linux, or
   - Windows.
4. Ventoy.
5. An official Debian image for the `amd64` architecture.
6. Recommended: another computer on the same network to access the MacBook over **SSH** during the post-install steps.

### Downloads

- Ventoy: https://www.ventoy.net/en/download.html
- Debian: https://www.debian.org/download

For a network installation, you can use an image such as:

```text
debian-13.x.x-amd64-netinst.iso
```

The exact version will change as new Debian releases become available.

> [!NOTE]
> For this procedure, the standard **`amd64`** ISO is recommended. You do not need to use the `debian-mac-...-amd64-netinst.iso` image.

---

## Method 1: prepare the USB drive from Linux

### 1. Download Ventoy

Download the Linux version of Ventoy and extract it.

For example:

```bash
tar -xzf ventoy-x.x.xx-linux.tar.gz
cd ventoy-x.x.xx
```

### 2. Identify the USB drive

Connect the USB drive and run:

```bash
lsblk -d -e 7,11 -o NAME,SIZE,MODEL,TRAN
```

Example:

```text
NAME   SIZE MODEL             TRAN
sda    477G SSD
sdb   14.9G USB Flash Drive   usb
```

In this example, the USB drive is:

```text
/dev/sdb
```

> [!CAUTION]
> Carefully verify which device is the USB drive. Selecting the wrong disk can destroy data on another drive.

### 3. Install Ventoy on the USB drive

From the directory where Ventoy was extracted, run:

```bash
sudo ./Ventoy2Disk.sh -i /dev/sdX
```

Replace `/dev/sdX` with the correct device.

Example:

```bash
sudo ./Ventoy2Disk.sh -i /dev/sdb
```

Ventoy will ask for confirmation before modifying the drive.

> [!WARNING]
> A normal Ventoy installation formats the device and deletes its existing contents.

The command above uses the **MBR** partition scheme, which is Ventoy's default and the one used in this procedure.

### 4. Copy the Debian ISO

After Ventoy has been installed, disconnect and reconnect the USB drive if necessary.

A partition normally named `Ventoy` will appear. Copy the Debian ISO directly into it:

```text
debian-13.x.x-amd64-netinst.iso
```

Do not extract the ISO and do not write it using `dd`. Ventoy will detect the ISO file and display it in its boot menu.

---

## Method 2: prepare the USB drive from Windows

This method is easier because Ventoy provides a graphical interface.

### 1. Download Ventoy for Windows

Download `ventoy-x.x.xx-windows.zip` and extract it into a folder.

### 2. Run Ventoy

Inside the extracted folder, run `Ventoy2Disk.exe`. It is recommended to use **Run as administrator**.

### 3. Select the USB drive

Under **Device**, carefully choose the USB drive that will be used to install Debian.

Keep the partition scheme set to **MBR** and click **Install**.

Ventoy will display warnings indicating that the USB drive contents will be erased. Confirm only after verifying that the correct device is selected.

### 4. Copy Debian to the USB drive

When Ventoy finishes, Windows will show a new drive normally named `Ventoy`. Open it in File Explorer and copy the ISO into it:

```text
debian-13.x.x-amd64-netinst.iso
```

That's all. You do not need Rufus, `dd`, Etcher, or to extract the ISO contents.

---

## Booting the MacBook2,1

### 1. Connect the USB drive

With the MacBook powered off, connect the Ventoy USB drive.

### 2. Open the Apple startup manager

Turn on the Mac and immediately hold **Option / Alt (⌥)** until Apple's startup manager appears.

The USB device should appear as a boot option. Select it and press **Enter**.

### 3. Check that Ventoy started in IA32 mode

When the Ventoy menu appears, check the information displayed at the bottom of the screen. On this Mac, the expected mode is:

```text
IA32
```

This means Ventoy has booted through the MacBook's 32-bit EFI firmware.

### 4. Select Debian

In the Ventoy menu, select `debian-13.x.x-amd64-netinst.iso`, press **Enter**, and continue with the Debian installer.

Although the EFI firmware is 32-bit, the installed Debian system will be **64-bit (`amd64`)**, because the MacBook2,1 Core 2 Duo processor supports x86-64.

---

## Installing Debian

Once the installer starts, the procedure is almost the same as on any PC:

1. Select the language.
2. Configure the keyboard.
3. Configure the network.
4. Create a user and password.
5. Partition the disk according to the desired installation type.
6. Install the base system.
7. Install GRUB when requested by Debian.
8. In **tasksel**, select the desktop environment (see [Desktop environment](#desktop-environment)), **SSH server** and **standard system utilities**.

This guide focuses on solving the specific problem of **booting a 64-bit Debian installer from a 32-bit EFI firmware**. Disk partitioning will depend on whether Debian will completely replace macOS or coexist with another operating system.

> [!TIP]
> Installing the **SSH server** is strongly recommended. If the graphical session fails, you can diagnose and fix everything from another computer.

---

## Fix the intermittent black screen at boot

### Symptom

After selecting the kernel in GRUB, the screen stays black and the computer does not progress. The failure is **intermittent**: the same kernel sometimes boots and sometimes does not, and it affects every installed kernel.

The failed boot leaves **no lines in the persistent journal**: the hang happens very early, before the root filesystem is mounted.

### Cause

The kernel's calls to the Apple EFI **runtime services** (32-bit firmware, mixed mode) hang randomly during boot. Disabling those services with the kernel parameter `efi=noruntime` eliminates the failure.

**Verification:** 9 consecutive cold boots without failures, alternating between two kernels. Before the fix, roughly one boot out of two failed.

### Step 1. If it does not boot right after installing (temporary)

This change applies only to that boot and does not modify anything on disk. It lets you get into the system to make the permanent change in Step 2.

1. In the GRUB menu, use the arrow keys to highlight **Debian GNU/Linux** (or a kernel inside "Advanced options").
2. Press **`e`**. An editor opens with the boot commands for that entry.
3. Move down to the line that starts with the word **`linux`** (not the `echo` line that says "Loading Linux…").
4. Go to the end of that line. The MacBook has no **End** key: use **Ctrl+E** (in GRUB it moves the cursor to the end of the line). A long line may wrap on screen, but it is still a single line.
5. Type a space followed by `efi=noruntime`.
6. Press **Ctrl+X** (or **F10**) to boot with that change. **Esc** exits without booting and discards the edit.

Editor **before** editing (UUID and kernel version vary on every installation):

```text
setparams 'Debian GNU/Linux'
        load_video
        insmod gzio
        insmod part_gpt
        insmod ext2
        search --no-floppy --fs-uuid --set=root 244b083d-dbb8-4b17-...
        echo    'Loading Linux 6.12.107+deb13-amd64 ...'
        linux   /boot/vmlinuz-6.12.107+deb13-amd64 root=UUID=244b083d-dbb8-4b17-ba8b-27faa4a0025a ro quiet
        echo    'Loading initial ramdisk ...'
        initrd  /boot/initrd.img-6.12.107+deb13-amd64
```

**After** editing, the only difference is the end of the `linux` line:

```text
        linux   /boot/vmlinuz-6.12.107+deb13-amd64 root=UUID=244b083d-dbb8-4b17-ba8b-27faa4a0025a ro quiet efi=noruntime
```

Nothing else is deleted or changed.

> [!NOTE]
> If the screen still stays black, force a shutdown (hold the power button for 10 seconds) and try again. The original failure was intermittent, so a single failed attempt does not rule out the fix.

### Step 2. Make the parameter permanent

Once inside the system, add the parameter to `/etc/default/grub`. `update-grub` generates the menu entries from this file, so the parameter applies to every kernel, including future ones.

Line to change, **before** (Debian default):

```text
GRUB_CMDLINE_LINUX_DEFAULT="quiet"
```

**After**:

```text
GRUB_CMDLINE_LINUX_DEFAULT="quiet efi=noruntime"
```

**Option A**, with a single command (as root, using `su -`). It replaces the whole line, whatever it previously contained:

```bash
sed -i 's/^GRUB_CMDLINE_LINUX_DEFAULT=.*/GRUB_CMDLINE_LINUX_DEFAULT="quiet efi=noruntime"/' /etc/default/grub
```

**Option B**, by hand: `nano /etc/default/grub`, edit the line, save with **Ctrl+O** and **Enter**, and exit with **Ctrl+X**.

In either case, check and regenerate the menu:

```bash
grep CMDLINE /etc/default/grub
update-grub
```

The `grep` must show exactly:

```text
GRUB_CMDLINE_LINUX_DEFAULT="quiet efi=noruntime"
GRUB_CMDLINE_LINUX=""
```

About the values: `quiet` hides kernel messages during boot; to see them (useful when diagnosing), leave only `"efi=noruntime"`. You can also add `splash` for a graphical boot screen. The only mandatory part is that **`efi=noruntime` is always present**.

After the next reboot, `cat /proc/cmdline` should show something like:

```text
BOOT_IMAGE=/boot/vmlinuz-6.12.107+deb13-amd64 root=UUID=244b083d-... ro quiet efi=noruntime
```

Pressing `e` in GRUB will also show `efi=noruntime` already at the end of the `linux` line.

### Step 3. Protect future GRUB updates

With `efi=noruntime`, Linux cannot write to the NVRAM. Configure GRUB so it does not try to, and so it copies itself to the fallback path the Mac always checks:

```bash
echo "grub-efi-ia32 grub2/update_nvram boolean false" | debconf-set-selections
echo "grub-efi-ia32 grub2/force_efi_extra_removable boolean true" | debconf-set-selections
dpkg-reconfigure -f noninteractive grub-efi-ia32
```

### Step 4. Verify

```bash
cat /proc/cmdline                    # must include efi=noruntime
ls /boot/efi/EFI/BOOT/               # BOOTIA32.EFI must be present
grep GRUB_DEFAULT /etc/default/grub  # should be 0 (newest kernel)
apt-mark showhold                    # no kernels should be on hold
```

### Step 5 (optional). Persistent journal

It is already enabled on Debian 13. If it is not:

```bash
mkdir -p /var/log/journal
systemd-tmpfiles --create --prefix /var/log/journal
systemctl restart systemd-journald
```

### What you lose with `efi=noruntime`

- `efibootmgr` cannot be used from Linux. If you need it, boot once without the parameter by editing the entry in GRUB with `e`.
- `pstore` (crash logs stored in NVRAM) stops working. The persistent journal covers that role.

### Ruled-out hypotheses

| Hypothesis | Why it was ruled out |
|---|---|
| Not enough space in `/boot` | `/boot` is on the root filesystem with 87 GB free |
| Damaged initrd | The failure was intermittent; a broken initrd would always fail |
| DKMS modules | `dkms status` returned nothing |
| Intel microcode | Kernel 6.12.94 does not load it and failed anyway |
| Regression in kernel 6.12.107 | 6.12.94 failed too |
| Full NVRAM | Only 15 EFI variables and an empty pstore |

---

## Desktop environment

Two desktops were tested on this hardware:

| Desktop | Result |
|---|---|
| **Xfce** | Works correctly with no extra configuration. Recommended for a quick setup. |
| **LXQt + SDDM** | Login loop on a default install. Works after the fixes below. |

### Option A: Xfce

During Debian's software selection step (tasksel), choose **Xfce**. It is relatively lightweight and worked correctly without any additional changes.

### Option B: LXQt with SDDM

This is a from-scratch installation that avoids the login loop.

| Item | Value |
|---|---|
| Desktop | LXQt with the **xfwm4** window manager |
| Login manager | SDDM |
| Video driver | `modesetting` without acceleration |

#### Symptom

After entering the username and password in SDDM, the screen returns to the login prompt (loop). This did not happen with Xfce.

#### Cause

The X server (Xorg) crashed with a **Segmentation fault** in `pci_device_vgaarb_set_target` (libpciaccess) about 2 seconds after the session started. It was triggered by **xscreensaver**, which LXQt installs as a recommended package. In addition, with the `modesetting` driver, **glamor** acceleration prevents X from starting on the GMA 950.

> [!NOTE]
> The Qt messages about `libxcb-cursor0` in `~/.xsession-errors` are misleading: they appear because X has already died. They are not the cause.

#### 1. Base installation

- Install Debian 13 normally.
- In **tasksel**, select **LXQt**, **SSH server** and **standard system utilities**.
- If the installer asks for the display manager, choose **sddm**.
- When it finishes, **do NOT log into the graphical session yet**. Do the next steps over SSH from another computer or from a text console (**Ctrl + Alt + Fn + F3**).

> [!IMPORTANT]
> Steps 2 to 6 are done as root (`su -`), **except step 4**, which is done as your normal user.

#### 2. Remove and block xscreensaver (main cause)

Remove it:

```bash
apt purge xscreensaver xscreensaver-data
apt autoremove
dpkg -l | grep -i xscreensaver   # no "ii" lines
```

Block it so no future `apt install` brings it back:

```bash
cat > /etc/apt/preferences.d/no-xscreensaver << 'EOF'
Package: xscreensaver*
Pin: release *
Pin-Priority: -1
EOF
```

#### 3. Video driver: modesetting without acceleration

Remove the old intel driver:

```bash
apt purge xserver-xorg-video-intel
```

Create the configuration that disables glamor (without it, X does not start):

```bash
mkdir -p /etc/X11/xorg.conf.d
cat > /etc/X11/xorg.conf.d/20-modesetting.conf << 'EOF'
Section "Device"
    Identifier "GMA950"
    Driver     "modesetting"
    Option     "AccelMethod" "none"
EndSection
EOF
```

#### 4. xfwm4 as window manager, without compositor

> [!WARNING]
> Do this step with **your normal user, NOT as root**, to avoid breaking permissions in your home directory. If you are root, type `exit` first.

Make sure xfwm4 is installed (as root):

```bash
which xfwm4 || apt install xfwm4
```

Tell LXQt to use xfwm4 (as user):

```bash
mkdir -p ~/.config/lxqt
if [ -f ~/.config/lxqt/session.conf ]; then
  sed -i 's/^window_manager=.*/window_manager=xfwm4/' ~/.config/lxqt/session.conf
else
  printf '[General]\nwindow_manager=xfwm4\n' > ~/.config/lxqt/session.conf
fi
```

Disable the xfwm4 compositor (as user):

```bash
F="$HOME/.config/xfce4/xfconf/xfce-perchannel-xml/xfwm4.xml"
mkdir -p "$(dirname "$F")"
cat > "$F" << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<channel name="xfwm4" version="1.0">
  <property name="general" type="empty">
    <property name="use_compositing" type="bool" value="false"/>
  </property>
</channel>
EOF
cat "$F"     # check that it prints the XML
```

> [!NOTE]
> Type `F="$HOME/..."` on a single line. Do not use `su - user` followed by pasted commands: `su` waits for the password and swallows the next line.

#### 5. UTF-8 locale

Avoids Qt warnings and sets the desktop language (as root). Select your locale (for example `en_US.UTF-8` or `es_AR.UTF-8`) and set it as default:

```bash
dpkg-reconfigure locales
```

#### 6. Confirm SDDM and reboot

```bash
dpkg-reconfigure sddm     # if asked, choose sddm
reboot
```

At the SDDM login screen, choose the **LXQt** session and log in. If the graphical screen does not appear on its own, try **Ctrl + Alt + Fn + F2** (or F1/F7).

#### 7. Verification

```bash
# Must use modesetting, without glamor:
grep -E 'modeset\(0\)|AccelMethod|glamor' /var/log/Xorg.0.log | head

# Must not show anything installed:
dpkg -l | grep -E 'xscreensaver|light-locker|lightdm'

# Must say /usr/bin/sddm:
cat /etc/X11/default-display-manager
```

#### What NOT to do

| Avoid | Reason |
|---|---|
| Installing xscreensaver | Causes the Xorg crash in vgaarb and the login loop. |
| Installing light-locker | Depends on LightDM: it installs it and asks to change the login manager. It does not work with SDDM. |
| Using glamor / removing `20-modesetting.conf` | With modesetting and glamor acceleration, X does not start on the GMA 950. |
| Enabling the xfwm4 compositor | Unnecessary on this GPU; left off for stability. |

> [!TIP]
> **Screen locking:** if needed, try `xsecurelock` or `slock` (package `suckless-tools`) bound to a keyboard shortcut. Test them carefully: if they touch DPMS or gamma, they could trigger the same bug.

#### If the loop comes back: diagnosis

Attempt a login, then, over SSH as root:

```bash
date
journalctl -b -u sddm --no-pager | tail -40
grep -E '\(EE\)|Fatal|Segmentation|vgaarb' -A12 /var/log/Xorg.0.log.old
tail -40 /home/USER/.xsession-errors
```

| What you see | Meaning / what to do |
|---|---|
| Segmentation fault in `pci_device_vgaarb_set_target` | X crashes because of a program in the session. Check that xscreensaver is not installed (step 2). |
| `Failed to read display number from pipe` | X does not even start. Check `20-modesetting.conf` (step 3). |
| Qt errors about `libxcb-cursor0` | A consequence of X dying, not the cause. Look at `Xorg.0.log.old`. |
| Text screen after boot, but SSH works | The system is fine; only the graphical part fails. Diagnose over SSH. |

**Plan B:** if nothing works, LightDM also works on this hardware:

```bash
apt install lightdm lightdm-gtk-greeter
dpkg-reconfigure lightdm     # choose lightdm
reboot
```

---

## Verify that Debian was installed as 64-bit

After Debian starts, open a terminal and run:

```bash
dpkg --print-architecture
```

Expected output:

```text
amd64
```

Check the kernel architecture:

```bash
uname -m
```

Expected output:

```text
x86_64
```

Check whether the system booted using EFI:

```bash
if [ -d /sys/firmware/efi ]; then
    echo "System booted using EFI"
else
    echo "System booted without EFI"
fi
```

Check for 32-bit EFI GRUB packages:

```bash
dpkg -l | grep grub-efi-ia32
```

For a full summary of the system and hardware (like the screenshot in [Tested hardware](#tested-hardware)):

```bash
apt install fastfetch
fastfetch
```

---

## Troubleshooting

### The USB drive does not appear when holding Option/Alt

Shut down the Mac completely, reconnect the USB drive, and repeat the boot process while holding **Option / Alt (⌥)**.

If it still does not appear, try a **USB 2.0** flash drive and another USB port. You may also need to reset NVRAM/PRAM.

### Reset NVRAM / PRAM

Shut down the Mac. Turn it on and hold the following keys simultaneously:

```text
Command (⌘) + Option (⌥) + P + R
```

Then try booting again with the USB drive connected.

### Reset the SMC

The MacBook2,1 uses a removable battery.

1. Shut down the Mac.
2. Disconnect the power adapter.
3. Remove the battery.
4. Hold the power button for approximately **5 seconds**.
5. Reinstall the battery.
6. Connect the power adapter.
7. Turn the Mac on normally.

If this guide is being applied to another Intel Mac with a non-removable battery, the SMC reset procedure may be different and should be verified for that specific model.

### Ventoy boots but Debian does not start

First check that:

- an `amd64` ISO was downloaded;
- the ISO was copied as a file to the Ventoy partition;
- Ventoy displays `IA32` as its boot mode;
- a recent Ventoy version is being used;
- the ISO is not corrupted (verify it with the SHA checksum files published by Debian).

### Debian installer starts, but the installed system does not boot from the internal disk

The current Debian installer supports systems with a 64-bit CPU and 32-bit UEFI firmware and can install the appropriate EFI GRUB variant.

If a problem occurs, boot again from the Ventoy USB drive and use Debian rescue mode to inspect the EFI System Partition and the GRUB installation.

### Black screen after choosing the kernel in GRUB

See [Fix the intermittent black screen at boot](#fix-the-intermittent-black-screen-at-boot). If it still fails with `efi=noruntime`:

- Do not power off immediately: wait a minute and try `ping` or `ssh` from another computer. If it responds, the system booted and the problem is only video.
- Check whether GRUB's "Loading Linux…" and "Loading initial ramdisk…" messages remain on screen. If the hang happens there, it is during initrd loading, before the kernel starts.
- Then boot again and check the previous boot's log:

```bash
journalctl --list-boots --no-pager
journalctl -b -1 -p warning --no-pager | tail -50
```

**Plan B:** shrink the initrd (from about 60–70 MB to about 15–20 MB) to reduce loading through the firmware:

```bash
sed -i 's/^MODULES=.*/MODULES=dep/' /etc/initramfs-tools/initramfs.conf
update-initramfs -u -k all
ls -lh /boot/initrd.img-*
```

### Login loop with LXQt

See [If the loop comes back: diagnosis](#if-the-loop-comes-back-diagnosis).

---

## Why this works

The MacBook2,1 uses a 64-bit processor, but Apple equipped this model with 32-bit EFI firmware.

The limitation is not the processor's ability to run 64-bit Debian. The problem is the first stage of the boot process, and later the kernel's interaction with that 32-bit firmware at runtime.

Conceptually, the boot chain is:

```text
MacBook2,1
    │
    ├── 32-bit Apple EFI
    │
    ▼
Ventoy IA32 UEFI  (installation)   /   grub-efi-ia32  (installed system)
    │
    ▼
Debian amd64 kernel + efi=noruntime
    │
    ▼
64-bit Debian GNU/Linux
```

Ventoy acts as a bridge between the Mac's IA32 firmware and the Debian `amd64` installer. Once installed, `grub-efi-ia32` fills that role, and `efi=noruntime` prevents the kernel from calling the unstable EFI runtime services.

---

## References

- Ventoy - Getting Started: https://www.ventoy.net/en/doc_start.html
- Ventoy - IA32 UEFI Support: https://www.ventoy.net/en/doc_ia32.html
- Debian - Download Debian: https://www.debian.org/download
- Debian Wiki - UEFI: https://wiki.debian.org/UEFI
- Debian Wiki - Installing Debian on MacBook2,1: https://wiki.debian.org/InstallingDebianOn/Apple/MacBook/2-1
- Linux kernel - Kernel parameters: https://docs.kernel.org/admin-guide/kernel-parameters.html
- Apple - Startup key combinations: https://support.apple.com/102603

---

## Result

The final goal is:

```text
Apple MacBook2,1
x86-64 CPU
32-bit IA32 EFI
64-bit Debian amd64
Stable boot with efi=noruntime
Xfce, or LXQt + SDDM
```

without manually modifying a Debian ISO or manually building a 32-bit EFI GRUB loader for the installation USB drive.

---

## License

This procedure may be freely used, modified, and shared for educational and technical purposes.

If you find improvements or differences when following this procedure on another Intel Mac with 32-bit EFI, you can document them through an *issue* or *pull request*.
