# ROS2-Based-Differential-Drive-Robot-Simulation

This repository contains a **complete simulation pipeline for a differential drive mobile robot** using **ROS 2 and Gazebo**.  
The project demonstrates how to build a robot **from scratch**, simulate it in Gazebo, perform **mapping with SLAM**, and achieve **autonomous navigation using Nav2**.

The robot model is created manually using **URDF/Xacro**, including **geometry, inertia, sensors, and simulation plugins**.  
After integrating ROS2 with Gazebo through a **Gazebo–ROS bridge**, the robot performs **SLAM-based mapping** and later uses the generated map for **autonomous navigation**.

This project provides a clear example of how to construct a **complete robotics simulation workflow**, starting from robot modeling to autonomous navigation.

---

# Project Overview

The main stages of this project are:

1. **Robot Modeling**
   - Build a differential drive robot from scratch using **URDF/Xacro**
   - Define geometry, mass, inertia, and physical properties

2. **Sensor Integration**
   - Add sensors required for navigation and mapping
   - Lidar sensor
   - Camera sensor

3. **Gazebo Simulation**
   - Integrate the robot with Gazebo
   - Add required simulation plugins
   - Create a custom Gazebo world

4. **ROS2–Gazebo Communication**
   - Configure **Gazebo bridge**
   - Enable communication between simulation topics and ROS2

5. **SLAM Mapping**
   - Use **SLAM Toolbox**
   - Configure parameters for mapping
   - Generate and save environment maps

6. **Autonomous Navigation**
   - Use **Nav2 navigation stack**
   - Load the generated map
   - Perform autonomous navigation
---

# Installation

## 1 Install ROS2

Install ROS2 (Humble recommended).

Follow the official installation guide:

https://docs.ros.org/en/humble/Installation.html

---

## 2 Install Required Packages

Install required ROS2 packages.

```bash
sudo apt update

sudo apt install \
ros-humble-gazebo-ros-pkgs \
ros-humble-slam-toolbox \
ros-humble-nav2-bringup \
ros-humble-navigation2 \
ros-humble-xacro \
ros-humble-rviz2
```
---

## 3 Create ROS2 Workspace
```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
```
---

## 4 Clone the Repository
```bash
git clone https://github.com/saeed-5340/ROS2-Based-Differential-Drive-Robot-Simulation/tree/deployment/beta
```
---

## 5 Build the Workspace

```bash
cd ~/ros2_ws
colcon build --symlink-install
```
---

## 6 Source the Workspace

```bash
source install/setup.bash
```
---

# Running the Simulation

## 1 Launch Gazebo Simulation(Terminal-1)

```bash
ros2 launch mobile_robot_description gz_sim.launch.py
```

This will:
- Launch Gazebo
- Spawn the robot
- Load sensors
- Start ROS–Gazebo bridge
---

# Run SLAM Mapping(Terminal-2)

Start SLAM Toolbox.

```bash
ros2 launch slam_toolbox online_async_launch.py \
params_file:=./src/mobile_robot_description/config/slam_toolbox_params.yaml
```
Drive the robot to explore the environment and build the map.
---

# Run teleop_twist_keyboard(Terminal-3)
```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```
# Save the Map(Terminal-4)
After mapping is complete:
```bash
ros2 run nav2_map_server map_saver_cli -f /<folder-loaction-you-want-to-save-the-map>
```
This will generate:
```
my_map.pgm
my_map.yaml
```
---

# Autonomous Navigation
Launch navigation using the generated map.
```bash
ros2 launch nav2_bringup navigation_launch.py \
map:=<map-yaml-file-location> {example: (map:=src/mobile_robot_description/maps/my_map.yaml)}
```
If this nav2 not work properly then use turtlebot3_navigation
```bash
ros2 launch turtlebot3_navigation2 navigation2.launch.py use_sim_time:=true \
map:=<map-yaml-file-location> {example: (map:=src/mobile_robot_description/maps/my_map.yaml)}
```
Note: If you need to run turtlebot3_navigation2, you should download the turtlebot3 package.
```bash
sudo apt install ros-humble-navigation2 ros-humble-nav2-bringup ros-humble-turtlebot3*
```
You can now set goals in **RViz** and the robot will navigate autonomously.
---
# Visualization
Launch RViz with the provided configuration:
Note: If you use turtlebot3_navigation2, it will open automatically.
```bash
rviz2 -d src/mobile_robot_description/rviz/urdf_config.rviz
```
## Repository Structure

```
ROS2-Based-Differential-Drive-Robot-Simulation
│
├── src
│   └── mobile_robot_description
│       │
│       ├── config
│       │   ├── gazebo_bridge.yaml
│       │   ├── gz_bridge.yaml
│       │   └── slam_toolbox_params.yaml
│       │
│       ├── launch
│       │   └── gz_sim.launch.py
│       │
│       ├── maps
│       │   ├── my_map.yaml
│       │   └── my_map.pgm
│       │
│       ├── rviz
│       │   └── urdf_config.rviz
│       │
│       ├── urdf
│       │   ├── camera_sensor.xacro
│       │   ├── common_properties.xacro
│       │   ├── lidar.xacro
│       │   ├── mobile_base.xacro
│       │   ├── mobile_base_gazebo.xacro
│       │   ├── my_car.urdf.xacro
│       │   └── README_urdf.md
│       │
│       ├── world
│       │   ├── depot.sdf
│       │   ├── house_world.sdf
│       │   ├── maze.sdf
│       │   ├── test_world_1.sdf
│       │   └── warehouse.sdf
│       │
│       ├── CMakeLists.txt
│       ├── package.xml
│       └── LICENSE
│
├── frames_2026-03-10_16.42.30.gv
├── frames_2026-03-10_16.42.30.pdf
├── .gitignore
├── LICENSE
└── README.md
```

### Folder Description
**config/**
Contains configuration files for communication and SLAM parameters.
* `gz_bridge.yaml` – topic bridge configuration for simulation
* `slam_toolbox_params.yaml` – parameters for SLAM Toolbox mapping and localization
**launch/**
Launch files used to start the full simulation environment.
**maps/**
Saved maps generated using SLAM Toolbox.
**rviz/**
RViz configuration used to visualize the robot, sensors, TF frames, and environment.
**urdf/**
Robot description files written using Xacro modules.
**world/**
Gazebo simulation environments are used to test navigation and mapping.
**frames_*.gv / frames_*.pdf**
TF frame graph generated using `tf2_tools`.
# Robot Model Description

The robot is modeled using **Xacro (XML Macro)** files.  
Each part of the robot is separated into modular components.

### Main Robot File
**my_car.urdf.xacro**
This is the main robot description file.  
It includes all robot components, such as:
- Robot base
- Sensors
- Physical properties
- Gazebo plugins
---

### Robot Base
**mobile_base.xacro**
Defines:
- Base chassis
- Wheel joints
Includes:
- Robot geometry
- Wheel placement
---
### Sensor Files

**lidar.xacro**
Adds a **2D LiDAR sensor** used for:
- SLAM
- Obstacle detection
- Navigation
**camera_sensor.xacro**
Adds a **camera sensor** for visual perception.
---
### Common Properties
**common_properties.xacro**
Defines reusable parameters such as:
- Material definitions
- Physical constants
- Reusable macros

---

### Gazebo Simulation Configuration

**mobile_base_gazebo.xacro**

Adds Gazebo-specific configuration:

- Simulation plugins
- Sensor simulation
- Physics properties
- Differential drive structure
- Joint configuration
---
# Gazebo Simulation World
The simulation environment is defined in:
This world contains:
- Obstacles
- Environment used for mapping and navigation
---
# ROS2–Gazebo Bridge
Communication between Gazebo and ROS2 is configured using:

```
config/gz_bridge.yaml
```
The bridge allows:
- Sensor data transfer
- Command velocity publishing
- Simulation topic communication
This enables ROS2 nodes to interact with the Gazebo simulation.
---
# SLAM Mapping
The robot performs **Simultaneous Localization and Mapping (SLAM)** using:
**SLAM Toolbox**
Configuration file:

```
config/slam_toolbox_params.yaml
```

The SLAM process allows the robot to:

- Explore the environment
- Build a 2D map
- Localize itself within the environment

---

# Generated Map

After mapping is completed, the map is saved in:

```
maps/
```
Files:
```
my_map.pgm
my_map.yaml
```
These files represent:
- **PGM file** → Map image
- **YAML file** → Map metadata
---

# Autonomous Navigation

After generating the map, the robot performs autonomous navigation using:

**Nav2 (Navigation2 stack)**

Due to bringup issues, the project uses **TurtleBot3 navigation configuration** as a base.
This allows:
- Path planning
- Obstacle avoidance
- Goal-based navigation
The robot can autonomously move inside the mapped environment.
---
RViz allows visualization of:
- Robot model
- Laser scans
- Map
- Navigation path
- Sensor data
---
# Key Features

- Custom **Differential Drive Robot**
- Full **URDF/Xacro-based modeling**
- **LiDAR and Camera sensor integration**
- **Gazebo simulation**
- **ROS2–Gazebo communication bridge**
- **SLAM-based mapping**
- **Map saving**
- **Autonomous navigation using Nav2**
- **RViz visualization**

---

# Learning Outcomes

This project demonstrates important robotics concepts:

- Robot modeling with URDF
- Sensor integration
- Gazebo simulation
- ROS2 middleware communication
- SLAM mapping
- Autonomous navigation
It provides a strong foundation for developing **autonomous mobile robots using ROS2**.
---

# Future Improvements

Possible improvements include:

- Custom Nav2 bringup configuration
- Additional sensors (IMU, depth camera)
- Multi-robot simulation
- Autonomous exploration
- Path planning algorithm customization

---

# Author

**Saeed Ahamed Mridha**

Electrical and Electronic Engineering  
Shahjalal University of Science and Technology

---

# License

This project is open-source and available under the MIT License.
