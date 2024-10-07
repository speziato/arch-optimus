# Arch Installation from scratch - Optimus Laptop

## Introduction

This is a step by step guide about installing Arch on a laptop with dual-GPUs
(specifically Intel UHD + NVIDIA).
My personal objective is to have a nice gaming experience.

This guide wants to be a reference for myself if I need to do all this
again, and maybe a helpful starting point for someone else. YMMV.

The current status is WIP. <!--TODO: update status when finished -->

## System specifications

- Board: Dell G3 15 3590
- CPU: Intel i5-9300H @ 4.1GHz
- RAM: 16GB DDR4
- GPU0: Intel UHD Graphics 630
- GPU1: NVIDIA GeForce GTX 1660 Ti Mobile with Max-Q (6GB)
- NVME0: SkHynix 512GB SSD
- SATA1: Samsung 850 Pro 512GB SSD

## Guides

1. [Arch Linux installation](docs/1.Arch_Linux_installation/)
with `linux-zen` kernel
2. [GNOME DE](docs/2.Gnome_DE) running over Wayland
3. [Hybrid GPU configuration](docs/3.Hybrid_GPU_Configuration)
4. [Gaming](docs/4.Gaming) with Steam, GOG and Epic Games libraries
5. [Power management](docs/5.Power_management) for CPU and GPUs
6. Bypass GRUB using Unified Kernel Images booted up straight from the EFI firmware
(I absolutely **HATE** the screen going from the EFI firmware logo to text
interface to `plymouth`'s splash, turning off and on the display at every transition)
7. Bonus: being able to use TTYs on external monitors
8. Bonus: use LVFS with `fwupd` to enable EFI firmware updates from linux (see: https://fwupd.org/lvfs/devices/com.dell.uefi6482caa5.firmware)
9. Bonus: enable compressed zram instead of swap
10. Bonus: Small QoL things
11. Bonus: Add Ansible playbooks for everything

## Packages lists

Inside this repository, under the [packages](packages) folder, you can find my
own list of software packages, divided by category and by installation source
(`pacman` or `aur`). This repo is mainly for my own reference, but maybe some of
those packages can be helpful for you.

If you want to use them, clone this repository:

```bash
git clone https://github.com/Rick-1990/arch-optimus.git
```

Review the lists and customize them to your liking.

For example, to install the lists in the [packages/base](packages/base) folder, run:

```bash
cd arch-optimus
sudo pacman -S $(cat packages/base/pacman.list)
yay $(cat packages/base/aur.list)
```

> [!WARNING]
>
> Do not blindly install these packages. Review the lists and ask yoursef if
> you know what they do and if you actually need them.
> This is my own list for my own laptop for my own purposes.

## References

The always great [Arch Wiki's installation guide](https://wiki.archlinux.org/title/Installation_guide)
was the base for everything. Following that alone obviously leads to a usable
system in less than 30 minutes.

A major source of inspiration was [QuantiniumX's work](https://github.com/QuantiniumX/Guide-to-install-Arch-Linux).
This guide is heavily based on their great effort.

Also, [this gist](https://gist.github.com/LarryIsBetter/218fda4358565c431ba0e831665af3d1)
has helped me with some optimizations about laptops power management in general.
