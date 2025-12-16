# Package Overview

The Navigator robot software is organized into six main packages, each with a specific purpose.

## Package Structure

```
navigator/
├── navigator_bringup/      # Launch files for bringing up the robot
├── navigator_control/      # ros2_control configuration
├── navigator_description/  # URDF, meshes, robot model
├── navigator_msgs/         # Custom ROS messages
├── navigator_nav/          # Navigation and SLAM configuration
└── navigator_sim/          # Gazebo simulation
```

---

## 📦 navigator_bringup

**Purpose**: High-level launch files for starting the complete robot system

### Contents

```
navigator_bringup/
├── launch/
│   └── bringup_rviz.launch.py    # Launch control + RViz
├── config/                        # Configuration files
├── CMakeLists.txt
└── package.xml
```

### Key Features

- Combines multiple subsystems (control, sensors, navigation)
- Provides convenient entry points for common scenarios
- Manages startup sequences with delays

### When to Use

- Starting the complete robot system
- Production deployments
- Integrated testing

### Example Usage

```bash
ros2 launch navigator_bringup bringup_rviz.launch.py
```

---

## 🎮 navigator_control

**Purpose**: ros2_control configuration for differential drive control

### Contents

```
navigator_control/
├── config/
│   ├── ros2_control.yaml          # Controller configuration
│   ├── diff_controllers.yaml      # Differential drive settings
│   └── navigator.rviz             # RViz configuration
├── launch/
│   ├── ros2_control.launch.py     # Start controllers
│   └── view_robot.launch.py       # View robot in RViz
├── CMakeLists.txt
└── package.xml
```

### Key Features

- **Differential Drive Controller**: Controls two-wheeled robot motion
- **Joint State Broadcaster**: Publishes wheel positions/velocities
- **Mock Hardware Support**: Test without physical robot
- **Odometry Publishing**: Provides pose estimation

### Configuration Parameters

From `ros2_control.yaml`:
- `wheel_radius`: 0.033 m
- `wheel_separation`: 0.145 m
- `update_rate`: 50 Hz
- `cmd_vel_timeout`: 0.5 s

### Example Usage

```bash
# Start control system
ros2 launch navigator_control ros2_control.launch.py

# View in RViz
ros2 launch navigator_control view_robot.launch.py
```

---

## 🤖 navigator_description

**Purpose**: Robot model definition (URDF), meshes, and visual assets

### Contents

```
navigator_description/
├── urdf/
│   ├── navigator.urdf.xacro           # Main robot definition
│   ├── navigator_gazebo.urdf.xacro    # Gazebo-specific model
│   ├── ros2_control.xacro             # Control interfaces
│   ├── gazebo.xacro                   # Gazebo plugins
│   ├── bases/
│   │   └── chassis.xacro              # Chassis definition
│   ├── wheels/
│   │   └── wheel.xacro                # Wheel macros
│   └── sensors/
│       ├── lidar.xacro                # Lidar sensor
│       └── camera.xacro               # Camera sensor
├── meshes/                            # 3D mesh files (.STL)
│   ├── base_link.STL
│   ├── left_wheel_link.STL
│   ├── right_wheel_link.STL
│   └── lidar_link.STL
├── launch/                            # Launch files
├── config/                            # Configuration files
├── CMakeLists.txt
└── package.xml
```

### Key Features

- **Modular URDF**: Separate files for chassis, wheels, sensors
- **Two Model Variants**:
  - `navigator.urdf.xacro`: For ros2_control
  - `navigator_gazebo.urdf.xacro`: For Gazebo simulation
- **Parametric Design**: Easy to adjust dimensions
- **STL Meshes**: Accurate visual representation

### Robot Specifications

- **Dimensions**: 180mm × 120mm × 60mm
- **Wheel Radius**: 36mm
- **Wheel Separation**: 195mm
- **Mass**: ~5kg base + 2kg per wheel
- **Sensors**:
  - 360° Lidar (10m range)
  - RGB Camera (640×480)

### Example Usage

```bash
# View robot model
ros2 launch navigator_description view_robot.launch.py

# Check URDF syntax
check_urdf navigator.urdf.xacro
```

---

## 📨 navigator_msgs

**Purpose**: Custom ROS 2 message definitions for Navigator-specific data

### Contents

```
navigator_msgs/
├── msg/
│   ├── WheelState.msg          # Individual wheel data
│   ├── SystemStatus.msg        # System health
│   ├── NavigatorStatus.msg     # Robot state
│   ├── MotorCommand.msg        # Motor commands
│   └── BatteryStatus.msg       # Battery info
├── CMakeLists.txt
└── package.xml
```

### Message Definitions

#### WheelState.msg
```
float32 velocity    # rad/s
float32 position    # rad
float32 effort      # torque
```

#### SystemStatus.msg
```
bool is_ok
string message
```

#### NavigatorStatus.msg
```
string mode          # "IDLE", "MANUAL", "AUTO", "DOCKING"
bool motors_enabled
bool estop_pressed
```

#### MotorCommand.msg
```
float32 left_velocity
float32 right_velocity
```

#### BatteryStatus.msg
```
float32 voltage
float32 current
float32 percentage
bool is_charging
```

### Example Usage

```python
from navigator_msgs.msg import NavigatorStatus

# Publish status
status = NavigatorStatus()
status.mode = "AUTO"
status.motors_enabled = True
status.estop_pressed = False
pub.publish(status)
```

---

## 🗺️ navigator_nav

**Purpose**: Navigation and SLAM configuration files

### Contents

```
navigator_nav/
├── config/
│   ├── nav2_params.yaml        # Nav2 parameters
│   ├── amcl_params.yaml        # AMCL localization
│   └── slam_params.yaml        # SLAM Toolbox config
├── launch/
│   ├── navigation.launch.py    # Nav2 navigation
│   ├── slam.launch.py          # SLAM mapping
│   └── localization.launch.py  # AMCL localization
├── maps/                       # Saved maps
├── CMakeLists.txt
└── package.xml
```

### Key Features

- Pre-configured Nav2 parameters
- SLAM Toolbox integration
- AMCL localization
- Costmap configurations
- Behavior tree setup

### Example Usage

```bash
# Start SLAM mapping
ros2 launch navigator_nav slam.launch.py

# Start navigation with map
ros2 launch navigator_nav navigation.launch.py map:=~/maps/office.yaml
```

---

## 🎮 navigator_sim

**Purpose**: Gazebo Fortress simulation environment

### Contents

```
navigator_sim/
├── worlds/
│   ├── empty.sdf               # Empty world
│   ├── improved.sdf            # Basic world with lighting
│   └── obstacles.sdf           # World with obstacles
├── launch/
│   ├── gazebo.launch.py                # Basic simulation
│   ├── gazebo_slam.launch.py           # Simulation + SLAM
│   ├── gazebo_slam_nav2.launch.py      # Full navigation stack
│   ├── gazebo_navigation.launch.py     # Navigation with map
│   ├── slam.launch.py                  # SLAM only
│   ├── nav2.launch.py                  # Nav2 only
│   ├── navigation.launch.py            # Navigation with AMCL
│   └── rviz_nav2.launch.py             # RViz with Nav2 config
├── config/
│   ├── nav2_params.yaml        # Nav2 configuration
│   ├── amcl_params.yaml        # AMCL configuration
│   ├── nav2_default_view.rviz  # RViz layout
│   └── joint_state_config.yaml # Joint state publisher
├── save_map.sh                 # Map saving script
├── CMakeLists.txt
└── package.xml
```

### Key Features

- **Multiple Worlds**: Empty, improved, obstacles
- **Complete Launch Files**: From simple to full navigation
- **Gazebo Plugins**:
  - Differential drive
  - Lidar sensor
  - Camera sensor
  - Joint state publisher
  - Odometry
- **ROS-Gazebo Bridges**: All sensor data bridged to ROS 2

### World Descriptions

- **empty.sdf**: Minimal world with ground and sun
- **improved.sdf**: Better physics and lighting
- **obstacles.sdf**: Various obstacles for testing navigation

### Example Usage

```bash
# Basic simulation
ros2 launch navigator_sim gazebo.launch.py

# Full SLAM + Nav2
ros2 launch navigator_sim gazebo_slam_nav2.launch.py

# Navigation with existing map
ros2 launch navigator_sim gazebo_navigation.launch.py map:=~/maps/my_map.yaml

# Save current map
cd ~/navigator_ws/src/navigator/navigator_sim
./save_map.sh my_office_map
```

---

## 🔗 Package Dependencies

### navigator_bringup
- navigator_description
- navigator_control
- navigator_nav
- robot_state_publisher

### navigator_control
- controller_manager
- diff_drive_controller
- joint_state_broadcaster

### navigator_description
- robot_state_publisher
- xacro

### navigator_msgs
- rosidl_default_generators
- rosidl_default_runtime

### navigator_nav
- nav2_bringup
- slam_toolbox

### navigator_sim
- ros_gz_sim
- ros_gz_bridge
- navigator_description

---

## 🎯 Quick Reference

| Task | Package | Command |
|------|---------|---------|
| View robot model | navigator_description | `ros2 launch navigator_description view_robot.launch.py` |
| Start controllers | navigator_control | `ros2 launch navigator_control ros2_control.launch.py` |
| Run simulation | navigator_sim | `ros2 launch navigator_sim gazebo.launch.py` |
| Create map | navigator_sim | `ros2 launch navigator_sim gazebo_slam.launch.py` |
| Navigate autonomously | navigator_sim | `ros2 launch navigator_sim gazebo_navigation.launch.py` |
| Full system | navigator_bringup | `ros2 launch navigator_bringup bringup_rviz.launch.py` |

---

[← Back to Main Documentation](../README.md) | [Next: Robot Description →](robot-description.md)