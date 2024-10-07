# Hybrid GPU Configuration

This guide will walk you through the installation and configuration of the packages
required for Hybrid GPU systems. More specifically, it's aimed towards [NVIDIA Optimus](https://wiki.archlinux.org/title/NVIDIA_Optimus)
systems with Intel iGPUs and NVIDIA dGPUs.

The main objective is to be able to configure the system in hybrid mode, so it will
use the iGPU for low-requirements software and the dGPU for high-requirements
software seamlessly.

Another objective is to also enable power-saving features like RTD3 for the
NVIDIA card and make it function properly with suspend-resume cycles.

> [!WARNING]
>
> If you are using GDM and Wayland <u>**do not reboot until you have reached the
> end of the [DRM KMS](#14-drm-kms) steps**</u>, because the
> [Udev rules shipped with GDM](https://gitlab.gnome.org/GNOME/gdm/-/blob/main/data/61-gdm.rules.in?ref_type=heads#L48-L56)
> will make it select X11 instead of Wayland as the display protocol until you
> enable some NVIDIA services (see highlighted lines), and if you did not
> install the `xorg-server` package it will not be able to display anything.
>
> Also, in some guides (including [one in the Arch Wiki](https://wiki.archlinux.org/title/GDM#Wayland_and_the_proprietary_NVIDIA_driver),
> unfortunately) you can find suggestions to completely disable the GDM Udev
> rules by linking them to `/dev/null`.
> In my opinion, if the GDM developers made the effort to check
> for all those conditions there must be a reason, and I don't want
> to override that reasoning. I checked the rules file and saw that they simply
> require those NVIDIA services to be enabled, and it's working fine.

**Table of contents**:
<!-- TOC -->
- [1. NVIDIA Drivers](#1-nvidia-drivers)
  - [1.1. Packages](#11-packages)
  - [1.2. Services](#12-services)
  - [1.3. (Optional) Power Management on Turing](#13-optional-power-management-on-turing)
  - [1.4. DRM KMS](#14-drm-kms)
- [2. Hybrid mode](#2-hybrid-mode)
  - [2.1. Packages](#21-packages)
  - [2.2. Configuration](#22-configuration)
<!-- /TOC -->

## 1. NVIDIA Drivers

This first section is to install the required NVIDIA packages and configure them.

### 1.1. Packages

First, enable the `multilib` repo:

```bash
sudo vim /etc/pacman.conf # uncomment the entire [multilib] section
sudo pacman -Suy # update the system and download the multilib package list
```

Run the following command to install required packages:

```bash
sudo pacman -S lib32-mesa lib32-nvidia-utils lib32-opencl-nvidia lib32-vulkan-intel\
    libva-nvidia-driver libva-utils mesa mesa-utils nvidia-dkms nvidia-settings\
    nvidia-utils nvtop opencl-nvidia vdpauinfo vulkan-intel
```

> [!NOTE]
>
> If your NVIDIA GPU is Ampere or newer replace `nvidia-dkms` with `nvidia-open-dkms`,
> as that is the default package suggested by NVIDIA itself.
>
> I need to use the proprietary kernel module to enable RTD3 power-saving options
> because I own a Mobile GTX 1660 Ti, and for GPUs before Ampere (RTX 30-series)
> there is a known issue with no ETA for its resolution on the open source
> kernel driver. See [this comment on GitHub](https://github.com/NVIDIA/open-gpu-kernel-modules/issues/640#issuecomment-2186652596)
> and [Chapter 44](https://download.nvidia.com/XFree86/Linux-x86_64/560.35.03/README/kernel_open.html)
> of the NVIDIA open kernel driver manual for the known issues.

> [!NOTE]
>
> If you installed the regular `linux` kernel instead of `linux-zen` or
> other variants, use `nvidia`/`nvidia-open`, you don't need the `-dkms` packages.

### 1.2. Services

Enable some NVIDIA services to retain the ability to run in Wayland:

```bash
sudo systemctl enable nvidia-{suspend,resume,hibernate}
```

Also enable `nvidia-powerd.service` to make the driver scale frequencies
based on load:

```bash
sudo systemctl enable nvidia-powerd.service
```

See [Chapter 23](https://us.download.nvidia.com/XFree86/Linux-x86_64/560.35.03/README/dynamicboost.html)
for details.

### 1.3. (Optional) Power Management on Turing

If you have a Turing card AND want to enable power saving options for suspend
states and RTD3, you must edit the NVIDIA kernel driver parameters to disable
the onboard GSP firmware and enable S0ix-based power management:

```bash
cat <<EOF | sudo tee /etc/modprobe.d/nvidia-pm.conf > /dev/null
options nvidia NVreg_EnableGpuFirmware=0
options nvidia NVreg_EnableS0ixPowerManagement=1
EOF
```

See [Chapter 21](https://download.nvidia.com/XFree86/Linux-x86_64/560.35.03/README/powermanagement.html)
and [Chapter 43](https://download.nvidia.com/XFree86/Linux-x86_64/560.35.03/README/gsp.html)
for explanations about those parameters.

### 1.4. DRM KMS

We need to add the NVIDIA kernel modules to the initramfs to enable them as
fast as possible at boot:

```bash
sudo vim /etc/mkinitcpio.conf
```

Edit the following variables and add the specified values:

```plaintext
MODULES=(... i915 nvidia nvidia_modeset nvidia_uvm nvidia_drm ...)
```

Save, then regenerate initramfs and kernel images:

```bash
sudo mkinitcpio -P
```

At this point you should reboot to make sure everything is still working.

## 2. Hybrid mode

Now that NVIDIA drivers are in place, you can enable Hybrid mode to make efficient
use of both your GPUs.

### 2.1. Packages

I chose to use [Envycontrol](https://github.com/bayasdev/envycontrol)
as it automates some file creations and it works flawlessly with Wayland.

There is also the [GPU Profile Selector](https://github.com/LorenzoMorelli/GPU_profile_selector)
GNOME extension to quickly select a profile via GUI. Note, though, that it's still
(as of october 2024) [not compatible with GNOME 47](https://github.com/LorenzoMorelli/GPU_profile_selector/issues/23).
Nonetheless, as the developer says he's working on it, I will leave the
instructions to install it.

To also enable manual selection from Desktop entries, you can use [`switcheroo-control`](https://wiki.archlinux.org/title/PRIME#Gnome_integration).

To install each of these packages, run:

```bash
sudo pacman -S switcheroo-control
yay envycontrol gnome-shell-extension-gpu-profile-selector-git
```

### 2.2. Configuration

To setup your system to use both GPUs, run:

```bash
sudo systemctl enable --now switcheroo-control.service
sudo envycontrol -s hybrid --rtd3 2
```

> [!NOTE]
>
> See [Envycontrol's README](https://github.com/bayasdev/envycontrol?tab=readme-ov-file#hybrid)
> for the meaning of the `2` in the command above and other available values.
>
> See also the [Envycontrol wiki page](https://github.com/bayasdev/envycontrol/wiki/Frequently-Asked-Questions#hybrid)
> for details about selecting the GPU explicitly.

Reboot and enjoy hybrid mode!

[Go back to index](../#guides)

[Next: Gaming](../4.Gaming/)
