# Package Overview

The Navigator robot software is organized into six main packages.

## Package Structure

```
navigator/
├── navigator_bringup/      # Hardware interface, launch files, motor driver
├── navigator_control/      # ros2_control configuration (simulation only)
├── navigator_description/  # URDF, meshes, robot model
├── navigator_msgs/         # Custom ROS 2 messages
├── navigator_nav/          # Nav2 and SLAM configuration
└── navigator_sim/          # Gazebo Fortress simulation
```

---

## navigator_bringup

**Purpose**: Real robot hardware interface and bringup launch files.

```
navigator_bringup/
├── config/
│   └── nav2_params.yaml            # Nav2 parameters for real robot
├── launch/
│   ├── hardware.launch.py          # Full hardware bringup ← START HERE
│   ├── bringup_rviz.launch.py      # Hardware + navigation + RViz
│   ├── navigation.launch.py        # SLAM or map-based navigation
│   ├── hardware.launch.py          # Motors + lidar + camera
│   ├── slam.launch.py              # SLAM mapping only
│   └── calibrate_camera.launch.py  # Camera calibration (optional)
├── scripts/
│   ├── ros2_hardware_interface.py  # ROS 2 node: motors + odometry + TF
│   ├── motion.py                   # Motor control with S-curve smoothing
│   └── ddsm.py                     # DDSM motor driver (serial protocol)
├── CMakeLists.txt
└── package.xml
```

### Hardware nodes started by hardware.launch.py

| Node | Package | Purpose |
|------|---------|---------|
| `navigator_hardware` | navigator_bringup | Motors, odometry, TF broadcaster |
| `robot_state_publisher` | robot_state_publisher | URDF → TF |
| `lidar_static_tf` | tf2_ros | 180° lidar mount correction |
| `rplidar_node` | rplidar_ros | RPLidar driver → `/scan` |
| `camera_node` | camera_ros | OV5647 → `/camera/image_raw` |
| `camera_optical_tf` | tf2_ros | camera_link → camera_optical_frame |

### Launch commands

```bash
# Full hardware bringup (use this on the real robot)
ros2 launch navigator_bringup hardware.launch.py

# With SLAM
ros2 launch navigator_bringup navigation.launch.py slam:=true

# With saved map
ros2 launch navigator_bringup navigation.launch.py \
    map:=~/navigator_maps/my_map.yaml

# Camera calibration (optional)
ros2 launch navigator_bringup calibrate_camera.launch.py
```

### Motor driver details

The DDSM motors communicate via serial JSON protocol:
- **Left motor**: ID 2 (QinHeng CH343, `/dev/ttyACM0`)
- **Right motor**: ID 1
- **Wheel radius**: 0.036m
- **Wheel base**: 0.195m
- **Smoothing**: Exponential low-pass filter (alpha=0.3 linear, 0.5 angular)

---

## navigator_control

**Purpose**: ros2_control configuration — used in **simulation only**.

```
navigator_control/
├── config/
│   ├── ros2_control.yaml     # Controller parameters
│   ├── diff_controllers.yaml # Differential drive settings
│   └── navigator.rviz        # RViz config
└── launch/
    ├── ros2_control.launch.py  # Start controllers
    └── view_robot.launch.py    # View robot in RViz
```

**Note**: On the real robot, motor control is handled by `navigator_bringup/scripts/ros2_hardware_interface.py` directly. `navigator_control` is only used with Gazebo simulation.

---

## navigator_description

**Purpose**: Robot model definition (URDF/Xacro) and mesh files.

```
navigator_description/
├── urdf/
│   ├── navigator.urdf.xacro           # Main robot (ros2_control)
│   ├── navigator_gazebo.urdf.xacro    # Gazebo simulation model
│   ├── ros2_control.xacro             # Hardware interfaces
│   ├── gazebo.xacro                   # Gazebo plugins
│   ├── bases/chassis.xacro            # Chassis
│   ├── wheels/wheel.xacro             # Wheel macros
│   └── sensors/
│       ├── lidar.xacro                # RPLidar
│       └── camera.xacro               # OV5647 camera mount
└── meshes/                            # STL mesh files
```

### Robot specifications

| Property | Value |
|----------|-------|
| Dimensions | 180mm × 120mm × 60mm |
| Wheel radius | 36mm |
| Wheel separation | 195mm |
| Base mass | ~5kg |
| Max speed | 0.22 m/s (Nav2 config) |

---

## navigator_msgs

**Purpose**: Custom ROS 2 message definitions.

| Message | Fields | Use case |
|---------|--------|---------|
| `WheelState` | velocity, position, effort | Wheel feedback |
| `SystemStatus` | is_ok, message | Health monitoring |
| `NavigatorStatus` | mode, motors_enabled, estop_pressed | Robot state |
| `MotorCommand` | left_velocity, right_velocity | Direct motor control |
| `BatteryStatus` | voltage, current, percentage, is_charging | Battery monitoring |

---

## navigator_nav

**Purpose**: Nav2 and SLAM configuration for the real robot.

```
navigator_nav/
├── config/
│   ├── nav2_params.yaml    # Nav2 parameters
│   ├── amcl_params.yaml    # AMCL localization
│   └── slam_params.yaml    # SLAM Toolbox config
├── launch/
│   ├── navigation.launch.py
│   ├── slam.launch.py
│   └── localization.launch.py
└── maps/                   # Saved maps
```

---

## navigator_sim

**Purpose**: Gazebo Fortress simulation environment.

```
navigator_sim/
├── worlds/
│   ├── empty.sdf           # Minimal world
│   ├── improved.sdf        # Better physics and lighting
│   ├── obstacles.sdf       # Obstacles for navigation testing
│   └── house.sdf           # Indoor house environment
├── launch/
│   ├── gazebo.launch.py               # Basic simulation
│   ├── gazebo_slam.launch.py          # Simulation + SLAM
│   ├── gazebo_slam_nav2.launch.py     # Full navigation stack
│   └── gazebo_navigation.launch.py    # Navigation with saved map
└── config/
    ├── nav2_params.yaml
|   ├── amcl_params.yaml 
    └── nav2_default_view.rviz
│   └── joint_state_config.yaml # Joint state publisher
├── save_map.sh                 # Map saving script
├── CMakeLists.txt
└── package.xml
```

### Simulation launch commands

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
```
navigator_bringup
    ├── navigator_description
    ├── navigator_control
    ├── navigator_nav
    ├── camera_ros          ← libcamera bridge for OV5647
    ├── rplidar_ros
    └── robot_state_publisher

navigator_sim
    ├── navigator_description
    ├── ros_gz_sim
    └── ros_gz_bridge
```
---

## Package Dependencies

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

## Quick Reference

| Task | Command |
|------|---------|
| Start real robot | `ros2 launch navigator_bringup hardware.launch.py` |
| SLAM mapping | `ros2 launch navigator_bringup navigation.launch.py slam:=true` |
| Autonomous navigation | `ros2 launch navigator_bringup navigation.launch.py map:=~/maps/map.yaml` |
| Run simulation | `ros2 launch navigator_sim gazebo.launch.py` |
| View robot model | `ros2 launch navigator_control view_robot.launch.py` |
| Calibrate camera | `ros2 launch navigator_bringup calibrate_camera.launch.py` |

---

[← Back to Main Documentation](../README.md)