# MallARD_Pixhwak
This repository is about controlling the 4 thrusters in MallARD via MAVROS using Pixhawks4. Two tasks needs to be set down. First is to compile the custom firmware for Pixhawk4 named Audusub and flash it into Pixhawk4. The second is installation of MAVROS and MAVLINK. The two submodules in this repository contain the instruction and source files of Ardusub firmware and MAVROS.

## 1. Autopilot_MALLARD
ArduPilot MALLARD (AP-M) is a customised ArduPilot firmware for MALLARD. It is built based on the current stable ArduSub release (ArduSub-4.0.3).

AP-M provides a *Custom* frame configuration adapted to the thruster allocation used on MALLARD. This frame configuration allows higher level motion command input such as *move_forward*, *turn_left*, etc. AP-M also provides two new frame configurations, i.e. *Joystick PWM Control* and *ROS PWM Control*, which enables sending a PWM signal directly to each individual motor by pushing a joystick or publishing a ROS topic, respectively. The *ROS PWM Control* frame assumes thruster allocation is dealt with within ROS.

### Hardware
For this build guide, we use a Pixhawk 4 and direct USB connection to a PC running Ubuntu.

### Build
#### Install git
In case you have not yet installed *git*, run the following commands in terminal:
```
sudo apt update
sudo apt install git
```

### Clone repository
Open a terminal and `cd` to our desired root folder for the repository, then clone the main [ArduPilot MALLARD](https://github.com/EEEManchester/ArduPilot_MALLARD) repository using your preferred authentication protocol.

* HTTPS:
```
git clone --recursive https://github.com/EEEManchester/ArduPilot_MALLARD.git
```
* SSH:
```
git clone --recursive git@github.com:EEEManchester/ArduPilot_MALLARD.git
```

Make sure you log into the correct GitHub account that has access to [EEEManchester](https://github.com/EEEManchester).

### Setup environment
[A script](https://github.com/ArduPilot/ardupilot/blob/master/Tools/environment_install/install-prereqs-ubuntu.sh) is provided to automatically setup your build environment.
```
cd ArduPilot_MALLARD
Tools/environment_install/install-prereqs-ubuntu.sh -y
```
> **Note:** If the end of the command promp is not showing something similar to `echo xxx end------`, the setup is unsuccessful. If it complains about `'some-python-package' has no installation candidate`, your system is probably configured for Python3. You need to run the follow instead:
> ```
> Tools/environment_install/install-prereqs-ubuntu-py3.sh -y
> ```

Reload the path (log-out and log-in to make permanent):
```
. ~/.profile
```

### Build with WAF
Make sure you are in ardupilot folder.

> **Note:** For Python3 users, you need to replace `waf` with `waf-py3`.

If you have previously built the firmware, you may want to clean WAF first:
```
./waf clean
```

Then
```
./waf configure --board Pixhawk4
./waf sub
```

After the code finishes, you can find the new firmware `ardusub.apj` in the `build/pixhawk4/bin` directory.

### Upload
#### Using WAF
There are two conditions for this to work:
1. with a direct USB connection to the Pixhawk
2. only after configuring and building with `waf` before
```
./waf --upload sub
```
#### Using GCS
* [Mission Planner](https://ardupilot.org/planner/docs/common-loading-chibios-firmware-onto-pixhawk.html#uploading-as-custom-firmware)
* [QGroundControl](https://docs.qgroundcontrol.com/master/en/SetupView/Firmware.html#loading-firmware)

---
#### Original guides
In ~~unlikely~~ situation that the above instructions do not work, you may want to check out the original ArduPilot guides:

* [Developers on ArduSub Wiki](http://www.ardusub.com/developers/developers.html)
* [Setting up the Build Environment (Linux/Ubuntu)](https://ardupilot.org/dev/docs/building-setup-linux.html)
* [BUILD.md](https://github.com/ArduPilot/ardupilot/blob/master/BUILD.md)

### USE
#### Frame selection
#### ROS

## 2. mavros_mallard
A custom MAVROS for MALLARD
### Source installation

Use `wstool` utility for retrieving sources and  [`catkin` tool](https://catkin-tools.readthedocs.io/en/latest/)for build.

NOTE: The source installation instructions are for the ROS Melodic release.

```sh
sudo apt-get install python-catkin-tools python-rosinstall-generator -y
# For Noetic use that:
# sudo apt install python3-catkin-tools python3-rosinstall-generator python3-osrf-pycommon -y

# 1. Create the workspace: unneeded if you already has workspace
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws
catkin init
wstool init src

# 2. Install MAVLink
#    we use the Kinetic reference for all ROS distros as it's not distro-specific and up to date
rosinstall_generator --rosdistro melodic mavlink | tee /tmp/mavros.rosinstall

# 3. Install MAVROS: get source (upstream - released)
rosinstall_generator --upstream mavros | tee -a /tmp/mavros.rosinstall
# alternative: latest source
# rosinstall_generator --upstream-development mavros | tee -a /tmp/mavros.rosinstall
# For fetching all the dependencies into your catkin_ws, just add '--deps' to the above scripts
# ex: rosinstall_generator --upstream mavros --deps | tee -a /tmp/mavros.rosinstall

# 4. Create workspace & deps
wstool merge -t src /tmp/mavros.rosinstall
gedit ~/catkin_ws/src/.rosinstall
change 
'- git:
    local-name: mavros
    uri: https://github.com/mavlink/mavros.git
    version: 1.8.0'
to
'- git:
    local-name: mavros_mallard
    uri: https://github.com/EEEManchester/mavros_mallard.git
    version: dev'
wstool update -t src -j4
rosdep install --from-paths src --ignore-src -y

# 5. Install GeographicLib datasets:
./src/mavros/mavros/scripts/install_geographiclib_datasets.sh

# 6. Build source
catkin build

# 7. Make sure that you use setup.bash or setup.zsh from workspace.
#    Else rosrun can't find nodes from this workspace.
source devel/setup.bash
```
## QGroundControl Ground Control Station

[![Releases](https://img.shields.io/github/release/mavlink/QGroundControl.svg)](https://github.com/mavlink/QGroundControl/releases)
[![Travis Build Status](https://travis-ci.org/mavlink/qgroundcontrol.svg?branch=master)](https://travis-ci.org/mavlink/qgroundcontrol)
[![Appveyor Build Status](https://ci.appveyor.com/api/projects/status/crxcm4qayejuvh6c/branch/master?svg=true)](https://ci.appveyor.com/project/mavlink/qgroundcontrol)

[![Gitter](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/mavlink/qgroundcontrol?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)


*QGroundControl* (QGC) is an intuitive and powerful ground control station (GCS) for UAVs.

The primary goal of QGC is ease of use for both first time and professional users. 
It provides full flight control and mission planning for any MAVLink enabled drone, and vehicle setup for both PX4 and ArduPilot powered UAVs. Instructions for *using QGroundControl* are provided in the [User Manual](https://docs.qgroundcontrol.com/en/) (you may not need them because the UI is very intuitive!)

All the code is open-source, so you can contribute and evolve it as you want. 
The [Developer Guide](https://dev.qgroundcontrol.com/en/) explains how to [build](https://dev.qgroundcontrol.com/en/getting_started/) and extend QGC.


Key Links: 
* [Website](http://qgroundcontrol.com) (qgroundcontrol.com)
* [User Manual](https://docs.qgroundcontrol.com/en/)
* [Developer Guide](https://dev.qgroundcontrol.com/en/)
* [Discussion/Support](https://docs.qgroundcontrol.com/en/Support/Support.html)
* [Contributing](https://dev.qgroundcontrol.com/en/contribute/)
* [License](https://github.com/mavlink/qgroundcontrol/blob/master/COPYING.md)
