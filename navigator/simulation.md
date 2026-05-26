# Gazebo Simulation Guide

Complete guide to simulating the Navigator robot in Gazebo Fortress.


![Screenshot](/images/gazebo.png)

## Overview

The Navigator simulation provides:
- Realistic physics simulation
- Sensor simulation (Lidar, Camera)
- Differential drive control
- Multiple test environments including the Navigator World benchmark arena
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

Launch in the Navigator World (recommended starting point):

```bash
ros2 launch navigator_sim navigator_world.launch.py
```

Or launch in a custom world:

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

To use the Navigator World with SLAM:

```bash
ros2 launch navigator_sim gazebo_slam.launch.py \
  world:=$(ros2 pkg prefix navigator_sim)/share/navigator_sim/worlds/navigator_world.sdf
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

To use the Navigator World with the full stack:

```bash
ros2 launch navigator_sim gazebo_slam_nav2.launch.py \
  world:=$(ros2 pkg prefix navigator_sim)/share/navigator_sim/worlds/navigator_world.sdf
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

---

## Simulation Worlds

### Available Worlds

The `navigator_sim/worlds/` directory contains:

#### 1. navigator_world.sdf (Recommended)
The Navigator benchmark environment — an 8 m × 8 m walled arena purpose-built for SLAM and Nav2 testing, inspired by TurtleBot3 World.

Launch it directly:

```bash
ros2 launch navigator_sim navigator_world.launch.py
```

#### 2. empty.sdf
Minimal environment with ground plane and sun only. Good for basic testing and verifying the robot spawns correctly.

#### 3. improved.sdf
Better physics configuration, improved lighting, enhanced rendering. Recommended when you want a clean empty space with good visuals.

#### 4. obstacles.sdf
Multiple scattered obstacles, walls, and objects of various shapes. Good for quick navigation testing without the structured benchmark layout.

#### 5. house.sdf
Full indoor house environment with rooms, corridors, furniture, and doorways. Most realistic environment — use this once SLAM and Nav2 are tuned.

### Selecting a World

#### Using the dedicated launch file (navigator_world only):

```bash
ros2 launch navigator_sim navigator_world.launch.py
```

#### Passing a world to any launch file via argument:

```bash
# SLAM with the house world
ros2 launch navigator_sim gazebo_slam.launch.py \
  world:=$(ros2 pkg prefix navigator_sim)/share/navigator_sim/worlds/house.sdf

# Basic simulation with the empty world
ros2 launch navigator_sim gazebo.launch.py \
  world:=$(ros2 pkg prefix navigator_sim)/share/navigator_sim/worlds/empty.sdf
```

#### Changing the default inside a launch file:

Edit `gazebo.launch.py` and update the `default_value`:

```python
declare_world_arg = DeclareLaunchArgument(
    'world',
    default_value=os.path.join(nav_sim_share, "worlds", "navigator_world.sdf"),
    ...
)
```

---

## Gazebo Camera — Fixing the Fisheye Look

The default Gazebo orbit camera uses a very wide field of view which causes a fisheye/barrel distortion effect, especially when zoomed in close to the robot. There are two ways to fix it.

### Quick fix (GUI)

In the Gazebo window:

```
View → Camera → Field of View → set to 60
```

60° is the standard comfortable FOV. Values above 90° cause noticeable distortion.

### Permanent fix (world file)

Add this block inside the `<world>` tag of your SDF file, just before `</world>`:

```xml
<gui fullscreen="0">
  <camera name="user_camera">
    <pose>-6 -6 5 0 0.5 0.785</pose>
    <view_controller>orbit</view_controller>
    <projection_type>perspective</projection_type>
  </camera>
</gui>
```

The `<pose>` values are `x y z roll pitch yaw`. This example places the camera at a 45° angle looking down at the arena from the SW corner — the standard TurtleBot3-style view.

Adjust to taste:

| Goal | Change |
|------|--------|
| Move closer | Reduce x/y (e.g. `-4 -4 4`) |
| More top-down | Increase pitch (e.g. `0.8`) |
| Different angle | Change yaw (`0.785` = 45°, `1.57` = 90°) |

The `navigator_world.sdf` already includes this block with a good default view.

---

## Sensor Configuration

### Lidar Sensor

**Specifications:**
- Type: GPU-accelerated ray-based
- Range: 0.15m - 10m
- Resolution: 360 samples (1° per sample)
- Update Rate: 10 Hz
- Noise: Gaussian (σ = 0.01)

**Topic:** `/scan`

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
- `/camera/image_raw` — Image data
- `/camera/camera_info` — Camera parameters

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

---

## Robot Control in Simulation

### Command Velocity

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

```bash
# Keyboard
ros2 run teleop_twist_keyboard teleop_twist_keyboard

# Gamepad
ros2 launch teleop_twist_joy teleop-launch.py
```

---

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

---

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

---

## Performance Tuning

### If Simulation is Slow

1. **Reduce physics update rate** — edit world file:

```xml
<max_step_size>0.01</max_step_size>
```

2. **Disable camera** — comment out camera bridge in `gazebo.launch.py`

3. **Reduce lidar samples** — edit `gazebo.xacro`:

```xml
<samples>180</samples>
```

### If Robot Drifts or Slides

1. **Increase wheel friction** in `gazebo.xacro`:

```xml
<mu1>100.0</mu1>
<mu2>100.0</mu2>
```

2. **Adjust contact parameters:**

```xml
<kp>10000000.0</kp>
<kd>1000.0</kd>
```

---

## Debugging

### Check Active Nodes

```bash
ros2 node list
```

Expected nodes: `/gazebo`, `/robot_state_publisher`, `/joint_state_publisher`, multiple `/parameter_bridge` nodes.

### Verify Bridges

```bash
ros2 topic list
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
ign gazebo -v 4 -r navigator_world.sdf
```

---

## Common Issues

### Fisheye / Distorted View in Gazebo

**Cause**: Default Gazebo camera has a very wide FOV.

**Quick fix**: `View → Camera → Field of View → 60`

**Permanent fix**: Add a `<gui>` camera block to your world SDF — see [Fixing the Fisheye Look](#gazebo-camera--fixing-the-fisheye-look) above.

### Robot Falls Through Ground

**Cause**: Physics plugin not loaded.

**Solution**: Ensure physics plugin in world file:

```xml
<plugin filename="libignition-gazebo-physics-system.so"
        name="ignition::gazebo::systems::Physics">
</plugin>
```

### No Lidar Data

**Cause**: Sensor system not loaded.

**Solution**: Check world file includes:

```xml
<plugin filename="libignition-gazebo-sensors-system.so"
        name="ignition::gazebo::systems::Sensors">
  <render_engine>ogre2</render_engine>
</plugin>
```

### Robot Doesn't Move

**Cause**: No cmd_vel or bridge issue.

**Solutions**:
1. Check bridge: `ros2 topic info /cmd_vel`
2. Verify subscribers: `ros2 topic info /cmd_vel -v`
3. Manually publish: `ros2 topic pub /cmd_vel ...`

### Camera Not Working

**Cause**: Render engine not specified.

**Solution**: Verify sensor plugin has `<render_engine>ogre2</render_engine>`.

---

## Recording and Playback

```bash
# Record all topics
ros2 bag record -a

# Record specific topics
ros2 bag record /scan /odom /camera/image_raw

# Playback
ros2 bag play my_recording.db3
```

---

## Advanced: Custom Worlds

Create your own world file:

```xml
<?xml version="1.0" ?>
<sdf version="1.9">
  <world name="my_world">
    <physics name="1ms" type="ignored">
      <max_step_size>0.001</max_step_size>
      <real_time_factor>1.0</real_time_factor>
    </physics>

    <plugin filename="libignition-gazebo-physics-system.so"
            name="ignition::gazebo::systems::Physics"/>
    <plugin filename="libignition-gazebo-sensors-system.so"
            name="ignition::gazebo::systems::Sensors">
      <render_engine>ogre2</render_engine>
    </plugin>

    <!-- Optional: set a comfortable starting camera view -->
    <gui fullscreen="0">
      <camera name="user_camera">
        <pose>-6 -6 5 0 0.5 0.785</pose>
        <view_controller>orbit</view_controller>
        <projection_type>perspective</projection_type>
      </camera>
    </gui>

    <include><uri>model://ground_plane</uri></include>
    <include><uri>model://sun</uri></include>

    <!-- Add your models here -->
  </world>
</sdf>
```

Save to `navigator_sim/worlds/my_world.sdf`, then launch:

```bash
ros2 launch navigator_sim gazebo.launch.py \
  world:=$(ros2 pkg prefix navigator_sim)/share/navigator_sim/worlds/my_world.sdf
```

---

## Tips for Effective Simulation

1. **Start with Navigator World** — it's designed for SLAM and Nav2 and has a comfortable camera angle out of the box
2. **Use verbose mode** — launch with `-v 4` for debugging
3. **Fix the camera first** — set FOV to 60° if anything looks distorted
4. **Save maps** — don't lose your SLAM work
5. **Test incrementally** — verify basic motion before adding SLAM, then Nav2

---

[← Back to Main Documentation](../README.md) | [Next: SLAM Mapping →](slam.md)