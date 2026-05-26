# Autonomous Navigation

Learn how to use Nav2 for autonomous navigation with the Navigator robot.

## What is Autonomous Navigation?

Autonomous navigation allows the robot to:
- Plan paths to goal locations
- Avoid obstacles dynamically
- Recover from failures
- Navigate complex environments

## Prerequisites

✅ Completed [SLAM Mapping](slam.md) and have a saved map  
✅ Understanding of basic ROS 2 concepts

![auto_nav](</images/auto_navigation.png>)
## Quick Start - Navigation with Saved Map

### 1. Launch Navigation Stack

```bash
# Launch Gazebo + Navigation with your map
ros2 launch navigator_sim gazebo_navigation.launch.py map:=~/navigator_maps/my_map.yaml
```

**What launches:**
- Gazebo simulation
- AMCL localization
- Nav2 navigation stack
- All costmaps and planners

### 2. Open RViz

```bash
ros2 launch navigator_sim rviz_nav2.launch.py
```

### 3. Set Initial Pose

The robot needs to know where it is on the map:

1. Click **"2D Pose Estimate"** button in RViz
2. Click on the map where the robot actually is
3. Drag to set the orientation
4. Release

You should see the lidar scan align with the map.

### 4. Set Navigation Goal

1. Click **"Nav2 Goal"** button in RViz
2. Click where you want the robot to go
3. Drag to set the goal orientation
4. Release

The robot will:
- Plan a path (green line)
- Start moving
- Avoid obstacles
- Reach the goal

## Navigation Components

### Global Planner
Plans the overall path from start to goal:
- Uses A* algorithm
- Plans on global costmap
- Updates when goal changes

### Local Planner (DWB)
Generates velocity commands to follow the path:
- Samples possible trajectories
- Scores them based on:
  - Path following
  - Obstacle avoidance
  - Goal alignment
- Selects best trajectory

### Recovery Behaviors
What robot does when stuck:
1. **Clear costmap** - Clear old obstacles
2. **Rotate** - Spin to find clear path
3. **Back up** - Reverse a bit
4. **Wait** - Pause and reassess

## Costmaps

### Global Costmap
- Uses the static map
- Adds current obstacles
- For long-range planning

### Local Costmap
- Small area around robot
- Rolling window
- For real-time obstacle avoidance

### Costmap Layers
1. **Static Layer** - From saved map
2. **Obstacle Layer** - From sensors
3. **Inflation Layer** - Safety buffer around obstacles

## Configuration

### Key Parameters

**Planner Speed:**
```yaml
planner_server:
  expected_planner_frequency: 20.0  # Hz
```

**Controller Speed:**
```yaml
controller_server:
  controller_frequency: 20.0  # Hz
```

**Robot Velocity Limits:**
```yaml
FollowPath:
  max_vel_x: 0.5      # m/s
  max_vel_theta: 1.0  # rad/s
```

**Goal Tolerance:**
```yaml
general_goal_checker:
  xy_goal_tolerance: 0.15    # meters
  yaw_goal_tolerance: 0.25   # radians
```

## Common Tasks

### Navigate to Multiple Waypoints

```bash
# Create waypoints file: waypoints.yaml
waypoints:
  - {x: 1.0, y: 0.0, theta: 0.0}
  - {x: 2.0, y: 1.0, theta: 1.57}
  - {x: 0.0, y: 2.0, theta: 3.14}

# Use Nav2 waypoint follower
ros2 action send_goal /follow_waypoints nav2_msgs/action/FollowWaypoints ...
```

### Cancel Navigation

```bash
# Cancel current goal
ros2 action send_goal /navigate_to_pose nav2_msgs/action/NavigateToPose --cancel
```

### Check Navigation Status

```bash
# Check if navigation is active
ros2 topic echo /navigate_to_pose/_action/status

# Monitor current goal
ros2 topic echo /goal_pose
```

## Behavior Trees

Nav2 uses behavior trees for decision making.

**Default behavior tree flow:**
```
NavigateToPose
├── ComputePathToPose (Global Planner)
├── FollowPath (Local Planner)
└── Recovery
    ├── ClearCostmap
    ├── Spin
    ├── BackUp
    └── Wait
```

### Custom Behavior Trees

Create your own navigation logic:

```xml
<!-- custom_bt.xml -->
<root main_tree_to_execute="MainTree">
  <BehaviorTree ID="MainTree">
    <Sequence>
      <ComputePathToPose/>
      <FollowPath/>
    </Sequence>
  </BehaviorTree>
</root>
```

Use it:
```yaml
bt_navigator:
  default_nav_to_pose_bt_xml: /path/to/custom_bt.xml
```

## Tuning Navigation

### If Robot is Too Slow
```yaml
FollowPath:
  max_vel_x: 0.8  # Increase from 0.5
```

### If Robot is Too Aggressive
```yaml
FollowPath:
  acc_lim_x: 1.5  # Decrease from 2.5
```

### If Robot Gets Too Close to Obstacles
```yaml
inflation_layer:
  inflation_radius: 0.7  # Increase from 0.55
```

### If Robot Can't Find Path
```yaml
planner_server:
  GridBased:
    tolerance: 0.8  # Increase from 0.5
    allow_unknown: true
```

## Advanced Features

### Dynamic Obstacles

Nav2 automatically handles moving obstacles using sensor data.

### Cost-Aware Planning

Plan paths that prefer certain areas:

```yaml
static_layer:
  map_topic: /map
  # Prefer certain zones
```

### Multiple Planners

Switch between planners for different scenarios:

```yaml
planner_server:
  planner_plugins: ["GridBased", "Smac2D"]
  
  GridBased:
    plugin: "nav2_navfn_planner/NavfnPlanner"
  
  Smac2D:
    plugin: "nav2_smac_planner/SmacPlanner2D"
```

Select planner:
```bash
ros2 param set /planner_server planner "Smac2D"
```

## Monitoring Navigation

### View in RViz

Add these displays:
- **Global Path** - Planned path
- **Local Path** - Current trajectory
- **Global Costmap** - Planning map
- **Local Costmap** - Obstacle avoidance map

### Command Line

```bash
# Current robot pose
ros2 topic echo /amcl_pose

# Current goal
ros2 topic echo /goal_pose

# Navigation feedback
ros2 topic echo /navigate_to_pose/_action/feedback
```

## Troubleshooting

### Robot Won't Start Moving

**Check:**
```bash
# Is goal set?
ros2 topic echo /goal_pose

# Is planner running?
ros2 lifecycle get /planner_server
```

### Robot Oscillates

**Solution:**
```yaml
FollowPath:
  trans_stopped_velocity: 0.25
  Oscillation:
    oscillation_distance: 0.05
```

### Path Not Found

**Solutions:**
1. Increase planner tolerance
2. Check if goal is in free space
3. Verify map is loaded correctly

### Robot Gets Stuck

**Solutions:**
1. Enable recovery behaviors
2. Increase costmap clearing frequency
3. Tune local planner parameters

## Performance Tips

1. **Reduce update rates** if CPU is high
2. **Decrease costmap size** for faster planning
3. **Use simpler planners** for basic environments
4. **Tune DWB critics** for specific behaviors

## Navigation Patterns

### Patrol Route
```python
# patrol.py
waypoints = [
    {'x': 1.0, 'y': 0.0},
    {'x': 2.0, 'y': 1.0},
    {'x': 1.0, 'y': 2.0},
    {'x': 0.0, 'y': 1.0}
]

while True:
    for wp in waypoints:
        navigate_to(wp)
```

### Return to Dock
```python
# When battery low, return to charging station
if battery.percentage < 20:
    navigate_to({'x': 0.0, 'y': 0.0})  # Dock position
```

## Next Steps

- Learn about [Localization](localization.md) for better pose estimation
- Explore [Parameter Tuning](tuning.md) for optimal performance
- Check [Troubleshooting](troubleshooting.md) for common issues

---

[← Back to Main Documentation](../README.md) | [Next: Localization →](localization.md)