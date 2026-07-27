# Tunnel Navigation Program

## Usage

The program is designed to allow a Jetson Orin Nano to control a UGV01 to navigate through a tunnel, using a Livox Avia lidar in order to see obstacles so that it can avoid them. While a different robot or lidar can be used, some alteration will be necessary.

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

- [Livox AVIA LiDAR](https://www.livoxtech.com/avia)
- Two ultrasonic sensors, one on each side of the robot (connected to the 40 pin UART, the trig and echo pins should be 29 and 31 for the first sensor and 33 and 32 for the second)

### Usage

Before using the program, make sure the lidar and the UGV01 are connected to the Jetson.

To use the program, open up the lidar driver and begin publishing point cloud data (if using ASIG-X's driver you would enter your workspace and run `ros2 launch livox_ros2_avia livox_lidar_launch.py`).

Then, open the ipynb file on the Jetson and run it. The file should stop right before the navigation loop runs and wait for a mouse button to be pressed. Alternatively, it can be set to wait for a command to be send to the UGV01 by an external device through its hotspot.

## Code Overview

### Robot Setup

Lets the Jetson know whether to communicate with the robot through the serial port (if connected through 40 pin UART) or through the web interface (if connected through hotspot).

All sensor related function and basic movement functions are defined here.

The pins for the two ultrasonic sensors are also set here.

### Lidar Setup

Sets the functions for getting the Lidar data.

The Lidar data will be a point cloud, or a list of points where each point represents an object that the lidar has been detected. This list is split into several different lists, with each one containing points from a certain area.
