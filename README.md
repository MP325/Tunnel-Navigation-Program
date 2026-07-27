# Tunnel Navigation Program

## Usage

The program is designed to allow a Jetson Orin Nano to control a [UGV01](https://www.waveshare.com/wiki/UGV01) to navigate through a tunnel, using a [Livox Avia LiDAR](https://www.livoxtech.com/avia) in order to see obstacles so that it can avoid them. While a different robot or LiDAR can be used, some alteration to the code will be necessary.

The program can either communicate with the UGV01 by connecting the Jetson to the ESP32 through the 40 pin UART, or by connecting to the UGV01's hotspot.

Data can be recorded as the UGV01 moves throughout the tunnel, allowing for a scan of the walls of the tunnel to be completed as the robot moves from start to end and returns. This can assist in identifying cracks and structural issues in the tunnel without needing a human to enter the tunnel at all.

## Installation and Usage

### System Requirements

- **ROS 2 Humble** installed on your system.
- **Python 3.x** with `pip` configured.

### Software Prerequisites

To use the program, the following must be installed:

- A driver that can read and publish point cloud data in the pointcloud2 format (for the Avia **[ASIG-X's driver](https://github.com/ASIG-X/livox_ros2_avia)** works very well)
- **[UGV Jetson](https://github.com/waveshareteam/ugv_jetson/tree/main)** (in the case that the Jetson Orin Nano is being connected to the UGV01 through the 40 pin UART)
- **Jupyter** (comes with UGV Jetson)

You can run this command to install all of the dependencies with `pip`:

`python -m pip install pyserial numpy altair pandas requests pynput ipython ipywidgets`

### Hardware Requirements

To use the program, the following sensors need to be affixed to the robot:

- One **Livox AVIA LiDAR**, which should be mounted on the front of the robot
- Two **ultrasonic sensors**, one on each side of the robot (connected to the 40 pin UART, the trig and echo pins should be 29 and 31 for the first sensor and 33 and 32 for the second)

### Usage

Before using the program, make sure the LiDAR and the UGV01 are connected to the Jetson.

To use the program, open up the LiDAR driver and begin publishing point cloud data (if using ASIG-X's driver you would enter your workspace and run `ros2 launch livox_ros2_avia livox_LiDAR_launch.py`).

Then, open the ipynb file on the Jetson and run it. The file should stop right before the navigation loop runs and wait for a mouse button to be pressed. Alternatively, it can be set to wait for a command to be send to the UGV01 by an external device through its hotspot. The navigation should be started when the robot is ready to go and placed in front of the tunnel, as straight as possible.

## Basic Code Overview

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

**How the LiDAR data is handled:**

The LiDAR data is published as a point cloud, or a list of points where each point represents an object that the LiDAR has been detected. This list is split into several different lists, with each one containing points from a certain area. There can be any amount of these sections, but there is a base number of 7. Each section is about as big as the robot. 

The closest point in each section is then identified.

<p align="center"><img width="800" height="450" alt="LiDAR-scanning-gif" src="https://github.com/user-attachments/assets/f663da8e-1923-4900-a3d2-b70f8a768cb9" /></p>

The idea is that by looking at the closest point in each section, we can see which sections have obstacles nearby and how close they are. Since each section is about the size of the robot, we can easily have the robot avoid blockages by just moving to a clear section from a blocked one and simply going forward.

### Navigation

- Defines weights and navigation parameters
- Defines more advanced sensor/movment related functions used for navigation
- Creates navigation loop

**Navigation loop overview:**

This idea is then put into practice in the navigation loop. The robot identifies obstacles and moves around them, continuously scanning and rerouting.

<p align="center"><img width="800" height="450" alt="robot-navigating-gif" src="https://github.com/user-attachments/assets/88c0536a-c104-4d66-892f-547983c761d3" /></p>

This continues until the robot detects it has reached a dead end, where it then turns around and navigates until it reaches the point where it started from. The robot also uses various sensors like the ultrasonic and motor encoders in an attempt to negate drift.

When looking at this section, there are some notable items that weren't previously mentioned such as the blocked sections list. These are mainly due to the massive limitation that is the Livox Avia LiDAR's enormous 1 meter blind spot, which led to quite a few workarounds being added.
