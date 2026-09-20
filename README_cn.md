# hex_ros_demo_arm_force_feedback — 机械臂双边力反馈演示

**中文** | [English](README.md)

## 目录

- [项目概述](#项目概述)
- [快速使用](#快速使用)
- [安装](#安装)
- [话题接口](#话题接口)
- [参数](#参数)
- [项目结构](#项目结构)

## 项目概述

`hex_ros_demo_arm_force_feedback` 是 Archer Y6 / Firefly Y6 双边力反馈演示包。从臂跟随主臂运动，主臂根据从臂状态产生反馈，使操作者能够感知从端交互。

主要提供：

- 双仿真机械臂验证；
- 真机主臂到仿真从臂的验证；
- 同型号主从真机的完整双边力反馈；
- 可独立集成的力反馈控制节点。

本包支持 **ROS 2 Humble**，兼容 **ROS 1 Noetic**。

## 快速使用

> 请先完成[安装](#安装)，再选择以下启动入口。

### 1. 启动场景

#### 仿真对仿真（sim2sim）

该场景使用两台 Archer Y6 仿真机械臂。

ROS 2：

```shell
ros2 launch hex_ros_demo_arm_force_feedback sim2sim_force_feedback.launch.py
```

ROS 1：

```shell
roslaunch hex_ros_demo_arm_force_feedback sim2sim_force_feedback.launch
```

#### 真机对仿真（real2sim）

主臂可为 Archer Y6 或 Firefly Y6 真机，从臂为 Archer Y6 仿真。适合在没有从臂真机时验证力反馈逻辑。

ROS 2：

```shell
ros2 launch hex_ros_demo_arm_force_feedback real2sim_force_feedback.launch.py \
    master_robot_host:=192.168.1.100 master_robot_port:=8439 \
    robot_type:=archer robot_grip_type:=empty
```

ROS 1：

```shell
roslaunch hex_ros_demo_arm_force_feedback real2sim_force_feedback.launch \
    master_robot_host:=192.168.1.100 master_robot_port:=8439 \
    robot_type:=archer robot_grip_type:=empty
```

#### 真机对真机（real2real）

不应使用不同型号的机械臂。

ROS 2：

```shell
ros2 launch hex_ros_demo_arm_force_feedback real2real_force_feedback.launch.py \
    master_robot_host:=192.168.1.100 master_robot_port:=8439 \
    slave_robot_host:=192.168.1.101 slave_robot_port:=8439 \
    robot_type:=archer robot_grip_type:=empty
```

ROS 1：

```shell
roslaunch hex_ros_demo_arm_force_feedback real2real_force_feedback.launch \
    master_robot_host:=192.168.1.100 master_robot_port:=8439 \
    slave_robot_host:=192.168.1.101 slave_robot_port:=8439 \
    robot_type:=archer robot_grip_type:=empty
```

### 2. 键盘控制与启动顺序

完整 launch 会固定启动一个键盘节点。

1. launch 启动并收到主、从臂状态后，两台机械臂会立即自动运动到 `arm_start_pos`。
2. 到达启动稳定位置后，节点等待键盘输入。
3. 按 **`s`** 进入双边力反馈控制；`s` 不控制前面的启动运动。
4. 按 **`q`** 停止力反馈控制，机械臂运动到 `arm_end_pos` 后退出。

如果终端持续显示 `waiting for master state...` 或 `waiting for slave state...`，请检查 IP、端口和 `robot_type`，确认对应驱动正在运行，并确认 `master/manip_state` 与 `slave/manip_state` 持续发布。当前节点会持续等待状态，不会自动超时退出。

## 安装

### 前置条件

- 已安装 **ROS 2 Humble**；使用 ROS 1 时安装 **ROS 1 Noetic**。
- 已安装 Python 3、`pip3`、Git，以及所选 ROS 版本的构建工具。
- 真机场景需确保设备网络可达，并准备好实际 IP、端口和设备型号。

### 1. 安装 Python 依赖

```shell
pip3 install \
    'hex-util-msg>=0.1.0' \
    'hex-util-ros>=0.1.0' \
    'hex-driver-robot>=0.1.0'
```

### 2. 创建并进入工作空间

```shell
mkdir -p <your_ws>/src
cd <your_ws>/src
```

### 3. 克隆 ROS 包

```shell
git clone https://github.com/hexfellow/hex_ros_msgs.git
git clone https://github.com/hexfellow/hex_ros_demo_arm_force_feedback.git
git clone https://github.com/hexfellow/hex_ros_robot_arm.git
git clone https://github.com/hexfellow/hex_ros_sim_archer_y6.git
git clone https://github.com/hexfellow/hex_ros_teleop_keyboard.git
git clone https://github.com/hexfellow/hex_ros_urdf_archer_y6.git
```

### 4. 编译包

**ROS 2：**

```shell
source /opt/ros/humble/setup.bash
cd <your_ws>
colcon build
source install/setup.bash
```

**ROS 1：**

```shell
source /opt/ros/noetic/setup.bash
cd <your_ws>
catkin_make
source devel/setup.bash
```

## 话题接口

| 方向 | 话题 | 类型 | 说明 |
|------|------|------|------|
| 发布 | `master/manip_ctrl` | `hex_ros_msgs/msg/HexRosRoboManipCtrlStamped` | 主臂控制消息 |
| 发布 | `slave/manip_ctrl` | `hex_ros_msgs/msg/HexRosRoboManipCtrlStamped` | 从臂控制消息 |
| 订阅 | `master/manip_state` | `hex_ros_msgs/msg/HexRosRoboManipStateStamped` | 主臂状态消息 |
| 订阅 | `slave/manip_state` | `hex_ros_msgs/msg/HexRosRoboManipStateStamped` | 从臂状态消息 |
| 订阅 | `teleop_keyboard_state` | `hex_ros_msgs/msg/HexRosTeleopKeyboardStateStamped` | 键盘状态消息 |

> 消息类型描述：[hex_ros_msgs public APIs](https://github.com/hexfellow/hex_ros_msgs#public-apis)

## 参数

### 启动参数

| 参数 | 含义 / 取值 |
|---|---|
| `master_robot_host / master_robot_port` | 主臂 IP / 端口 |
| `slave_robot_host / slave_robot_port` | 从臂 IP / 端口；仅完整真机场景 |
| `robot_type` | `archer` 或 `firefly` |
| `robot_grip_type` | gp100 / gp80 / gr100 / empty |
| `viewer / rviz` | 仿真窗口开关；仅仿真场景 |

### 节点参数

参数在 `config/ros1/arm_force_feedback.yaml` 和 `config/ros2/arm_force_feedback.yaml` 中设置。

| 参数 | 默认值 | 说明 |
|------|--------|------|
| `rate_ros` | `1000.0` | 力反馈控制循环频率 [Hz] |
| `rate_teleop` | `100.0` | 键盘监听频率 [Hz] |
| `model_urdf` | `""` | URDF 模型文件路径（用于动力学计算） |
| `model_frame_id` | `base_link` | 机器人基坐标系 |
| `pose_end_in_flange` | `[0.187, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0]` | 末端执行器在法兰盘上的位姿 [x, y, z, qw, qx, qy, qz] |
| `gravity` | `[0.0, 0.0, -9.81]` | 重力加速度向量 [m/s²] |
| `arm_start_pos` | `[0.0, 0.1, 2.54, -1.07, 0.0, 0.0]` | 机械臂启动稳定位置 [rad] |
| `arm_end_pos` | `[0.0, -1.5, 3.0, 0.07, 0.0, 0.0]` | 机械臂退出归位位置 [rad] |
| `grip_stable_pos` | `[0.5]` | 夹爪稳定位置 |
| `arm_stable_kp` | `[200.0, 200.0, 250.0, 150.0, 100.0, 100.0]` | 机械臂稳定运动 PD 增益 — 比例 |
| `arm_stable_kd` | `[5.0, 5.0, 5.0, 5.0, 2.0, 2.0]` | 机械臂稳定运动 PD 增益 — 微分 |
| `grip_stable_kp` | `[10.0]` | 夹爪稳定运动 PD 增益 — 比例 |
| `grip_stable_kd` | `[0.5]` | 夹爪稳定运动 PD 增益 — 微分 |
| `arm_master_kp` | `[50.0, 50.0, 50.0, 20.0, 5.0, 1.5]` | 主臂力反馈 PD 增益 — 比例 |
| `arm_master_kd` | `[0.0, 0.0, 0.0, 0.0, 0.0, 0.0]` | 主臂力反馈 PD 增益 — 微分 |
| `grip_master_kp` | `[20.0]` | 主夹爪力反馈 PD 增益 — 比例 |
| `grip_master_kd` | `[0.0]` | 主夹爪力反馈 PD 增益 — 微分 |
| `arm_slave_kp` | `[200.0, 200.0, 250.0, 200.0, 100.0, 100.0]` | 从臂跟随 PD 增益 — 比例 |
| `arm_slave_kd` | `[5.0, 5.0, 5.0, 5.0, 2.0, 2.0]` | 从臂跟随 PD 增益 — 微分 |
| `grip_slave_kp` | `[200.0]` | 从夹爪跟随 PD 增益 — 比例 |
| `grip_slave_kd` | `[10.0]` | 从夹爪跟随 PD 增益 — 微分 |
| `arm_master_deadzone` | `[0.05, 0.05, 0.05, 0.1, 0.1, 0.1]` | 主臂 deadzone — 误差低于此阈值归零 [rad] |
| `arm_master_clip` | `[1.0, 1.0, 1.0, 1.0, 1.0, 1.0]` | 主臂误差饱和限幅 [rad] |
| `arm_slave_deadzone` | `[0.1, 0.1, 0.1, 0.1, 0.1, 0.1]` | 从臂 deadzone — 误差低于此阈值归零 [rad] |
| `arm_slave_clip` | `[0.5, 0.5, 0.5, 0.5, 0.5, 0.5]` | 从臂误差饱和限幅 [rad] |
| `grip_master_deadzone` | `[0.01]` | 主夹爪 deadzone — 误差低于此阈值归零 [rad] |
| `grip_master_clip` | `[0.5]` | 主夹爪误差饱和限幅 [rad] |
| `grip_slave_deadzone` | `[0.01]` | 从夹爪 deadzone — 误差低于此阈值归零 [rad] |
| `grip_slave_clip` | `[0.3]` | 从夹爪误差饱和限幅 [rad] |

> 默认值来自 `config/<ros_version>/arm_force_feedback.yaml`，ROS 1 与 ROS 2 配置一致。

## 项目结构

```text
hex_ros_demo_arm_force_feedback/
├── config/
│   ├── ros1/
│   │   └── arm_force_feedback.yaml               # ROS 1 节点参数
│   └── ros2/
│       └── arm_force_feedback.yaml               # ROS 2 节点参数
├── hex_ros_demo_arm_force_feedback/
│   ├── utility/
│   │   ├── __init__.py                           # utility 包初始化文件
│   │   ├── interface_base.py                     # ROS 接口基类
│   │   ├── ros1_interface.py                     # ROS 1 接口
│   │   └── ros2_interface.py                     # ROS 2 接口
│   ├── __init__.py                               # Python 包初始化文件
│   ├── arm_force_feedback.py                     # 力反馈节点
│   └── TrajectoryController.py                   # 轨迹控制类
├── launch/
│   ├── ros1/
│   │   ├── arm_force_feedback.launch             # ROS 1 力反馈节点启动文件
│   │   ├── real2real_force_feedback.launch       # ROS 1 真机对真机启动文件
│   │   ├── real2sim_force_feedback.launch        # ROS 1 真机对仿真启动文件
│   │   └── sim2sim_force_feedback.launch         # ROS 1 仿真对仿真启动文件
│   └── ros2/
│       ├── arm_force_feedback.launch.py          # ROS 2 力反馈节点启动文件
│       ├── real2real_force_feedback.launch.py    # ROS 2 真机对真机启动文件
│       ├── real2sim_force_feedback.launch.py     # ROS 2 真机对仿真启动文件
│       └── sim2sim_force_feedback.launch.py      # ROS 2 仿真对仿真启动文件
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
