# Linux kernel module for Thrustmaster T300RS, T248 and (experimental) TX, T128, T598, T-GT II, TS-PC and TS-XW wheels

> **⚠️ MRJUIZY FORK - VIBECODED T598 FIXES ⚠️**  
> This repository fork contains specific, vibecoded fixes for the Thrustmaster T598 Steering Wheel to get Force Feedback working correctly on Linux.  
> 
> **Applied Fixes in this fork:**
> * **Merged PR #205:** Added base Force Feedback mapping for the Thrustmaster T598.
> * **Firmware bcdDevice Fix:** Modified `tmff2_probe` to correctly identify T598 firmwares 5.98, 5.99, and 6.00 (preventing it from defaulting back to T248 behavior and breaking FFB).
> * **xpad Hijack Prevention:** Added udev rules to unbind `xpad`, preventing the generic Xbox controller driver from hijacking the wheel during its initialization phase (ID `b6a5`).
> * **USB Autosuspend Fix:** Added udev rules disabling USB autosuspend (`power/autosuspend="-1"`) to prevent the wheel's FFB from dying after 2 seconds due to the Direct Drive safety watchdog starving.
> * **URB Flood Fix:** Added a default modprobe configuration (`timer_msecs=16`) to decrease the update aggressiveness, stopping the USB controller from locking up with `-1 (EPERM)` buffer errors.

> **DISCLAIMER:** The module is ready for use in most force
> feedback games, supports rangesetting as well as gain and autocentering along
> with most force feedback effects. While I haven't personally come across any
> crashes or lockups with this version, I can't promise that they won't occur
> under any circumstances.

![GitHub last commit (master)](https://img.shields.io/github/last-commit/Kimplul/hid-tmff2/master)
![License](https://img.shields.io/github/license/Kimplul/hid-tmff2)
![GitHub contributors](https://img.shields.io/github/contributors/Kimplul/hid-tmff2)


## Description

A Linux kernel module for Thrustmaster T300RS, T248, and (experimental support)
TX, T128, T598, TS-PC and TS-XV wheels.

I've been working on enhancing the real-time updating of effects, and although
it's not flawless yet, the overall experience is gradually improving. There are
a couple of issues, though. First, there might be occasional inaccuracies in how
the effects compare to the Windows driver. Second, in certain games, the mapping
of pedal inputs can be inconsistent. This means that while all pedals should be
recognized by the games, they might not be mapped correctly.

I only have access to the base editions of T300RS and T248 wheels to test with, but
from reports it seems that other editions (F1, GT, Alcantara, etc.) should also work
with this driver.

TX support was contributed by
[@davidedmundson](https://github.com/davidedmundson),

TS-XW support was contributed by
[@yassineimounachen](https://github.com/yassineimounachen).

TS-PC support was contributed by
[@BDave95](https://github.com/BDave95)

## Installation

> **⚠️ IMPORTANT: Only the DKMS installation method has been tested in this fork. ⚠️**  
> **The manual build-from-source method is NOT tested and may not work correctly.**  
> **Use DKMS unless you know what you are doing.**

You can install this kernel module using DKMS (recommended and tested) or
manually building from source (untested in this fork). If you're unsure which to pick,
go with DKMS — it will automatically recompile the driver
whenever needed and is the only method verified to work with the fixes in this fork.

> **⚠️ NOTE: This fork is NOT available in the AUR.**  
> The AUR package [hid-tmff2-dkms-git](https://aur.archlinux.org/packages/hid-tmff2-dkms-git) tracks the upstream repository, **not this fork**. Install manually using the DKMS instructions below to get the T598 fixes.

### Dependencies

Kernel modules require kernel headers to be installed. Use any
one of the right command for your distribution:

```shell
sudo apt install linux-headers-generic dkms       # Debian-based
sudo pacman -S linux-headers dkms                 # Arch-based
sudo yum install kernel-devel kernel-headers dkms # Fedora-based
```

The SteamDeck has a few possible options it seems, try some of these:
```shell
sudo pacman -S linux-neptune-61-headers
sudo pacman -S linux-neptune-65-headers
sudo pacman -S linux-neptune-68-headers
```

If none of the above work, please do open up an issue.

Joystick utilities from [linuxconsole tools](http://sf.net/projects/linuxconsole/)
are needed for `udev` rules to work:
```shell
sudo apt install joystick          # Debian-based
sudo pacman -S joyutils            # Arch-based
sudo yum install linuxconsoletools # Fedora-based
```

#### Manual installation *(NOT tested in this fork — use DKMS instead)*
+ Unplug wheel from computer
+ Run
  ```shell
  git clone --recurse-submodules https://github.com/Kimplul/hid-tmff2.git
  cd hid-tmff2
  make
  sudo make install
  # sudo make steamdeck-rules # ONLY run if you're on a SteamDeck
  ```
+ Plug wheel back in
+ Reboot *(Optional, yet Recommended)*

#### DKMS (Dynamic Kernel Module Support) — ✅ Tested & Recommended

+ Unplug wheel from computer
+ Run
  ```shell
  git clone --recurse-submodules https://github.com/MrJuizy/hid-tmff2.git
  cd hid-tmff2
  sudo ./dkms/dkms-install.sh
  sudo make udev-rules # optional but should fix some common issues
  # sudo make steamdeck-rules # ONLY run if you're on a SteamDeck
  ```
+ Plug wheel back in
+ Reboot *(Optional, yet Recommended)*

> **NOTE:** See [INTEGRATION](./docs/INTEGRATION.md)
> for install instructions for other linux distributions.

> **NOTE:** On some systems, you will get an error/warning about SSL. This is
> normal for unsigned modules. For info on signing modules yourself
> (completely optional), see
> [here](https://www.kernel.org/doc/html/latest/admin-guide/module-signing.html).

> **NOTE:** Thrustmaster TX and TS-XW wheels aren't supported by `hid-tminit` as of yet,
> meaning that the wheels have to be initialized with `tmdrv`. Please see
> https://github.com/Kimplul/hid-tmff2/issues/48.

> **NOTE:** When using Secure Boot and DKMS, you need to remember to add DKMS MOK certificate
> otherwise the module won't be loaded and the wheel might function incorrectly/not at all.
> You can follow the steps [here](https://github.com/dell/dkms?tab=readme-ov-file#secure-boot)
> on how to add DKMS MOK certificate.

> **WARNING:** There have been reports that this driver does not work if
> the wheel's firmware version is older than v. 31. To update the firmware, you
> will have to fire up a Windows installation and update the firmware using the
> official Thrustmaster tools.

> **WARNING:** There was a name change when adding support for the T248
> from `hid-tmt300rs` to `hid-tmff-new`, and you may have to uninstall the older
> version of the driver.

## Contributing

If you have a wheel that's not not supported, but suspect it might fit into the
driver, please feel free to open up an issue about it. Currently open requests
for wheels:

+ [T500 RS](https://github.com/Kimplul/hid-tmff2/issues/18)
+ [T818](https://github.com/Kimplul/hid-tmff2/issues/58)

## Common issues and notes

+ If buttons work in games but there's no FFB, try
  ```shell
  echo 'options hid-tmff-new open_mode=0' | sudo tee /etc/modprobe.d/hid-tmff-new.conf
  ```

  Generally, the wheel only starts handling force effects when 'opened' by an
  application, but some tools like key remappers may interfere with this.
  `open_mode=0` 'opens' the wheel immediately to work around this, but increases
  power draw and sets the fan spinning when not using the wheel, which might be
  a bit annoying.

+ To change gain, autocentering etc. use
  [Oversteer](https://github.com/berarma/oversteer).

+ Reportedly some games running under Wine/Proton won't recognize wheels without
  the official Thrustmaster drivers installed within the prefix. See
  [#46](https://github.com/Kimplul/hid-tmff2/issues/46#issuecomment-1199080845).
  For installation instructions, see
  [DRIVER](./docs/DRIVER.md).

  Note that you will still need
  the Linux driver, the Windows driver just installs some files needed by games to
  correctly recognize the Linux driver. The Windows driver itself does not work
  under Wine/Proton.

+ If games don't detect any input from the wheel, try disabling Steam Input.

+ Until the updated `hid-tminit` is
  [upstreamed](https://github.com/scarburato/hid-tminit), you might want to
  blacklist the kernel module `hid-thrustmaster`. Do this with
  ```shell
  echo 'blacklist hid_thrustmaster' | sudo tee /etc/modprobe.d/hid_thrustmaster.conf
  ```

+ If you've bought a new wheel, you might have to update the firmware
  through Windows before it will work with this driver.

+ T300 RS has an advanced F1 mode that can be activated with an F1 attachment
  when in PS3 mode. The base wheel will also work in PS4 mode, but it's less
  tested and if you encounter issues with this mode, please feel free to open up
  an issue about it.

+ T248 isn't as extensively tested as T300 RS, please see issues and open new
  ones if you encounter problems. There is currently no support for the built-in
  screen.

+ TX support is considered experimental, please see issues
  (especially https://github.com/Kimplul/hid-tmff2/issues/48)
  and open new ones if you encounter any problems.

+ There have been reports that some games work better with a different timer
  period (see [#11](https://github.com/Kimplul/hid-tmff2/issues/11) and
  [#10](https://github.com/Kimplul/hid-tmff2/issues/10)).

  To change the timer period, create `/etc/modprobe.d/hid-tmff-new.conf`
  and add `options hid-tmff-new timer_msecs=NUMBER` into it.
  The default timer period is 8, but numbers as low as 2 should work alright.

+ The T-GT II might show up as a T300 at the moment, since it reuses the T300
  USB product ID.
