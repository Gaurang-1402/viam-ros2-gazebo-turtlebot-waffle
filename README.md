# viam-ros2-gazebo-turtlebot-waffle

![ezgif com-video-to-gif](https://github.com/Gaurang-1402/viam-ros2-gazebo-turtlebot-waffle/assets/71042887/c83a330f-1044-44df-91d1-ac8bcf1a77fd)



Some of these instructions have been adapted from

https://github.com/viam-soleng/viam-ros2-integration/tree/main

## Simulation setup

```
mkdir turtlebot_ws
cd turtlebot_ws
git clone https://github.com/Gaurang-1402/viam-ros2-gazebo-turtlebot-waffle/
```

## Requirements
1. viam account 
2. viam organization 
3. viam location
4. ROS2 Humble
5. python venv module installed



## Viam local Install
After creating you account you can follow these [instructions](https://docs.viam.com/installation/) to 
install Viam on your ROS2 robot.


### Preparing ROS2

With ROS2 and the use of DDS, we might need to configure our DDS instance to ensure Viam can successfully
connect to the ROS2 system. When using fastdds we might be using shared memory as the transport, if this 
is the case the Viam module will not be able to connect to the ROS2 node by default.

Currently, modules start as `root` by default[^1] which causes issue with shared memory as ROS2 typically
runs as a non-root user.  To compensate for this Viam has provided a [sample config](./sample_configs/fastdds_rpi.xml)
which shows how to configure the `UDP4` transport.

Based on the configuration you are using replace the contents of the xml file for your ROS2 DDS configuration
with the ones found in the link above.

We first go to the etc folder

```shell
cd /etc
mkdir turtlebot
```

Make a new directory in /etc

```shell
cd /etc
mkdir turtlebot
cd turtlebot
nano setup.sh
```

Add the following code

```
export FASTRTPS_DEFAULT_PROFILES_FILE=/etc/turtlebot/fastdds_rpi.xml
export ROBOT_NAMESPACE=/tb4lite
export ROS_DOMAIN_ID=0
export ROS_DISCOVERY_SERVER=
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
export TURTLEBOT4_DIAGNOSTICS=1
export WORKSPACE_SETUP=/opt/ros/humble/setup.bash

```
Save and exit

We need to also create the fastdds_rpi.xml. To do this, we run,

```shell

  GNU nano 6.2                                                 fastdds_rpi.xml                                                          
<?xml version="1.0" encoding="UTF-8" ?>
<dds>
    <profiles xmlns="http://www.eprosima.com/XMLSchemas/fastRTPS_Profiles" >
        <transport_descriptors>
            <transport_descriptor>
                <transport_id>CustomUdpTransport</transport_id>
                <type>UDPv4</type>
            </transport_descriptor>
        </transport_descriptors>

        <participant profile_name="turtlebot4_default_profile" is_default_profile="true">
            <rtps>
                <userTransports>
                    <transport_id>CustomUdpTransport</transport_id>
                </userTransports>

                <useBuiltinTransports>false</useBuiltinTransports>
            </rtps>
        </participant>
    </profiles>
</dds>

```

Save and Exit

### Configuration Files

Viam has provided an example configure [here](./sample_configs/setup.bash)

This file should contain you ROS2 environment scripts as well as the `VIAM_NODE_NAME` which is used by
the python process to start the viam integration.


```shell
cd /etc
mkdir viam
```

Make a new directory in /etc

```shell
cd /etc
mkdir viam
cd viam
nano setup.sh
```


Now paste the following code

```
#
# example file to be used in the /etc/viam/setup.bash file
#

# source required ROS variables
# this exposes all variables to viam module process
. /etc/turtlebot/setup.bash
. /opt/ros/humble/setup.bash

# set environment variables as needed
export VIAM_NODE_NAME="viam"
export VIAM_ROS_NAMESPACE=""


```
Save and Exit

## Viam Configuration

On the Config tab in Viam, under Components, Create component and add ``` viam-soleng:ros2:base ``` plus ``` viam-soleng:ros2:camera ```

![Screenshot from 2023-11-15 18-15-05](https://github.com/Gaurang-1402/viam-ros2-gazebo-turtlebot-waffle/assets/71042887/52d7ca77-c93f-46b7-a850-e8b56da4f464)


Enter the following details

```
{
  "ros_topic": "/camera/image_raw"
}

```
and

```
{
  "publish_rate": "0.9",
  "ros_topic": "/cmd_vel"
}
```


## Credits
Simulation adapted from: 
- https://github.com/ROBOTIS-GIT/turtlebot3_simulations.git -b humble-devel
- https://github.com/ROBOTIS-GIT/turtlebot3.git -b humble-devel
- https://github.com/ROBOTIS-GIT/turtlebot3_msgs.git -b humble-devel
- https://github.com/ROBOTIS-GIT/DynamixelSDK.git -b humble-devel

https://medium.com/@nilutpolkashyap/setting-up-turtlebot3-simulation-in-ros-2-humble-hawksbill-70a6fcdaf5de

https://www.viam.com/post/creating-a-ros2-integration-module-and-deploying-to-a-robot
I am deeply appreciative of these individuals/teams for sharing their work to build on top of!



