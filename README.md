# Arch Installation from scratch - Optimus Laptop

## Introduction

This is a step by step guide about installing Arch on a laptop with dual-GPUs
(specifically Intel UHD + NVIDIA).
My personal objective is to have a nice gaming experience.

This guide wants to be a reference for myself if I need to do all this
again, and maybe a helpful starting point for someone else. YMMV.

The current status is WIP. <!--TODO: update status when finished -->

## References

The always great [Arch Wiki's installation guide](https://wiki.archlinux.org/title/Installation_guide)
was the base for everything. Following that alone obviously leads to a usable
system in less than 30 minutes.

A major source of inspiration was [QuantiniumX's work](https://github.com/QuantiniumX/Guide-to-install-Arch-Linux).
This guide is heavily based on their great effort.

Also, [this gist](https://gist.github.com/LarryIsBetter/218fda4358565c431ba0e831665af3d1)
has helped me with some optimizations about laptops power management in general.

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
2. GNOME DE (with my own selection of software), running over Wayland
3. Hybrid GPU configuration (*MAYBE* in the future I will configure the dGPU for
PCIe Passthrough to a VM, but not now)
4. Be able to launch games from Steam, GOG and Epic Games libraries, using Wine
(or one of its forks) if they do not provide a native Linux experience.
It has to be at least playable with good-enough performance
(I.E.: no framerate drops, at least 60 fps at medium settings...)
5. Full configuration of power management for CPU and GPUs (eg: being able to
put the laptop to sleep and resume, or automatically turn off the dGPU when not needed)
6. Bypass GRUB using Unified Kernel Images booted up straight from the EFI firmware
(I absolutely **HATE** the screen going from the EFI firmware logo to text
interface to `plymouth`'s splash, turning off and on the display at every transition)
7. Bonus: being able to use TTYs on external monitors (NVIDIA drivers do not
provide a framebuffer, so I believe this one will be hard)
8. Bonus: use LVFS with `fwupd` to enable EFI firmware updates from linux (see: https://fwupd.org/lvfs/devices/com.dell.uefi6482caa5.firmware)
9. Bonus: enable compressed zram instead of swap
10. Small QoL things
