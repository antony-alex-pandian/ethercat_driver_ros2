## POC Hardware Nodes - Requirements Setup 

### Conveyor Plugin and Emergency Alarm Node Requirements 

freeopcua C++ - An LGPL OPC-UA server and client library written in C++. 

#### Installation 

Clone the repo using vcs import. 
```
cd ~/ws_fmms/src 
vcs import . < fmms/fmms_poc/control_center.repos 
```

Install the dependencies specified in debian.soft in the thirdparty/freeopcua folder. 
```
cd ~/ws_fmms 
sudo ./src/thirdparty/freeopcua/debian.soft 
 ```

The freeopcua package can now be built along with the other packages using colcon build. 
```
cd ~/ws_fmms 
colcon build --mixin release lld 
 ```

### Detection Node Requirements 

SQLiteCpp - An encapsulation around the native C APIs of SQLite (SQLite3 Wrapper). 

yaml-cpp - A YAML parser and emitter in C++. 

#### Installation 

Clone the repo using vcs import, depending on the workcell. 
```
cd ~/ws_fmms/src 
vcs import . < fmms/fmms_poc/workcell_1.repos 
(or) 
vcs import . < fmms/fmms_poc/workcell_2.repos 
(or) 
vcs import . < fmms/fmms_poc/workcell_3.repos 
 ```

Use rosdep command to install the yaml-cpp package, which is specified in package.xml. 

rosdep install --from-paths src --ignore-src --rosdistro humble -yr 

The SQLiteCpp package can now be built along with the other packages using colcon build. 
```
cd ~/ws_fmms 
colcon build --mixin release lld 
 ```

### MES Node Requirements 

libjsoncpp-dev - A library for reading and writing JSON for C++. 

mosquitto - A message broker that implements the MQTT protocol. 

mosquittopp-dev - A mosquitto client C++ wrapper class for the mosquitto C library. 

#### Installation 

Use rosdep command to install the above-mentioned packages, which are specified in package.xml. 
```
rosdep install --from-paths src --ignore-src --rosdistro humble -yr 
 ```
 
### Gripper and Label Printer/Dispenser Nodes Requirements 

[EtherLab IgH EtherCAT Master](https://gitlab.com/etherlab.org/ethercat) - An open-source EtherCAT master implementation for Linux. 

[EtherCAT Driver ROS2](https://github.com/ICube-Robotics/ethercat_driver_ros2) - Implementation of a Hardware Interface for simple Ethercat module integration with ros2_control and building upon IgH EtherCAT Master for Linux. 

[GPIO Controllers](https://github.com/mcbed/ros2_controllers/tree/gpio_controllers/gpio_controllers) - Collection of controllers for gpios that work with multiple interfaces. 

[Custom Interfaces](https://github.com/ros2torial/custom_interfaces) - Custom interfaces for testing purposes (optional). 

[GPIO Control](https://github.com/ros2torial/ros2_ethercat/tree/main/gpio_control) - Contains the EtherCAT device configs and urdfs (xacros). 

#### Installation 

Install the EtherLab IgH EtherCAT Master based on the instructions given here. If this link doesn't work, follow these instructions, only the EtherLab installation section. 


Clone the repo containing EtherCAT driver, GPIO Controllers, Custom Interfaces and GPIO control package using vcs import, depending on the workcell. 
```
cd ~/ws_fmms/src 
vcs import . < fmms/fmms_poc/workcell_2.repos 
(or) 
vcs import . < fmms/fmms_poc/workcell_3.repos 
 ```

Use rosdep command to install the necessary packages like control_msgs, which are specified in package.xml. 
```
rosdep install --from-paths src --ignore-src --rosdistro humble -yr  
```

The EtherCAT packages mentioned above can now be built along with the other packages using colcon build. 
```
cd ~/ws_fmms 
colcon build --mixin release lld 
```
 

### IgH EtherCAT Master 
```
git clone git@gitlab.com:etherlab.org/ethercat.git 
```
#### Building and installing 
 ```
./bootstrap # to create the configure script, if downloaded from the repo 
 
./configure --sysconfdir=/etc 
make all modules 
```
and as root: 
```
make modules_install install 
depmod 
 ```
 
and then customizing the appropriate configuration file:
```
vi /etc/ethercat.conf      # For systemd based distro 
vi /etc/sysconfig/ethercat # For init.d based distro 
 ```
Make sure, that the 'udev' package is installed, to automatically create the 
EtherCAT character devices. The character devices will be created with mode 
0660 and group root by default. If you want to give normal users reading 
access, create a udev rule like this: 
 ```
echo KERNEL==\"EtherCAT[0-9]*\", MODE=\"0664\" > /etc/udev/rules.d/99-EtherCAT.rules 
```
 
Now you can start the EtherCAT master: 
```
systemctl start ethercat   # For systemd based distro 
/etc/init.d/ethercat start # For init.d based distro 
 ```

### EtherLab 

#### Installation

The proposed development builds upon the IgH EtherCAT Master. Installation steps are summarized here: 

Install required tools: 
```
$ sudo apt-get update 
$ sudo apt-get upgrade 
$ sudo apt-get install git autoconf libtool pkg-config make build-essential net-tools 
```
Setup sources for the EtherCAT Master: 
```
$ git clone https://gitlab.com/etherlab.org/ethercat.git 
$ cd ethercat 
$ git checkout stable-1.5 
$ sudo rm /usr/bin/ethercat 
$ sudo rm /etc/init.d/ethercat 
$ ./bootstrap  # to create the configure script 
```
Configure, build and install libs and kernel modules: 
```
$ ./configure --prefix=/usr/local/etherlab  --disable-8139too --disable-eoe --enable-generic 
$ make all modules 
$ sudo make modules_install install 
$ sudo depmod 
```
NOTE: This step is needed every time the Linux kernel is updated. 

Configure system: 
```
$ sudo ln -s /usr/local/etherlab/bin/ethercat /usr/bin/ 
$ sudo ln -s /usr/local/etherlab/etc/init.d/ethercat /etc/init.d/ethercat 
$ sudo mkdir -p /etc/sysconfig 
$ sudo cp /usr/local/etherlab/etc/sysconfig/ethercat /etc/sysconfig/ethercat 
```
Create a new udev rule: 
```
$ sudo gedit /etc/udev/rules.d/99-EtherCAT.rules 
```
containing: 
```
KERNEL=="EtherCAT[0-9]*", MODE="0664" 
```
Configure the network adapter for EtherCAT: 
```
$ sudo gedit /etc/sysconfig/ethercat 
```
In the configuration file specify the mac address of the network card to be used and its driver 

MASTER0_DEVICE="ff:ff:ff:ff:ff:ff"  # mac address 
DEVICE_MODULES="generic" 

Now you can start the EtherCAT master: 
```
$ sudo /etc/init.d/ethercat start 
```
it should print 

Starting EtherCAT master 1.5.2  done 

You can check connected slaves: 
```
$ ethercat slaves 
```
It should print information of connected slave device: 
```
<id>  <alias>:<position>  <device_state>  +  <device_name> 
```
Example: 

0  0:0  PREOP  +  <device_0_name> 
0  0:1  PREOP  +  <device_1_name> 

 

### ethercat_driver_ros2 

#### Building
Install ros2 packages. The current development is based of ros2 humble. Installation steps are described here. 

Source your ros2 environment: 
```
source /opt/ros/humble/setup.bash 
```
NOTE: The ros2 environment needs to be sources in every used terminal. If only one distribution of ros2 is used, it can be added to the ~/.bashrc file. 

Install colcon and its extensions : 
```
sudo apt install python3-colcon-common-extensions 
```
Create a new ros2 workspace: 
```
mkdir ~/ros2_ws/src 
```
Pull relevant packages, install dependencies, compile, and source the workspace by using: 
```
cd ~/ros2_ws 
git clone https://github.com/ICube-Robotics/ethercat_driver_ros2.git src/ethercat_driver_ros2 
rosdep install --ignore-src --from-paths . -y -r 
colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release --symlink-install 
source install/setup.bash 
 ```

### gpio_controllers 

This is a collection of controllers for gpios that work with multiple interfaces. 

#### Hardware interface type 

These controllers work with gpios using user defined command interfaces. 

#### Using GPIO Command Controller 

The controller expects at least one gpio interface abd the corresponding command interface names. A yaml file for using it could be: .. code-block:: yaml 

#### controller_manager: 
```
ros__parameters: 

update_rate: 100 # Hz 
joint_state_broadcaster: type: joint_state_broadcaster/JointStateBroadcaster 
gpio_command_controller: type: gpio_controllers/GpioCommandController 

gpio_command_controller: 
ros__parameters: 
gpios: 
  Gpio1 
  Gpio2 
command_interfaces: 
Gpio1: 
  dig.1 
  dig.2 
  dig.3 

Gpio2: 
  ana.1 
  ana.2 
```

 
