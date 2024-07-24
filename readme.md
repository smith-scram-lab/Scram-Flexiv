------------------------------------------------------------------------------------------
+ # Current State
+ 2024-07-24 Yicheng start patching the new ubuntu20.04 native kernel according to the instructions listed in the [RT-Kernel-Patching](#RT-Kernel-Patching) section below
+ 2024-07-23 Yicheng installed a new ubuntu20.04 on another partition in the lab computer, name `scram-rt`
+ 2024-07-21 Yicheng discovered that the native RT kernel in ubuntu22.04 requires a paid subscription
- 2024-06-21 Yicheng have a half-finished CAD but I don't think it's necessary to move forward.
  -- I think it's okay to just get a board cut to be an ellipse which has its major axis as 1830mm, minor axis 1600mm.
+ 2024-06-21 Yicheng is planning on purchasing a new usb drive to install ubuntu22.04 which comes with a native RT kernel.
-
+
-
+
-
------------------------------------------------------------------------------------------



# Scram-Flexiv

Welcome to the Scram-Flexiv repository! This repository contains documentation, guides, and resources for using the Flexiv Rizon 4 robot in our lab.

## Table of Contents
- [Introduction](#introduction)
- [Getting Started](#getting-started)
- [Software Setup](#software-setup)
- [Safety Protocols](#safety-protocols)
- [User Guides](#user-guides)
- [Maintenance Schedule](#maintenance-schedule)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## Introduction
This repository contains all the necessary documentation to help you operate the Flexiv Rizon 4 robot. Whether you are a new user or an experienced operator, you'll find valuable information here to ensure safe and efficient use of the robot.

## Getting Started
Follow these steps to get started with the Flexiv Rizon 4 robot:

1. [Software Setup](#software-setup)
2. [Safety Protocols](#safety-protocols)
3. [User Guides](#user-guides)

## Software Setup
1. **Connect Teach Pendant**: Connect the teach pendant to the robot via Wi-Fi or a wired connection.
2. **Initial Configuration**: Follow the instructions in the [User Manual](https://rdk.flexiv.com/manual/index.html) to perform the initial configuration and test-run your first program.
3. **Flexiv Elements**: Use Flexiv Elements software to control the robot and execute programs. Refer to the [Training Video](path/to/Training_Video.pdf) for a detailed guide on using the software.

## Safety Protocols
Safety is paramount when operating the Flexiv Rizon 4 robot. Please refer to the [Safety Section](path/to/User_Manual.pdf#page=5) of the User Manual for detailed safety instructions and protocols.

- **Emergency Stop**: Ensure the emergency stop function is configured and tested.
- **Safety Zones**: Define and configure safety zones to prevent collisions and ensure operator safety.
- **Risk Assessment**: Conduct a thorough risk assessment before operating the robot.

## User Guides
- [Startup Guide](docs/guides/Flexiv_Rizon_Quick_Start_Guide_V3_202301.pdf)
- [Safety Protocol Document](docs/safety/safety_protocol.md)
- [Troubleshooting Tips](path/to/Troubleshooting.pdf)

## Maintenance Schedule
Regular maintenance is crucial to keep the robot in optimal working condition. Refer to the [Maintenance Schedule](path/to/Maintenance_Schedule.pdf) for detailed instructions on maintaining the robot.

## RT-Kernel-Patching 

### (written on 07/24/2024)
Followed the tutorial [here](https://mshields.name/blog/2023-08-30-preempt-rt-install-for-ubuntu-20-04/), some edits were made.

`sudo apt-get install build-essential bc ca-certificates gnupg2 libssl-dev wget gawk flex bison dwarves zstd`

`uname -r` --- 5.15.0-116-generic

Find the suitable version of patch [here](https://wiki.linuxfoundation.org/realtime/preempt_rt_versions)

Here I used 

`wget https://cdn.kernel.org/pub/linux/kernel/projects/rt/5.15/patch-5.15.160-rt77.patch.xz`

`wget https://cdn.kernel.org/pub/linux/kernel/projects/rt/5.15/patch-5.15.160-rt77.patch.sign`

`wget https://www.kernel.org/pub/linux/kernel/v5.x/linux-5.15.160.tar.xz`

`wget https://www.kernel.org/pub/linux/kernel/v5.x/linux-5.15.160.tar.sign`

`xz -dk patch-5.15.160-rt77.patch.xz`

`xz -d linux-5.15.160.tar.xz `

I skipped the verification step because the patch maintainer key listed in the instruction is no longer valid

Probably you still should do it if you are patient enough. :D

I installed `ncurses-dev` on the system at this point because some error were thrown

`sudo apt-get install libncurses-dev`

`tar xf linux-5.15.160.tar`

`cd linux-5.15.160`

`cp /boot/config-5.15.0-116-generic .config`

`make menuconfig`

At this point a graphic UI should be seen in your terminal

Activate “Fully Preemptible Kernel (Real-Time)” option from “General setup” / “Preemption Model” then SAVE and EXIT.(quote the [original tutorial](https://mshields.name/blog/2023-08-30-preempt-rt-install-for-ubuntu-20-04/))

Now a fix need to be done to the `.config` file, according to [this thread](https://stackoverflow.com/questions/67670169/compiling-kernel-gives-error-no-rule-to-make-target-debian-certs-debian-uefi-ce).

You can use whatever text editor, find the `Certificates for signature checking section` and change it to be looking like the following
```
#
# Certificates for signature checking
#
CONFIG_MODULE_SIG_KEY="certs/signing_key.pem"
CONFIG_MODULE_SIG_KEY_TYPE_RSA=y
CONFIG_MODULE_SIG_KEY_TYPE_ECDSA=y
CONFIG_SYSTEM_TRUSTED_KEYRING=y
CONFIG_SYSTEM_TRUSTED_KEYS="/usr/local/src/debian/canonical-certs.pem"
CONFIG_SYSTEM_EXTRA_CERTIFICATE=y
CONFIG_SYSTEM_EXTRA_CERTIFICATE_SIZE=4096
CONFIG_SECONDARY_TRUSTED_KEYRING=y
CONFIG_SYSTEM_BLACKLIST_KEYRING=y
CONFIG_SYSTEM_BLACKLIST_HASH_LIST=""
CONFIG_SYSTEM_REVOCATION_LIST=y
CONFIG_SYSTEM_REVOCATION_KEYS="/usr/local/src/debian/canonical-revoked-certs.pem"
# end of Certificates for signature checking

```
### Build and Install

`sudo make` And wait for a lonnnng time

Fix any error that might pop out

`sudo make modules_install`
`sudo make install`

### Real-Time User Privileges

```
sudo groupadd realtime
sudo usermod -aG realtime $(whoami)
```
Edit `/etc/security/limits.conf` to contain... (you might need admin access to edit it `sudo vi limits.conf`)

```
@realtime soft rtprio 99
@realtime soft priority 99
@realtime soft memlock 102400
@realtime hard rtprio 99
@realtime hard priority 99
@realtime hard memlock 102400
```

### Grub Settings
Edit `/etc/default/grub` (you might also need admin access to edit it `sudo vi grub`)

Change it to contain:
```
 GRUB_DEFAULT=saved
 GRUB_SAVEDEFAULT=true
```
Update Grub

`sudo update-grub`

Reboot and apply the changes

`sudo reboot`

### Check if the kernel is patched correctly
`uname -r` and look for 'rt' keyword


## Troubleshooting
Encountering issues? Check out our [Troubleshooting Guide](path/to/Troubleshooting.pdf) for solutions to common problems.

## Contributing
We welcome contributions from the community! If you would like to contribute to this repository, please fork the repo and create a pull request. For major changes, please open an issue first to discuss what you would like to change.

## License
This repository is licensed under the MIT License. See the [LICENSE](path/to/LICENSE) file for more details.

---
