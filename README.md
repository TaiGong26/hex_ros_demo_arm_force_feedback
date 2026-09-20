# hex_ros_demo_arm_force_feedback — Bilateral Arm Force-Feedback Demo

[中文](README_cn.md) | **English**

## Table of Contents

- [Overview](#overview)
- [Quick Start](#quick-start)
- [Installation](#installation)
- [Topics](#topics)
- [Parameters](#parameters)
- [Project Structure](#project-structure)

## Overview

`hex_ros_demo_arm_force_feedback` demonstrates bilateral force feedback with Archer Y6 and Firefly Y6 arms. The slave follows the master motion, while the master reflects feedback derived from the slave state so the operator can perceive interaction at the remote side.

It provides:

- dual-simulation validation;
- a real master with a simulated slave;
- complete bilateral feedback between same-type real arms;
- a standalone force-feedback node for custom integration.

This package supports **ROS 2 Humble** and is compatible with **ROS 1 Noetic**.

## Quick Start

> Complete [Installation](#installation) before launching.

### 1. Launch scenarios

#### Simulation-to-simulation (sim2sim)

This scenario uses two simulated Archer Y6 arms.

ROS 2:

```shell
ros2 launch hex_ros_demo_arm_force_feedback sim2sim_force_feedback.launch.py
```

ROS 1:

```shell
roslaunch hex_ros_demo_arm_force_feedback sim2sim_force_feedback.launch
```

#### Real-to-simulation (real2sim)

This scenario connects a real Archer Y6 or Firefly Y6 master to a simulated Archer Y6 slave for force-feedback validation.

ROS 2:

```shell
ros2 launch hex_ros_demo_arm_force_feedback real2sim_force_feedback.launch.py \
    master_robot_host:=192.168.1.100 master_robot_port:=8439 \
    robot_type:=archer robot_grip_type:=empty
```

ROS 1:

```shell
roslaunch hex_ros_demo_arm_force_feedback real2sim_force_feedback.launch \
    master_robot_host:=192.168.1.100 master_robot_port:=8439 \
    robot_type:=archer robot_grip_type:=empty
```

#### Real-to-real (real2real)

Different arm models should not be used.

ROS 2:

```shell
ros2 launch hex_ros_demo_arm_force_feedback real2real_force_feedback.launch.py \
    master_robot_host:=192.168.1.100 master_robot_port:=8439 \
    slave_robot_host:=192.168.1.101 slave_robot_port:=8439 \
    robot_type:=archer robot_grip_type:=empty
```

ROS 1:

```shell
roslaunch hex_ros_demo_arm_force_feedback real2real_force_feedback.launch \
    master_robot_host:=192.168.1.100 master_robot_port:=8439 \
    slave_robot_host:=192.168.1.101 slave_robot_port:=8439 \
    robot_type:=archer robot_grip_type:=empty
```

### 2. Keyboard Controls and Startup Sequence

Complete launches always start one keyboard node.

1. After launch and receipt of both arm states, the master and slave arms immediately move to `arm_start_pos`.
2. After reaching the startup stable position, the node waits for keyboard input.
3. Press **`s`** to enter bilateral force-feedback control; `s` does not gate the preceding startup motion.
4. Press **`q`** to stop force-feedback control, move both arms to `arm_end_pos`, and exit.

If the terminal repeatedly prints `waiting for master state...` or `waiting for slave state...`, verify the IP addresses, ports, and `robot_type`; ensure the corresponding drivers are running and that `master/manip_state` and `slave/manip_state` are published continuously. The current node waits indefinitely rather than timing out.

## Installation

### Prerequisites

- **ROS 2 Humble** is installed; use **ROS 1 Noetic** for ROS 1 compatibility.
- Python 3, `pip3`, Git, and the build tools for the selected ROS version are installed.
- For real-hardware scenarios, devices are reachable and the actual IP addresses, ports, and device models are known.

### 1. Install Python Dependencies

```shell
pip3 install \
    'hex-util-msg>=0.1.0' \
    'hex-util-ros>=0.1.0' \
    'hex-driver-robot>=0.1.0'
```

### 2. Create and Enter the Workspace

```shell
mkdir -p <your_ws>/src
cd <your_ws>/src
```

### 3. Clone ROS Packages

```shell
git clone https://github.com/hexfellow/hex_ros_msgs.git
git clone https://github.com/hexfellow/hex_ros_demo_arm_force_feedback.git
git clone https://github.com/hexfellow/hex_ros_robot_arm.git
git clone https://github.com/hexfellow/hex_ros_sim_archer_y6.git
git clone https://github.com/hexfellow/hex_ros_teleop_keyboard.git
git clone https://github.com/hexfellow/hex_ros_urdf_archer_y6.git
```

### 4. Build

**ROS 2:**

```shell
source /opt/ros/humble/setup.bash
cd <your_ws>
colcon build
source install/setup.bash
```

**ROS 1:**

```shell
source /opt/ros/noetic/setup.bash
cd <your_ws>
catkin_make
source devel/setup.bash
```

## Topics

| Direction | Topic | Type | Description |
|-----------|-------|------|-------------|
| pub | `master/manip_ctrl` | `hex_ros_msgs/msg/HexRosRoboManipCtrlStamped` | Master arm control message |
| pub | `slave/manip_ctrl` | `hex_ros_msgs/msg/HexRosRoboManipCtrlStamped` | Slave arm control message |
| sub | `master/manip_state` | `hex_ros_msgs/msg/HexRosRoboManipStateStamped` | Master arm state message |
| sub | `slave/manip_state` | `hex_ros_msgs/msg/HexRosRoboManipStateStamped` | Slave arm state message |
| sub | `teleop_keyboard_state` | `hex_ros_msgs/msg/HexRosTeleopKeyboardStateStamped` | Keyboard state message |

> Message type description: [hex_ros_msgs public APIs](https://github.com/hexfellow/hex_ros_msgs#public-apis)

## Parameters

### Launch Arguments

| Argument | Meaning / Values |
|---|---|
| `master_robot_host / master_robot_port` | Master IP / port |
| `slave_robot_host / slave_robot_port` | Slave IP / port; complete real launches only |
| `robot_type` | `archer` or `firefly` |
| `robot_grip_type` | gp100 / gp80 / gr100 / empty |
| `viewer / rviz` | Simulation window switches; simulation scenarios only |

### Node Parameters

Parameters are set in `config/ros1/arm_force_feedback.yaml` and `config/ros2/arm_force_feedback.yaml`.

| Param | Default | Description |
|-------|---------|-------------|
| `rate_ros` | `1000.0` | Force feedback control loop rate [Hz] |
| `rate_teleop` | `100.0` | Keyboard monitor rate [Hz] |
| `model_urdf` | `""` | URDF model file path (for dynamics computation) |
| `model_frame_id` | `base_link` | Robot base frame ID |
| `pose_end_in_flange` | `[0.187, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0]` | End-effector pose in flange [x, y, z, qw, qx, qy, qz] |
| `gravity` | `[0.0, 0.0, -9.81]` | Gravity acceleration vector [m/s²] |
| `arm_start_pos` | `[0.0, 0.1, 2.54, -1.07, 0.0, 0.0]` | Arm stable start position [rad] |
| `arm_end_pos` | `[0.0, -1.5, 3.0, 0.07, 0.0, 0.0]` | Arm exit home position [rad] |
| `grip_stable_pos` | `[0.5]` | Gripper stable position |
| `arm_stable_kp` | `[200.0, 200.0, 250.0, 150.0, 100.0, 100.0]` | Arm stable motion PD gain — proportional |
| `arm_stable_kd` | `[5.0, 5.0, 5.0, 5.0, 2.0, 2.0]` | Arm stable motion PD gain — derivative |
| `grip_stable_kp` | `[10.0]` | Gripper stable motion PD gain — proportional |
| `grip_stable_kd` | `[0.5]` | Gripper stable motion PD gain — derivative |
| `arm_master_kp` | `[50.0, 50.0, 50.0, 20.0, 5.0, 1.5]` | Master arm force feedback PD gain — proportional |
| `arm_master_kd` | `[0.0, 0.0, 0.0, 0.0, 0.0, 0.0]` | Master arm force feedback PD gain — derivative |
| `grip_master_kp` | `[20.0]` | Master gripper force feedback PD gain — proportional |
| `grip_master_kd` | `[0.0]` | Master gripper force feedback PD gain — derivative |
| `arm_slave_kp` | `[200.0, 200.0, 250.0, 200.0, 100.0, 100.0]` | Slave arm tracking PD gain — proportional |
| `arm_slave_kd` | `[5.0, 5.0, 5.0, 5.0, 2.0, 2.0]` | Slave arm tracking PD gain — derivative |
| `grip_slave_kp` | `[200.0]` | Slave gripper tracking PD gain — proportional |
| `grip_slave_kd` | `[10.0]` | Slave gripper tracking PD gain — derivative |
| `arm_master_deadzone` | `[0.05, 0.05, 0.05, 0.1, 0.1, 0.1]` | Master arm deadzone — error below threshold is zeroed [rad] |
| `arm_master_clip` | `[1.0, 1.0, 1.0, 1.0, 1.0, 1.0]` | Master arm error saturation clip [rad] |
| `arm_slave_deadzone` | `[0.1, 0.1, 0.1, 0.1, 0.1, 0.1]` | Slave arm deadzone — error below threshold is zeroed [rad] |
| `arm_slave_clip` | `[0.5, 0.5, 0.5, 0.5, 0.5, 0.5]` | Slave arm error saturation clip [rad] |
| `grip_master_deadzone` | `[0.01]` | Master gripper deadzone — error below threshold is zeroed [rad] |
| `grip_master_clip` | `[0.5]` | Master gripper error saturation clip [rad] |
| `grip_slave_deadzone` | `[0.01]` | Slave gripper deadzone — error below threshold is zeroed [rad] |
| `grip_slave_clip` | `[0.3]` | Slave gripper error saturation clip [rad] |

> Defaults come from `config/<ros_version>/arm_force_feedback.yaml`; ROS 1 and ROS 2 configurations match.

## Project Structure

```text
hex_ros_demo_arm_force_feedback/
├── config/
│   ├── ros1/
│   │   └── arm_force_feedback.yaml               # ROS 1 node parameters
│   └── ros2/
│       └── arm_force_feedback.yaml               # ROS 2 node parameters
├── hex_ros_demo_arm_force_feedback/
│   ├── utility/
│   │   ├── __init__.py                           # utility package initializer
│   │   ├── interface_base.py                     # ROS interface base class
│   │   ├── ros1_interface.py                     # ROS 1 interface
│   │   └── ros2_interface.py                     # ROS 2 interface
│   ├── __init__.py                               # Python package initializer
│   ├── arm_force_feedback.py                     # Force-feedback node
│   └── TrajectoryController.py                   # Trajectory control classes
├── launch/
│   ├── ros1/
│   │   ├── arm_force_feedback.launch             # ROS 1 force-feedback-node launch file
│   │   ├── real2real_force_feedback.launch       # ROS 1 real-to-real launch file
│   │   ├── real2sim_force_feedback.launch        # ROS 1 real-to-simulation launch file
│   │   └── sim2sim_force_feedback.launch         # ROS 1 simulation-to-simulation launch file
│   └── ros2/
│       ├── arm_force_feedback.launch.py          # ROS 2 force-feedback-node launch file
│       ├── real2real_force_feedback.launch.py    # ROS 2 real-to-real launch file
│       ├── real2sim_force_feedback.launch.py     # ROS 2 real-to-simulation launch file
│       └── sim2sim_force_feedback.launch.py      # ROS 2 simulation-to-simulation launch file
├── resource/
│   └── hex_ros_demo_arm_force_feedback
├── .gitignore
├── CMakeLists.txt
├── LICENSE
├── package.xml
├── README_cn.md
├── README.md
├── setup.cfg
└── setup.py
```
