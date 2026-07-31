# Tunnel Navigation Program

## Table of Contents
1. [Purpose](#purpose)
2. [Installation and Usage](#installation-and-usage)
    <br>&nbsp;&nbsp;2.1. [System Requirements](#system-requirements)
    <br>&nbsp;&nbsp;2.2. [Software Prerequisites](#software-prerequisites)
    <br>&nbsp;&nbsp;2.3. [Hardware Requirements](#hardware-requirements)
    <br>&nbsp;&nbsp;2.4. [Usage](#usage)
3. [Basic Code Overview](#basic-code-overview)
    <br>&nbsp;&nbsp;3.1. [Jetson Setup](#jetson-setup)
    <br>&nbsp;&nbsp;3.2. [Robot Setup](#robot-setup)
    <br>&nbsp;&nbsp;3.3. [LiDAR Setup](#lidar-setup)
       <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3.3.1. [LiDAR Data Handling Overview](#lidar-data-handling-overview)
    <br>&nbsp;&nbsp;3.4. [Navigation](#navigation)
       <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3.4.1. [Navigation Loop Overview](#navigation-loop-overview)
       <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3.4.2. [Dealing with Blind Spots](#dealing-with-blind-spots)
       <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3.4.3. [Blank Sections](#blank-sections)
       <br>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3.4.4. [Preventing Drift](#preventing-drift)
4. [Troubleshooting](#troubleshooting)
    <br>&nbsp;&nbsp;4.1. [LiDAR Issues](#lidar-issues)

## Purpose

The program is designed to allow a Jetson Orin Nano to control a [UGV01](https://www.waveshare.com/wiki/UGV01) to navigate through a tunnel, using a [Livox Avia LiDAR](https://www.livoxtech.com/avia) in order to see obstacles so that it can avoid them. Two ultrasonic sensors are also placed on the robot, one on each side, to assist with navigation. 

While a different robot or LiDAR can be used, some alteration to the code will be necessary.

The program can either communicate with the UGV01 by connecting the Jetson to the ESP32 through the 40 pin UART, or by connecting to the UGV01's hotspot.

Data can be recorded as the UGV01 moves throughout the tunnel, allowing for a scan of the walls of the tunnel to be completed as the robot moves from start to end and returns. This can assist in identifying cracks and structural issues in the tunnel without needing a human to enter the tunnel at all.

## Installation and Usage

### System Requirements

- **[ROS 2 Humble](https://docs.ros.org/en/humble/Installation/Ubuntu-Install-Debs.html)** installed on your system.
- **Python 3.x** with `pip` configured.

### Software Prerequisites

To use the program, the following must be installed:

- A driver that can read and publish point cloud data in the pointcloud2 format (for the Avia **[ASIG-X's driver](https://github.com/ASIG-X/livox_ros2_avia)** works very well)
- **[UGV Jetson](https://github.com/waveshareteam/ugv_jetson/tree/main)**
- **Jupyter** (comes with UGV Jetson)

You can run this command to install all of the dependencies with `pip`:

`python -m pip install pyserial numpy altair pandas requests pynput ipython ipywidgets`

### Hardware Requirements

To use the program, the following sensors need to be affixed to the robot:

- One **Livox AVIA LiDAR**, which should be mounted on the front of the robot
- Two **ultrasonic sensors**, one on each side of the robot (connected to the 40 pin UART, the trig and echo pins should be 29 and 31 for the first sensor and 33 and 32 for the second)

### Usage

Before using the program, ensure the LiDAR and the UGV01 are connected to the Jetson (the UGV01 does not need to be connected if using the hotspot to connect).

To use the program, open up the LiDAR driver and begin publishing point cloud data (ex: if using ASIG-X's driver you would enter your workspace in another terminal and run `ros2 launch livox_ros2_avia livox_lidar_launch.py`). If you are using ASIG-X's driver, skip this step as it will run automatically when the code is started. If opening the driver automatically, make sure to click your mouse after the driver is completely started as the Jetson will wait for an input so that the code doesn't run before the driver is ready.

Enter the ugv_jetson directory (`cd ugv_jetson`) and run the file from in there (either using `jupyter lab` or `jupyter notebook` to open the file and then run all cells). Make sure the file is placed in the ugv_jetson folder beforehand which should be created after installing all the software prerequisites.

The program should stop right before the navigation loop and wait for a mouse button to be pressed. Alternatively, it can be set to wait for a command to be send to the UGV01 by an external device through its hotspot. Once the command is recieved, the robot will immediately begin navigating through the tunnel.

Make sure to place the robot in the center of the entrance, facing as straight forward as possible.

## Basic Code Overview

### Jetson Setup

- Defines wait for input function
- Opens the lidar driver automatically

If using the livox_ros2_avia driver from ASIG-X previously mentioned, no change to the code is needed. However, if not using this driver, setupAvia MUST be set to false and whatever LiDAR driver you are using must be started manually.

Note that the program will wait for an input to indicate that the driver has finished starting before continuing.

### Robot Setup

- Lets the Jetson know whether to communicate with the robot through the serial port (if connected through 40 pin UART) or through the web interface (if connected through hotspot)
- Sets pins for ultrasonic sensor
- Defines basic sensor related functions
- Sets up GPIO for ultrasonic and defines ultrasonic related function
- Defines basic movement functions

### LiDAR Setup

- Sets up rclpy node to read LiDAR data
- Defines basic data collection functions for LiDAR
- Defines data interpretation functions for LiDAR

#### LiDAR Data Handling Overview

The LiDAR data is published as a point cloud, or a list of points where each point represents an object that the LiDAR has been detected. This list is split into several different lists, with each one containing points from a certain area. There can be any amount of these sections, but there is a base number of 7. Each section is about as big as the robot. 

The closest point in each section is then identified.

<p align="center"><img width="800" height="450" alt="scanning" src="https://github.com/user-attachments/assets/f663da8e-1923-4900-a3d2-b70f8a768cb9" /></p>

The idea is that by looking at the closest point in each section, we can see which sections have obstacles nearby and how close they are. Since each section is about the size of the robot, we can easily have the robot avoid blockages by just moving to a clear section from a blocked one and simply going forward.

### Navigation

- Defines weights and navigation parameters
- Defines more advanced sensor/movment related functions used for navigation
- Creates navigation loop

#### Navigation Loop Overview

The sections created with the LiDAR data are quickly put to use the navigation loop. The robot identifies obstacles and moves around them, continuously scanning and rerouting.

<p align="center"><img width="800" height="450" alt="navigating" src="https://github.com/user-attachments/assets/88c0536a-c104-4d66-892f-547983c761d3" /></p>

This continues until the robot detects it has reached a dead end, where it then turns around and navigates until it reaches the point where it started from. The robot also uses various sensors like the ultrasonic and motor encoders in an attempt to negate drift.

#### Dealing with Blind Spots

The Livox Avia LiDAR is not suitable for a navigation program like this. The most glaringly obvious reason for this is the massive 1 meter blind spot which prevents the LiDAR from seeing anything close to the robot. This means the navigation program must see the obstacles and move to the proper sections over 1 meter away from them.

However, in order to not set the Jetson aflame with the most intensive program ever, the data is only read from the LiDAR periodically and is not saved. Each time a new measurement is taken, the navigation loop is run completely anew independently of any previous runs. This means that there is a possibility that there is an obstacle that the robot "forgets" is there as it moves into the blind spot, leading the robot to run into the obstacle.

<p align="center"><img width="800" height="450" alt="cantsee" src="https://github.com/user-attachments/assets/a075f681-778c-49ae-a4c9-152895900a4c" /></p>

To solve this, an 2d array is created that stores the location of obstacles, as well as their position in x and y coordinates (where x is forward and y is sideways). This way, we can know the position of obstacles accurately even if the robot moves left or right, and remove them accordingly from the list when we go past them.

This way, we can remember where obstacles are and avoid them.

<p align="center"><img width="800" height="450" alt="avoid" src="https://github.com/user-attachments/assets/8fcf60af-b1b0-4cf8-b1e3-e25521426772" /></p>

#### Blank Sections

In the gif showcasing an example navigation loop in the **Navigation Loop Overview** section, it can be seen that walls are seen as obstacles that are 0 meters away. This is because the walls fill the entirety of the blind spot in that section, making it so that no points are read. While this would normally just throw an exception, the robot is programmed to treat these sections with no points as walls.

<p align="center"><img width="800" height="450" alt="idwall" src="https://github.com/user-attachments/assets/01945d75-efc5-4e51-bf4e-c88cdea491d0" /></p>

However, in the case that there is simply no obstacle for as far as the LiDAR can see, the sections will similarly not contain any points.

<p align="center"><img width="800" height="450" alt="blindrobot" src="https://github.com/user-attachments/assets/930dbe68-fbd4-41f9-bd33-2d011e595ce0" /></p>

This is problematic as in these cases, the robot will assume that its path is completely blocked. Therefore, a method must be devised to determine if the section is actually blocked or if there is just nothing that the sensor can see. For this, the ultrasonic sensors on the sides of the robot can be used. Since the ultrasonic can see how far away the walls are, it can identify which section are within the walls.

<p align="center"><img width="800" height="450" alt="canseenow" src="https://github.com/user-attachments/assets/e12aad5c-7af6-4314-bc49-9086697dd21d" /></p>

This ignores a critical detail though. If an obstacle is in the blind spot of the LiDAR, and is large enough to cover the whole section, there will similarly be 0 points logged. Based on the criteria that we set earlier, the robot will identify this blocked section as a clear one and could attempt to move to it. However, this is easily resolvable. Using the blocked section list from earlier, we can easily find out if this is the case.

<p align="center"><img width="800" height="450" alt="rememberobstacle" src="https://github.com/user-attachments/assets/b67ae148-1e2f-40c3-afff-ea16ad073d85" /></p>

#### Preventing Drift

This program is extremely dependent on moving in a precise and consistent way. If the robot is moving at an angle, it could drift into an adjacent blocked section from the clear section that it is supposed to be travelling in. This becomes very apparent as it moves to avoid obstacles extremely far back from them, which gives ample time for drift to become a serious issue. This is due to the data from the LiDAR having a lot of variation, making a large margin for error a necessity.

Since the robot will need to travel all the way down long tunnels and back, some sort of measure to prevent drift is necessary.

This is where the ultrasonic sensor comes in handy once again. Through comparing the distance of the left and right walls against previously saved distances as the robot moves forward, we can easily identify if the robot is drifting and correct.

<p align="center"><img width="800" height="450" alt="ultrasonicstraighten" src="https://github.com/user-attachments/assets/954e77e0-1280-4c78-85ec-22149bfaa90e" /></p>

By constantly taking these measurements while moving, the robot can correct as it goes and ensure that it isn't drifting to the side.

## Troubleshooting

### LiDAR Issues

#### LiDAR data not publishing properly

1. Ensure the driver is working properly (can use rviz to see output)
2. Check to see if the IP of the Jetson is matching the IP of the LiDAR (sometimes needed to properly recieve data)
3. Check the name of the node (driver might be publishing data in a node named differently than the one being read)
4. Check the data format (must be pointcloud2 data)
5. Check LiDAR's blind spot size, it's possible an obstacle inside the blind spot is preventing LiDAR from seeing anything.

#### LiDAR is seeing floor/ceiling

Adjust the zSensitivity variable, which determines how high above and below the LiDAR data is taken (variable is in cm)

Similarly the ySensitivity can be adjusted if the sections are too big/small
