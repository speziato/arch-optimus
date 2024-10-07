# Power Management

In this section you will configure power management options to ensure that the suspend/resume
cycles work as they should, and also apply some tweaks to the dGPU to make sure it's
really unloaded when not in use.

## Suspend method

The goal here is to select an appropriate method for suspending the laptop's state.
Also, personally I find that hibernation is useless because I have a fast SSD and
cold boot is fast enough for me, so I'm going to show how to disable hibernation
completely.

First, check what the current method is:

```bash
cat /sys/power/mem_sleep
```

The selected method is sorrounded with square brackes, e.g.:

```plaintext
[s2idle] deep
```

See the [Arch Wiki Power Management/Suspend and hibernate](https://wiki.archlinux.org/title/Power_management/Suspend_and_hibernate)
page to see what the values mean. In my case, suspend to RAM (so the `deep` method)
is not working well, as it does not power off the screens. If the selected method
is already `s2idle` you're good, otherwise you can do:

```bash
sudo echo "s2idle" > /sys/power/mem_sleep
```

and try a few suspend/resume cycles to check if it works correctly.
To make the change permanent, and to also disable hibernation, edit the `/etc/systemd/sleep.conf`
(or create a drop-in conf file in the `/etc/systemd/sleep.conf.d` folder) with the
following content:

```ini
[Sleep]
MemorySleepMode=s2idle
AllowHibernation=no
AllowSuspendThenHibernate=no
AllowHybridSleep=no
```

## ACPI Events

This section focuses on handling opening/closing the laptop lid.
Systemd allows for some fine-grained customization (e.g.: different handling
for the lid if the laptop is charging vs not charging vs external screen attached)
just by editing the `/etc/systemd/login.conf` file (or a drop-in conf file in the
`/etc/systemd/login.conf.d` folder).

In my I case I wanted to completely ignore the lid closing actions when using
external screens (with or without AC power) and when on AC power, because I
use my laptop almost always with the lid closed as I plugged 2 external displays.

This is how the configuration looks like to achieve the above behaviour:

```ini
[Login]
HandleLidSwitch=suspend
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```

See the comments in the `/etc/systemd/login.conf` file to see other available options.

## Power profile and timeouts

To enable customization of power profiles and timeouts for screen-off and suspend,
run the following command:

```bash
sudo pacman -S gnome-power-manager power-profiles-daemon
```

Now you can find the "Power" section in the Settings application.

## Kernel modules

Some kernel modules can be disabled to increase performance and lower power consumption.
For example, the hardware watchdog provided by Intel can be safely disabled. Also,
if you don't plan to use the HDMI audio, you can disable another module.

Disabling modules has also the side effect of reducing boot time, so in this case
it's a win-win.

To disable both of those modules, you can run:

```bash
cat <<EOF | sudo tee /etc/modprobe.d/disable-intel-modules.conf > /dev/null
blacklist iTCO_wdt
blacklist snd_hda_codec_hdmi
EOF
```

You should also add `nowatchdog` to the boot parameters in `/etc/default/grub` to
disable software watchdogs.

## NVIDIA GPU

If you followed the [Hybrid GPU Configuration](../3.Hybrid_GPU_Configuration/)
guide you don't need anything more.

The TL;DR is:

- enable NVIDIA services:

  ```bash
  sudo systemctl enable nvidia-{suspend,resume,hibernate,powerd}
  ```

- check if your card supports DynamicPowerManagement [here](https://us.download.nvidia.com/XFree86/Linux-x86_64/560.35.03/README/dynamicpowermanagement.html#SupportedConfigaffb4)

  - if it does, make sure you are setting the kernel parameter `NVreg_DynamicPowerManagement`
    to an appropriate value (`0x02` or `0x03`). This is automated by [Envycontrol](https://github.com/bayasdev/envycontrol)
    running:

    ```bash
    sudo envycontrol -s hybrid --rtd3 [2 or 3]
    ```

[Go back to index](../#guides)
