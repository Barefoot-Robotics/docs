# SLAM Mapping Guide

Learn how to create accurate maps of your environment using SLAM (Simultaneous Localization and Mapping) Toolbox.

## What is SLAM?

SLAM allows the robot to:
- Build a map of an unknown environment
- Localize itself within that map
- Update the map as it explores

## Prerequisites

```bash
# Install SLAM Toolbox
sudo apt install ros-humble-slam-toolbox

# Verify installation
ros2 pkg list | grep slam_toolbox
```

## Quick Start

### 1. Launch SLAM in Simulation

```bash
# Terminal 1: Start Gazebo
ros2 launch navigator_sim gazebo.launch.py

# Terminal 2: Start SLAM
ros2 launch navigator_sim slam.launch.py

# Terminal 3: Visualize in RViz
ros2 launch navigator_sim rviz_nav2.launch.py

# Terminal 4: Control robot
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

### 2. Drive and Map

- Drive slowly (0.2-0.3 m/s linear)
- Rotate slowly (0.3-0.5 rad/s angular)
- Cover all areas systematically
- Return to starting point for loop closure

### 3. Save Your Map

```bash
# Option 1: Using provided script
cd ~/navigator_ws/src/navigator/navigator_sim
./save_map.sh my_warehouse_map

# Option 2: Manual save
mkdir -p ~/navigator_maps
ros2 run nav2_map_server map_saver_cli -f ~/navigator_maps/my_map
```

## SLAM Configuration

### Key Parameters

Located in `navigator_sim/launch/slam.launch.py`:

```python
parameters=[{
    'use_sim_time': True,
    'odom_frame': 'odom',
    'map_frame': 'map',
    'base_frame': 'base_link',
    'scan_topic': '/scan',
    'mode': 'mapping',
    
    # Update rates
    'map_update_interval': 1.0,      # Update map every 1 second
    'transform_publish_period': 0.02, # 50 Hz transform updates
    
    # Matching parameters
    'minimum_travel_distance': 0.1,   # Process scan every 10cm
    'minimum_travel_heading': 0.1,    # Or every 0.1 radian
    
    # Quality settings
    'resolution': 0.05,               # 5cm per pixel
    'max_laser_range': 10.0,          # Use points up to 10m
}]
```

### Understanding Parameters

#### Update Rates

| Parameter | Value | Description |
|-----------|-------|-------------|
| `map_update_interval` | 1.0 | How often map is published (Hz) |
| `transform_publish_period` | 0.02 | TF update rate (50 Hz) |
| `minimum_time_interval` | 0.2 | Minimum time between scans |

#### Movement Thresholds

| Parameter | Value | Purpose |
|-----------|-------|---------|
| `minimum_travel_distance` | 0.1 | Min distance before processing (m) |
| `minimum_travel_heading` | 0.1 | Min rotation before processing (rad) |

Lower values = more frequent updates = better accuracy but slower performance

#### Quality Settings

| Parameter | Value | Impact |
|-----------|-------|--------|
| `resolution` | 0.05 | Map detail (5cm cells) |
| `max_laser_range` | 10.0 | Max useful lidar distance |
| `max_beams` | 60 | Rays used per scan |

## Mapping Best Practices

### 1. Driving Techniques

**DO:**
- ✅ Drive slowly and smoothly
- ✅ Make gradual turns
- ✅ Overlap your coverage
- ✅ Return to known areas for loop closure
- ✅ Face features directly when possible

**DON'T:**
- ❌ Drive too fast (>0.3 m/s)
- ❌ Make sharp turns
- ❌ Skip areas
- ❌ Only drive in straight lines
- ❌ Stay too far from walls

### 2. Environmental Considerations

**Good Environments:**
- Structured rooms with walls
- Corridors with features
- Cluttered areas with objects
- Static environments

**Challenging Environments:**
- Large open spaces
- Highly symmetric rooms
- Dynamic environments (moving objects)
- Glass walls or mirrors
- Very dark or bright areas

### 3. Coverage Pattern

```
Start here ──┐
             │
    ┌────────┴────────┐
    │        │        │
    │    →   │   →    │
    │        │        │
    │   ←────┘        │
    │                 │
    └─────────────────┘
     Return for loop closure
```

**Recommended Pattern:**
1. Map perimeter first
2. Fill in interior
3. Return to start for loop closure
4. Revisit any unclear areas

## Monitoring SLAM Performance

### Check Map Topic

```bash
# See map updates
ros2 topic hz /map

# Echo map data
ros2 topic echo /map --once
```

### View Transform Tree

```bash
# Check transform chain
ros2 run rqt_tf_tree rqt_tf_tree
```

Expected: `map` → `odom` → `base_link`

### Monitor SLAM Node

```bash
# Check if running
ros2 node list | grep slam

# See parameters
ros2 param list /slam_toolbox

# Get specific parameter
ros2 param get /slam_toolbox resolution
```

## Map Quality Assessment

### Good Map Indicators

✅ **Sharp edges**: Walls are straight lines
✅ **Closed loops**: Rooms properly connected
✅ **No drift**: Returning to start matches original position
✅ **Consistent thickness**: Walls have uniform width
✅ **Clear obstacles**: Objects are distinct

### Poor Map Indicators

❌ **Blurry edges**: Multiple overlapping walls
❌ **Disconnected rooms**: Gaps in walls
❌ **Drift**: Start and end positions don't match
❌ **Varying thickness**: Walls expand/contract
❌ **Ghost objects**: Artifacts from motion

### Fixing Poor Maps

**Problem**: Blurry walls

**Solution**: 
- Drive slower
- Reduce `minimum_travel_distance` to 0.05
- Increase `map_update_interval` to 2.0

**Problem**: Disconnected areas

**Solution**:
- Revisit problematic areas
- Drive through doorways multiple times
- Ensure loop closure

**Problem**: Drift over time

**Solution**:
- Return to starting position
- Wait for loop closure to complete
- Remap area if severe

## Advanced SLAM Features

### Loop Closure

SLAM detects when robot returns to previously mapped area:

```python
# In configuration
'link_match_minimum_response_fine': 0.1,  # Sensitivity
'use_scan_matching': True,                # Enable matching
'use_scan_barycenter': True,              # Better matching
```

**Trigger loop closure:**
1. Return to starting location
2. Face same direction
3. Wait 5-10 seconds
4. Watch map "snap" into place

### Localization Mode

After mapping, use map for localization:

```python
'mode': 'localization',  # Change from 'mapping'
```

This:
- Uses existing map
- No map updates
- Only localizes robot
- More efficient

### Serialization (Save Map State)

Save complete SLAM state for later:

```bash
# During mapping
ros2 service call /slam_toolbox/serialize_map slam_toolbox/srv/SerializePoseGraph "filename: '/tmp/my_slam_map'"
```

This saves:
- Map data
- Pose graph
- Loop closures
- Can resume mapping later

## Map File Formats

### Output Files

When you save a map, you get:

#### 1. map_name.yaml
```yaml
image: map_name.pgm
mode: trinary
resolution: 0.05
origin: [-10.0, -10.0, 0]
negate: 0
occupied_thresh: 0.65
free_thresh: 0.25
```

#### 2. map_name.pgm
- Grayscale image
- Black (0) = Obstacle
- White (255) = Free space
- Gray (127) = Unknown

### Editing Maps

You can edit maps in any image editor:

```bash
# Open in GIMP
gimp ~/navigator_maps/my_map.pgm

# Or use ImageMagick
convert my_map.pgm -threshold 50% my_map_binary.pgm
```

**Tips:**
- Paint white to add free space
- Paint black to add obstacles
- Keep resolution consistent
- Save as PGM format

## Troubleshooting

### No Map Appearing

**Check:**
```bash
# Is SLAM running?
ros2 node list | grep slam

# Is map being published?
ros2 topic hz /map

# Is lidar working?
ros2 topic hz /scan
```

**Solution:**
- Restart SLAM node
- Check scan topic is `/scan`
- Verify `use_sim_time` is `true`

### Map Not Updating

**Symptoms**: Robot moves but map stays same

**Causes:**
1. Not moving enough (below thresholds)
2. Lidar not publishing
3. Transform issues

**Solutions:**
```bash
# Check transforms
ros2 run tf2_ros tf2_echo odom base_link

# Reduce movement thresholds
ros2 param set /slam_toolbox minimum_travel_distance 0.05
```

### Poor Map Quality

**Symptoms**: Blurry, disconnected, or drifting

**Solutions:**

1. **Slow down**:
```bash
# In teleop, use slower speeds
# Press 'z' to decrease speed
```

2. **Better scan matching**:
```python
# In slam.launch.py
'minimum_travel_distance': 0.05,  # More frequent
'scan_buffer_size': 20,           # Keep more scans
```

3. **Remap the area**:
- Start fresh in problem area
- Use better driving pattern
- Ensure good lighting (if using camera)

### SLAM Node Crashes

**Check logs:**
```bash
ros2 node list
# If slam_toolbox missing, check terminal output
```

**Common causes:**
- Out of memory (reduce `scan_buffer_size`)
- Corrupted parameters (reset to defaults)
- TF timeout (check `transform_timeout`)

## Tips for Production Mapping

1. **Survey first**: Walk the area before mapping
2. **Plan route**: Decide coverage pattern
3. **Mark start**: Physical marker for loop closure
4. **Monitor live**: Watch RViz during mapping
5. **Multiple attempts**: Keep best map from several runs
6. **Edit after**: Clean up artifacts in image editor
7. **Verify**: Test navigation on new map

## Next Steps

After creating maps:
- Use for [Autonomous Navigation](navigation.md)
- Set up [Localization](localization.md) with AMCL
- [Tune parameters](tuning.md) for better performance

---

[← Back to Main Documentation](../README.md) | [Next: Autonomous Navigation →](navigation.md)