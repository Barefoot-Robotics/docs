# Quick Start Guide

Get your Navigator robot running in simulation in just a few minutes!

## Prerequisites

- Completed [Installation Guide](installation.md)
- ROS 2 workspace built and sourced

##  Launch Your First Simulation

### Option 1: Simple Gazebo Simulation

Launch the robot in Gazebo:

```bash
ros2 launch navigator_sim gazebo.launch.py
```

This will:
- Start Gazebo Fortress
- Spawn the Navigator robot
- Start all necessary bridges
- Publish robot transforms

### Option 2: Simulation with Visualization

Launch with RViz for visualization:

```bash
# Terminal 1: Start Gazebo
ros2 launch navigator_sim gazebo.launch.py

# Terminal 2: Start RViz (after Gazebo loads)
ros2 launch navigator_sim rviz_nav2.launch.py
```

##  Control the Robot

### Using Keyboard Teleop

```bash
# In a new terminal
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

**Controls:**
- `i` - Move forward
- `k` - Stop
- `,` - Move backward
- `j` - Turn left
- `l` - Turn right
- `u/o` - Move in arcs
- `q/z` - Increase/decrease max speeds

### Using Joy/Gamepad

```bash
# Install joy package (if not already)
sudo apt install ros-humble-joy

# Launch joy node
ros2 launch teleop_twist_joy teleop-launch.py
```

##  Create Your First Map (SLAM)

### Start SLAM Mapping

```bash
# Terminal 1: Launch Gazebo simulation
ros2 launch navigator_sim gazebo.launch.py

# Terminal 2: Launch SLAM
ros2 launch navigator_sim slam.launch.py

# Terminal 3: Launch RViz
ros2 launch navigator_sim rviz_nav2.launch.py

# Terminal 4: Control the robot
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

### Drive Around to Map

1. Drive the robot slowly using keyboard teleop
2. Cover all areas you want mapped
3. Watch the map build in RViz (gray = unknown, white = free, black = obstacles)

### Save Your Map

```bash
# Create directory for maps
mkdir -p ~/navigator_maps

# Save the map
ros2 run nav2_map_server map_saver_cli -f ~/navigator_maps/my_first_map
```

You'll get two files:
- `my_first_map.yaml` - Map metadata
- `my_first_map.pgm` - Map image

##  Navigate Autonomously

### Using a Saved Map

```bash
# Launch navigation with your map
ros2 launch navigator_sim gazebo_navigation.launch.py map:=~/navigator_maps/my_first_map.yaml
```

### Set Navigation Goals

In RViz:
1. Click "2D Pose Estimate" button
2. Click on the map where the robot is and drag to set orientation
3. Click "Nav2 Goal" button
4. Click on the map where you want the robot to go
5. Watch the robot navigate autonomously!

##  Monitor Robot Status

Check various topics:

```bash
# Odometry
ros2 topic echo /odom

# Laser scan
ros2 topic echo /scan

# Camera feed
ros2 topic hz /camera/image_raw

# Joint states
ros2 topic echo /joint_states

# Transform tree
ros2 run rqt_tf_tree rqt_tf_tree
```

##  Quick Command Reference

### Launch Commands

```bash
# Gazebo only
ros2 launch navigator_sim gazebo.launch.py

# Gazebo + SLAM
ros2 launch navigator_sim gazebo_slam.launch.py

# Gazebo + SLAM + Nav2
ros2 launch navigator_sim gazebo_slam_nav2.launch.py

# Gazebo + Navigation (with map)
ros2 launch navigator_sim gazebo_navigation.launch.py map:=~/path/to/map.yaml

# RViz with Nav2 config
ros2 launch navigator_sim rviz_nav2.launch.py
```

### Useful Commands

```bash
# List all nodes
ros2 node list

# List all topics
ros2 topic list

# View robot model in RViz
ros2 launch navigator_control view_robot.launch.py

# Kill all ROS/Gazebo processes
killall -9 gzserver gzclient ruby rviz2
```

##  Common First-Time Issues

### Robot Not Moving

**Problem**: Robot doesn't respond to commands

**Solution**:
```bash
# Check if cmd_vel is being published
ros2 topic hz /cmd_vel

# Verify bridges are running
ros2 node list | grep bridge
```

### No Map in RViz

**Problem**: Map doesn't appear in RViz

**Solution**:
1. Check Fixed Frame is set to "map" in RViz
2. Verify SLAM is running: `ros2 node list | grep slam`
3. Add "Map" display in RViz if missing

### Gazebo Crashes

**Problem**: Gazebo closes unexpectedly

**Solution**:
```bash
# Clear Gazebo cache
rm -rf ~/.ignition

# Restart with verbose output
ign gazebo -v 4
```

##  What's Next?

Now that you have the basics working:

- Learn about [SLAM Mapping](slam.md) in detail
- Explore [Autonomous Navigation](navigation.md)
- Understand [Robot Control](control.md)
- Try different [Simulation Worlds](worlds.md)

## 💡 Tips for Success

1. **Start Simple**: Get basic simulation working before adding complexity
2. **One Terminal at a Time**: Launch components separately to identify issues
3. **Check Topics**: Use `ros2 topic list` and `ros2 topic echo` frequently
4. **Save Maps Often**: Don't lose your mapping work
5. **Use RViz**: Visual feedback makes debugging much easier

---

[← Back to Main Documentation](../README.md) | [Next: Package Overview →](packages.md)