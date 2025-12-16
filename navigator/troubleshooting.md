# Troubleshooting Guide

Solutions to common problems when working with the Navigator robot.

## Table of Contents

- [Build Issues](#build-issues)
- [Launch Problems](#launch-problems)
- [Simulation Issues](#simulation-issues)
- [Navigation Problems](#navigation-problems)
- [SLAM Issues](#slam-issues)
- [Control Problems](#control-problems)
- [Sensor Issues](#sensor-issues)
- [Performance Problems](#performance-problems)

---

## Build Issues

### "Package not found" errors

**Error:**
```
CMake Error: Could not find a package configuration file provided by "navigator_xxx"
```

**Solution:**
```bash
# Install dependencies
cd ~/navigator_ws
rosdep install --from-paths src --ignore-src -r -y

# Rebuild
colcon build --symlink-install
source install/setup.bash
```

### Missing dependencies

**Error:**
```
fatal error: xxx/xxx.h: No such file or directory
```

**Solution:**
```bash
# For Nav2
sudo apt install ros-humble-nav2-bringup

# For SLAM
sudo apt install ros-humble-slam-toolbox

# For control
sudo apt install ros-humble-ros2-control ros-humble-ros2-controllers

# For Gazebo
sudo apt install ros-humble-ros-gz
```

### Build fails with linker errors

**Solution:**
```bash
# Clean workspace
cd ~/navigator_ws
rm -rf build install log

# Rebuild with verbose output
colcon build --symlink-install --event-handlers console_direct+
```

---

## Launch Problems

### "No such file or directory" for launch file

**Error:**
```
Package 'navigator_xxx' not found
```

**Solution:**
```bash
# Source workspace
source ~/navigator_ws/install/setup.bash

# Verify package installed
ros2 pkg list | grep navigator

# If missing, rebuild
cd ~/navigator_ws
colcon build --packages-select navigator_xxx
```

### Launch file starts but nodes fail immediately

**Check:**
```bash
# See which nodes are running
ros2 node list

# Check for error messages
ros2 topic list
ros2 service list
```

**Solution:**
1. Check terminal output for specific errors
2. Verify all dependencies are installed
3. Ensure previous instances are killed:
```bash
killall -9 gzserver gzclient ruby rviz2
```

---

## Simulation Issues

### Gazebo won't start

**Symptoms**: Black screen or immediate crash

**Solutions:**

1. **Clear Gazebo cache:**
```bash
rm -rf ~/.ignition
rm -rf ~/.gz
```

2. **Verify installation:**
```bash
ign gazebo --version
# Should show Fortress version
```

3. **Try minimal world:**
```bash
ign gazebo -v 4 empty.sdf
```

4. **Check graphics drivers:**
```bash
glxinfo | grep "OpenGL version"
# Should show OpenGL 3.3 or higher
```

### Robot falls through ground

**Cause**: Physics plugin not loaded

**Solution:**

Check world file includes:
```xml
<plugin filename="libignition-gazebo-physics-system.so"
        name="ignition::gazebo::systems::Physics">
</plugin>
```

### Robot spawns but is frozen

**Symptoms**: Robot appears but doesn't respond to commands

**Check:**
```bash
# Verify bridges are running
ros2 node list | grep bridge

# Check if cmd_vel has subscribers
ros2 topic info /cmd_vel

# Try publishing directly
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist "{linear: {x: 0.2}}"
```

**Solution:**
1. Restart bridges
2. Check `gazebo.launch.py` includes all bridges
3. Verify robot URDF has diff drive plugin

### No sensor data

**Symptoms**: `/scan` or `/camera/image_raw` topics missing

**Check:**
```bash
# List available topics
ros2 topic list

# Check if sensors plugin loaded
ign topic -l | grep lidar
```

**Solution:**

Ensure world file has sensors plugin:
```xml
<plugin filename="libignition-gazebo-sensors-system.so"
        name="ignition::gazebo::systems::Sensors">
  <render_engine>ogre2</render_engine>
</plugin>
```

---

## Navigation Problems

### Robot won't reach goal

**Symptoms**: Plans path but stops or circles

**Debugging:**
```bash
# Check controller status
ros2 topic echo /controller_server/transition_event

# View costmaps
# In RViz, add GlobalCostmap and LocalCostmap displays

# Check for errors
ros2 node list
ros2 lifecycle get /controller_server
```

**Solutions:**

1. **Increase tolerance:**
```yaml
# In nav2_params.yaml
general_goal_checker:
  xy_goal_tolerance: 0.25  # Increase from 0.15
  yaw_goal_tolerance: 0.50 # Increase from 0.25
```

2. **Check costmap inflation:**
```yaml
inflation_layer:
  inflation_radius: 0.55  # Adjust based on robot size
```

3. **Verify path exists:**
- Check if goal is in free space
- Ensure no obstacles blocking
- Try closer goal first

### "No path found" errors

**Causes:**
- Goal in occupied/unknown space
- Start position incorrect
- Costmaps not initialized

**Solutions:**

1. **Set initial pose** (if using AMCL):
   - Click "2D Pose Estimate" in RViz
   - Click on robot's actual position

2. **Wait for costmaps:**
```bash
# Check if costmaps publishing
ros2 topic hz /global_costmap/costmap
ros2 topic hz /local_costmap/costmap
```

3. **Increase planner tolerance:**
```yaml
planner_server:
  GridBased:
    tolerance: 0.5  # Increase from 0.2
```

### Robot oscillates or gets stuck

**Symptoms**: Robot rocks back and forth

**Solutions:**

1. **Adjust DWB parameters:**
```yaml
FollowPath:
  trans_stopped_velocity: 0.25
  max_vel_x: 0.3  # Reduce from 0.5
  min_vel_x: 0.1  # Add minimum
```

2. **Increase progress checker tolerance:**
```yaml
progress_checker:
  required_movement_radius: 0.5
  movement_time_allowance: 10.0
```

---

## SLAM Issues

### No map appears

**Check:**
```bash
# Is SLAM running?
ros2 node list | grep slam

# Is map publishing?
ros2 topic hz /map
ros2 topic echo /map --once

# Is scan data available?
ros2 topic hz /scan
```

**Solution:**
```bash
# Restart SLAM with verbose output
ros2 launch navigator_sim slam.launch.py --screen
```

### Map quality is poor

**Symptoms**: Blurry walls, drift, disconnected rooms

**Solutions:**

1. **Drive slower** - Speed affects accuracy

2. **Adjust SLAM parameters:**
```python
'minimum_travel_distance': 0.05,  # More frequent updates
'minimum_travel_heading': 0.05,
'map_update_interval': 1.0,
```

3. **Better driving pattern:**
   - Overlap coverage
   - Return to start for loop closure
   - Face features directly

### Map doesn't update

**Cause**: Movement below threshold

**Solution:**
```bash
# Check current thresholds
ros2 param get /slam_toolbox minimum_travel_distance

# Reduce thresholds temporarily
ros2 param set /slam_toolbox minimum_travel_distance 0.05
ros2 param set /slam_toolbox minimum_travel_heading 0.05
```

---

## Control Problems

### Controllers won't start

**Error:**
```
[ERROR] Failed to activate controller
```

**Check:**
```bash
# List controllers
ros2 control list_controllers

# Check controller_manager
ros2 node list | grep controller_manager
```

**Solution:**
```bash
# Restart controllers
ros2 control load_controller joint_state_broadcaster
ros2 control load_controller diff_drive_controller

ros2 control set_controller_state joint_state_broadcaster start
ros2 control set_controller_state diff_drive_controller start
```

### Odometry drift

**Symptoms**: Robot position in RViz doesn't match actual position

**Solutions:**

1. **Check wheel parameters:**
```yaml
# In ros2_control.yaml
wheel_radius: 0.033      # Measure actual
wheel_separation: 0.145  # Measure actual
```

2. **Increase friction in simulation:**
```xml
<!-- In wheel.xacro -->
<mu1>100.0</mu1>
<mu2>100.0</mu2>
```

3. **Use sensor fusion:**
   - Add IMU
   - Use robot_localization package

### No cmd_vel response

**Check:**
```bash
# Is diff_drive_controller active?
ros2 control list_controllers

# Are commands being received?
ros2 topic echo /cmd_vel

# Check for velocity limits
ros2 param list /diff_drive_controller
```

---

## Sensor Issues

### Lidar not publishing

**Check:**
```bash
ros2 topic hz /scan
ros2 topic info /scan

# In Gazebo
ign topic -l | grep lidar
```

**Solution:**
1. Verify sensor plugin in `gazebo.xacro`
2. Check bridge is running
3. Restart simulation

### Camera image is black

**Causes:**
- Render engine not loaded
- Camera facing wrong direction
- Lighting issues in world

**Solutions:**

1. **Check render engine:**
```xml
<plugin filename="libignition-gazebo-sensors-system.so"
        name="ignition::gazebo::systems::Sensors">
  <render_engine>ogre2</render_engine>  <!-- Important -->
</plugin>
```

2. **View in RViz:**
```bash
ros2 run rqt_image_view rqt_image_view
```

3. **Add lighting to world:**
```xml
<light type="directional" name="sun">
  <cast_shadows>true</cast_shadows>
  <pose>0 0 10 0 0 0</pose>
  <diffuse>0.8 0.8 0.8 1</diffuse>
</light>
```

---

## Performance Problems

### Simulation is slow

**Solutions:**

1. **Reduce physics rate:**
```xml
<!-- In world file -->
<max_step_size>0.01</max_step_size>  <!-- Increase from 0.001 -->
```

2. **Reduce sensor rates:**
```xml
<!-- In gazebo.xacro -->
<update_rate>5</update_rate>  <!-- Reduce from 10 -->
```

3. **Disable camera:**
```bash
# Comment out camera bridge in gazebo.launch.py
```

4. **Close unnecessary programs**

### High CPU usage

**Check:**
```bash
top
# Look for gazebo, rviz2 processes
```

**Solutions:**
1. Reduce RViz displays
2. Lower sensor update rates
3. Use lighter world (empty.sdf)

### Memory issues

**Solutions:**

1. **Reduce SLAM parameters:**
```python
'scan_buffer_size': 10,  # Reduce from 20
'max_particles': 1000,   # For AMCL
```

2. **Clear ROS logs:**
```bash
ros2 daemon stop
rm -rf ~/.ros/log
ros2 daemon start
```

---

## General Debugging Commands

### Check system status

```bash
# All running nodes
ros2 node list

# All topics
ros2 topic list

# Topic rate
ros2 topic hz /topic_name

# Topic info
ros2 topic info /topic_name -v

# Node info
ros2 node info /node_name

# TF tree
ros2 run rqt_tf_tree rqt_tf_tree

# View transforms
ros2 run tf2_ros tf2_echo map odom
```

### Kill stuck processes

```bash
# Kill all ROS/Gazebo
killall -9 gzserver gzclient ruby rviz2

# Kill specific node
ros2 node list
# Find PID and kill
ps aux | grep node_name
kill -9 PID
```

### Reset everything

```bash
# Stop all
killall -9 gzserver gzclient ruby rviz2

# Clear cache
rm -rf ~/.ignition
rm -rf ~/.ros/log

# Rebuild workspace
cd ~/navigator_ws
rm -rf build install log
colcon build --symlink-install
source install/setup.bash
```

---

## Getting Help

If your issue isn't covered here:

1. **Check logs:** Look at terminal output for errors
2. **Search issues:** Check GitHub issues for similar problems
3. **Ask community:** ROS Answers, ROS Discourse
4. **Report bug:** Open issue with:
   - Error messages
   - Steps to reproduce
   - System information (`ros2 doctor`)

### Useful information to provide:

```bash
# ROS 2 version
echo $ROS_DISTRO

# System info
ros2 doctor

# Package versions
ros2 pkg list | grep navigator

# Current config
ros2 param dump /node_name
```

---

[← Back to Main Documentation](../README.md)