------------------------------------------------------------------------------------------
+ # Current State
+ 
+ 2024-07-24 Yicheng is now updating the software setup. 
+ 2024-07-24 The RT kernel is patched successfully. `5.15.160-rt77`
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
2. **Initial Configuration**: Follow the instructions in the [User Manual](https://rdk.flexiv.com/manual/index.html) to perform the initial configuration and test-run your first program. Step by step guide can be found in this [section](#step-by-step-software)
4. **Flexiv Elements**: Use Flexiv Elements software to control the robot and execute programs. Refer to the [Training Video](path/to/Training_Video.pdf) for a detailed guide on using the software.

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

------------------------------------------

## Step By Step Software
I'm a big conda user and I encourage you to do the same. :D

### Conda installation
Thus, before we install anything, install [Anaconda](https://docs.anaconda.com/anaconda/install/linux/). Follow [this guide](https://docs.anaconda.com/anaconda/install/linux/).

The conda distribution used here is `Anaconda3-2024.06-1-Linux-x86_64.sh`

### Conda env create and activate

`conda create -n flexiv python=3.10` (why 3.10? just because, also because in the manual the default python version is 3.10)
`conda activate flexiv`

### [C++ RDK](https://rdk.flexiv.com/manual/install_on_linux.html#c-rdk)
#### Prepare build tools
This I just use system wide installation
`sudo apt install build-essential cmake cmake-qt-gui -y` 

#### Download RDK
------Written on 07/24/2024------
Obviously you need to install `git` if you still don't have it
`sudo apt-get install git`
Go to the directory that you want to build this RDK
```
git clone https://github.com/flexivrobotics/flexiv_rdk.git
cd flexiv_rdk
git checkout v1.4
```
#### Install C++ dependencies
Choose a directory, in my case, `/flexiv_rdk_cpp`
```
cd flexiv_rdk/thirdparty
bash build_and_install_dependencies.sh ~/flexiv_rdk_cpp
```
#### Install C++ RDK
After all dependencies are installed, open a new terminal and use CMake to confiure the flexiv_rdk project
```
cd flexiv_rdk
mkdir build && cd build
cmake .. -DCMAKE_INSTALL_PREFIX=~/flexiv_rdk_cpp
cmake --build . --target install --config Release

```
If the installation was successful, you can find these following directories in the `/flexiv_rdk_cpp` folder
```
include/flexiv/rdk/
lib/cmake/flexiv_rdk/
```
#### Link to C++ RDK from a user program
*I copied the following from the manual, not sure what to do here. But I think if this can run successfully, your installation is correct. BRAVO!*
After C++ RDK is installed, it can be found as a CMake library and linked to by other CMake projects. Use the provided examples project for instance:
```
cd flexiv_rdk/example
mkdir build && cd build
cmake .. -DCMAKE_INSTALL_PREFIX=~/flexiv_rdk_cpp
cmake --build . --config Release -j 4
```

### Python RDK
### Dependencies
Here according to the manual we want to install numpy and spdlog
`pip install numpy spdlog`

### Build Flexiv RDK
**If you didn't follow the instruction in C++ building, go check on it and see what you need, where to get the rdk repository, don't skip to here. I assume you are following this STEP-BY-STEP**
Build in another directory, in this case I choose `/flexiv_rdk_python` **obviously you need to `mkdir flexiv_rdk_python`**
```
cd flexiv_rdk/build
cmake .. -DCMAKE_INSTALL_PREFIX=~/flexiv_rdk_python/ -DINSTALL_PYTHON_RDK=ON
```
If you encounter an issue saying the python version is not 3.10, go check if you activated your conda env.
BECAUSE the lines above will install rdk for 3.10 by default
Check the original manual for other python versions.

### Install Python RDK
```
cd flexiv_rdk/build
cmake --build . --target install --config Release

```
### Try it
if the installation is successful, you should be able to do this 

```
python3
import flexivrdk
```


## Troubleshooting
Encountering issues? Check out our [Troubleshooting Guide](path/to/Troubleshooting.pdf) for solutions to common problems.

## Contributing
We welcome contributions from the community! If you would like to contribute to this repository, please fork the repo and create a pull request. For major changes, please open an issue first to discuss what you would like to change.

## License
This repository is licensed under the MIT License. See the [LICENSE](path/to/LICENSE) file for more details.

---
