# Gazebo Simulation Guide

Complete guide to simulating the Navigator robot in Gazebo Fortress.

## Overview

The Navigator simulation provides:
- Realistic physics simulation
- Sensor simulation (Lidar, Camera)
- Differential drive control
- Multiple test environments
- Full integration with Nav2

## Prerequisites

```bash
# Install Gazebo Fortress
sudo apt install ignition-fortress

# Install ROS-Gazebo bridge
sudo apt install ros-humble-ros-gz

# Verify installation
ign gazebo --version
```

## Launch Options

### 1. Basic Simulation

Launch minimal simulation:

```bash
ros2 launch navigator_sim gazebo.launch.py
```

**What's included:**
- Gazebo world with obstacles
- Navigator robot model
- Differential drive controller
- Sensor bridges (Lidar, Camera)
- Odometry publisher
- TF broadcaster

### 2. Simulation with SLAM

Create maps while exploring:

```bash
ros2 launch navigator_sim gazebo_slam.launch.py
```

**What's included:**
- Everything from basic simulation
- SLAM Toolbox for mapping
- Map → Odom → Base_link transforms

### 3. Full Navigation Stack

Complete autonomous navigation:

```bash
ros2 launch navigator_sim gazebo_slam_nav2.launch.py
```

**What's included:**
- Everything from SLAM simulation
- Nav2 navigation stack
- Global and local costmaps
- Path planning
- Behavior trees

### 4. Navigation with Pre-saved Map

Use existing map for navigation:

```bash
ros2 launch navigator_sim gazebo_navigation.launch.py map:=~/maps/my_map.yaml
```

**What's included:**
- Basic simulation
- AMCL localization
- Nav2 navigation
- No SLAM (map is fixed)

## Simulation Worlds

### Available Worlds

The `navigator_sim/worlds/` directory contains:

#### 1. empty.sdf
- Minimal environment
- Ground plane and sun only
- Good for basic testing

#### 2. improved.sdf
- Better physics configuration
- Improved lighting
- Enhanced rendering
- Recommended for testing

#### 3. obstacles.sdf (Default)
- Multiple obstacles
- Walls and objects
- Various shapes (boxes, cylinders)
- Perfect for navigation testing

### Selecting a World

Edit `gazebo.launch.py`:

```python
# Change this line
world_file = os.path.join(nav_sim_share, "worlds", "obstacles.sdf")

# To use different world
world_file = os.path.join(nav_sim_share, "worlds", "improved.sdf")
```

Or specify via command line argument (if implemented).

## Sensor Configuration

### Lidar Sensor

**Specifications:**
- Type: GPU-accelerated ray-based
- Range: 0.15m - 10m
- Resolution: 360 samples (1° per sample)
- Update Rate: 10 Hz
- Noise: Gaussian (σ = 0.01)

**Topic:** `/scan`

**Visualization in RViz:**
```bash
ros2 topic echo /scan
ros2 topic hz /scan
```

### Camera Sensor

**Specifications:**
- Resolution: 640×480
- Format: RGB8
- FOV: 80° (1.396 rad)
- Update Rate: 30 Hz
- Range: 0.02m - 300m

**Topics:**
- `/camera/image_raw` - Image data
- `/camera/camera_info` - Camera parameters

**View camera feed:**
```bash
ros2 run rqt_image_view rqt_image_view
```

### Odometry

**Topic:** `/odom`

**Frame Chain:** `odom` → `base_link`

**Configuration:**
- Update Rate: 50 Hz
- Publishes: Position, velocity, orientation
- Includes: Covariance matrices

## Robot Control in Simulation

### Command Velocity

The robot accepts `geometry_msgs/Twist` messages on `/cmd_vel`:

```bash
# Publish single command
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist \
  "{linear: {x: 0.5}, angular: {z: 0.0}}"
```

### Velocity Limits

Defined in `gazebo.xacro`:
- Max linear: 1.0 m/s
- Max angular: 2.0 rad/s
- Max acceleration: 2.0 m/s²

### Teleop Control

#### Keyboard:
```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

#### Gamepad:
```bash
ros2 launch teleop_twist_joy teleop-launch.py
```

## ROS-Gazebo Bridges

Active bridges in simulation:

| Topic | Message Type | Direction |
|-------|-------------|-----------|
| `/cmd_vel` | Twist | ROS → Gazebo |
| `/scan` | LaserScan | Gazebo → ROS |
| `/camera/image_raw` | Image | Gazebo → ROS |
| `/camera/camera_info` | CameraInfo | Gazebo → ROS |
| `/odom` | Odometry | Gazebo → ROS |
| `/tf` | TFMessage | Gazebo → ROS |
| `/clock` | Clock | Gazebo → ROS |

## Visualization with RViz

### Launch RViz with Nav2 Config

```bash
ros2 launch navigator_sim rviz_nav2.launch.py
```

### RViz Displays

Pre-configured displays include:
- **Grid**: Reference grid
- **RobotModel**: 3D robot visualization
- **TF**: Transform frames
- **LaserScan**: Lidar visualization
- **Map**: SLAM or loaded map
- **GlobalCostmap**: Global planning costmap
- **LocalCostmap**: Local planning costmap
- **GlobalPath**: Planned global path
- **LocalPath**: Local trajectory
- **Camera**: Live camera feed

### Setting Navigation Goals

1. Ensure map is visible
2. Click "2D Pose Estimate" (set initial pose if using AMCL)
3. Click "Nav2 Goal"
4. Click on map where you want robot to go
5. Drag to set goal orientation

## Performance Tuning

### If Simulation is Slow

1. **Reduce physics update rate:**

Edit world file:
```xml
<max_step_size>0.001</max_step_size>
<!-- Change to -->
<max_step_size>0.01</max_step_size>
```

2. **Disable camera:**

Comment out camera bridge in `gazebo.launch.py`

3. **Reduce lidar samples:**

Edit `gazebo.xacro`:
```xml
<samples>360</samples>
<!-- Change to -->
<samples>180</samples>
```

### If Robot Drifts or Slides

1. **Increase wheel friction:**

Edit `gazebo.xacro`:
```xml
<mu1>100.0</mu1>
<mu2>100.0</mu2>
```

2. **Adjust contact parameters:**
```xml
<kp>10000000.0</kp>
<kd>1000.0</kd>
```

## Debugging

### Check Active Nodes

```bash
ros2 node list
```

Expected nodes:
- `/gazebo`
- `/robot_state_publisher`
- `/joint_state_publisher`
- Multiple `/parameter_bridge` nodes

### Verify Bridges

```bash
# Check if topics exist
ros2 topic list

# Check bridge nodes
ros2 node list | grep bridge
```

### Monitor TF Tree

```bash
ros2 run rqt_tf_tree rqt_tf_tree
```

Expected transforms:
```
odom → base_link → {left_wheel_link, right_wheel_link, lidar_link, camera_link}
```

### Gazebo Verbose Output

```bash
ign gazebo -v 4 -r obstacles.sdf
```

## Common Issues

### Robot Falls Through Ground

**Cause**: Physics not loaded

**Solution**: Ensure physics plugin in world file:
```xml
<plugin filename="libignition-gazebo-physics-system.so"
        name="ignition::gazebo::systems::Physics">
</plugin>
```

### No Lidar Data

**Cause**: Sensor system not loaded

**Solution**: Check world file includes:
```xml
<plugin filename="libignition-gazebo-sensors-system.so"
        name="ignition::gazebo::systems::Sensors">
  <render_engine>ogre2</render_engine>
</plugin>
```

### Robot Doesn't Move

**Cause**: No cmd_vel or bridge issue

**Solutions**:
1. Check bridge: `ros2 topic info /cmd_vel`
2. Verify subscribers: `ros2 topic info /cmd_vel -v`
3. Manually publish: `ros2 topic pub /cmd_vel ...`

### Camera Not Working

**Cause**: Render engine not specified

**Solution**: Verify sensor plugin has render engine specified (shown above)

## Recording and Playback

### Record Simulation Data

```bash
# Record all topics
ros2 bag record -a

# Record specific topics
ros2 bag record /scan /odom /camera/image_raw
```

### Playback Recorded Data

```bash
ros2 bag play my_recording.db3
```

## Advanced: Custom Worlds

Create your own world file:

```xml
<?xml version="1.0" ?>
<sdf version="1.9">
  <world name="my_world">
    <!-- Physics -->
    <physics name="1ms" type="ignored">
      <max_step_size>0.001</max_step_size>
      <real_time_factor>1.0</real_time_factor>
    </physics>
    
    <!-- Plugins -->
    <plugin filename="libignition-gazebo-physics-system.so"
            name="ignition::gazebo::systems::Physics"/>
    <plugin filename="libignition-gazebo-sensors-system.so"
            name="ignition::gazebo::systems::Sensors">
      <render_engine>ogre2</render_engine>
    </plugin>
    
    <!-- Ground -->
    <include>
      <uri>model://ground_plane</uri>
    </include>
    
    <!-- Light -->
    <include>
      <uri>model://sun</uri>
    </include>
    
    <!-- Add your models here -->
  </world>
</sdf>
```

Save to `navigator_sim/worlds/my_world.sdf`

## Tips for Effective Simulation

1. **Start Simple**: Test with empty world first
2. **Use Verbose Mode**: Launch with `-v 4` for debugging
3. **Monitor Performance**: Check `ign stats` while running
4. **Save Maps**: Don't lose your mapping work
5. **Test Incrementally**: Add complexity gradually

---

[← Back to Main Documentation](../README.md) | [Next: SLAM Mapping →](slam.md)