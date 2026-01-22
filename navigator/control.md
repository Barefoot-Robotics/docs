# Robot Control

Understanding the Navigator's control system using ros2_control and differential drive.

## Overview

The Navigator uses:
- **ros2_control** - Standardized hardware interface
- **Differential Drive Controller** - Two-wheeled robot control
- **Joint State Broadcaster** - Publishes wheel positions

## Control Architecture

```
cmd_vel (Twist) → Differential Drive Controller → Motor Commands → Wheels
                            ↓
                    Joint State Broadcaster → joint_states → Odometry
```

## ros2_control Configuration

Located in `navigator_control/config/ros2_control.yaml`:

```yaml
controller_manager:
  ros__parameters:
    update_rate: 50  # Hz

    joint_state_broadcaster:
      type: joint_state_broadcaster/JointStateBroadcaster

    diff_drive_controller:
      type: diff_drive_controller/DiffDriveController
```

## Differential Drive Controller

### Parameters

**Wheel Configuration:**
```yaml
diff_drive_controller:
  ros__parameters:
    left_wheel_names: ["left_wheel_joint"]
    right_wheel_names: ["right_wheel_joint"]
    
    wheel_radius: 0.033       # meters
    wheel_separation: 0.145   # meters
```

**Update Rates:**
```yaml
publish_rate: 50.0           # Hz
cmd_vel_timeout: 0.5         # seconds
```

**Frame IDs:**
```yaml
odom_frame_id: "odom"
base_frame_id: "base_link"
enable_odom_tf: true
```

**Covariance:**
```yaml
# Position uncertainty
pose_covariance_diagonal: [0.001, 0.001, 0.001, 0.001, 0.001, 0.01]

# Velocity uncertainty
twist_covariance_diagonal: [0.001, 0.001, 0.001, 0.001, 0.001, 0.01]
```

## Control Topics

### Subscribe (Input)

**`/cmd_vel` (geometry_msgs/Twist)**
```bash
# Move forward at 0.3 m/s
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist \
  "{linear: {x: 0.3}, angular: {z: 0.0}}"

# Rotate in place
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist \
  "{linear: {x: 0.0}, angular: {z: 0.5}}"

# Stop
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist \
  "{linear: {x: 0.0}, angular: {z: 0.0}}"
```

### Publish (Output)

**`/odom` (nav_msgs/Odometry)**
- Robot position and velocity
- Updated at 50 Hz

**`/joint_states` (sensor_msgs/JointState)**
- Wheel positions and velocities
- Updated at 50 Hz

**`/tf` (tf2_msgs/TFMessage)**
- Transform: odom → base_link

## Velocity Limits

### Linear Velocity
```yaml
max_linear_velocity: 1.0      # m/s
min_linear_velocity: -1.0     # m/s
```

### Angular Velocity
```yaml
max_angular_velocity: 2.0     # rad/s
min_angular_velocity: -2.0    # rad/s
```

### Acceleration
```yaml
max_linear_acceleration: 2.0   # m/s²
max_angular_acceleration: 3.0  # rad/s²
```

## Kinematics

### Forward Kinematics

Given wheel velocities, compute robot velocity:

```python
v = (v_left + v_right) / 2              # Linear velocity
ω = (v_right - v_left) / wheel_base     # Angular velocity
```

### Inverse Kinematics

Given robot velocity, compute wheel velocities:

```python
v_left = v - (ω × wheel_base / 2)
v_right = v + (ω × wheel_base / 2)
```

### Python Example

```python
import math

class DiffDrive:
    def __init__(self, wheel_radius=0.033, wheel_base=0.145):
        self.R = wheel_radius
        self.L = wheel_base
    
    def forward_kinematics(self, v_left, v_right):
        """Wheel velocities to robot velocity"""
        v = (v_left + v_right) / 2
        omega = (v_right - v_left) / self.L
        return v, omega
    
    def inverse_kinematics(self, v, omega):
        """Robot velocity to wheel velocities"""
        v_left = v - (omega * self.L / 2)
        v_right = v + (omega * self.L / 2)
        return v_left, v_right

# Usage
robot = DiffDrive()
v_left, v_right = robot.inverse_kinematics(v=0.3, omega=0.5)
print(f"Left: {v_left:.2f} m/s, Right: {v_right:.2f} m/s")
```

## Odometry

### Dead Reckoning

The controller estimates position using wheel encoders:

```python
# Update position
dt = 0.02  # 50 Hz update rate

# Compute velocity
v, omega = forward_kinematics(v_left, v_right)

# Update pose
if abs(omega) < 0.001:  # Straight line
    dx = v * cos(theta) * dt
    dy = v * sin(theta) * dt
    dtheta = 0
else:  # Arc motion
    radius = v / omega
    dtheta = omega * dt
    dx = radius * (sin(theta + dtheta) - sin(theta))
    dy = -radius * (cos(theta + dtheta) - cos(theta))

# Update state
x += dx
y += dy
theta += dtheta
```

### Drift Correction

Odometry drifts over time. Solutions:
1. **Better calibration** - Measure wheel parameters accurately
2. **Sensor fusion** - Combine with IMU data
3. **Localization** - Use AMCL with map

## Controller Management

### List Controllers

```bash
ros2 control list_controllers
```

Output:
```
joint_state_broadcaster[joint_state_broadcaster/JointStateBroadcaster] active
diff_drive_controller[diff_drive_controller/DiffDriveController] active
```

### Controller States

```bash
# Get state
ros2 control list_hardware_interfaces

# Set state
ros2 control set_controller_state diff_drive_controller start
ros2 control set_controller_state diff_drive_controller stop
```

### Load/Unload Controllers

```bash
# Load
ros2 control load_controller diff_drive_controller

# Unload
ros2 control unload_controller diff_drive_controller
```

## Hardware Interface

### Mock Hardware (Simulation)

Uses fake hardware for testing without physical robot:

```xml
<hardware>
  <plugin>mock_components/GenericSystem</plugin>
  <param name="use_fake_hardware">true</param>
  <param name="fake_sensor_commands">false</param>
</hardware>
```

### Real Hardware

Replace with actual hardware interface:

```xml
<hardware>
  <plugin>navigator_hardware/NavigatorHardware</plugin>
  <param name="serial_port">/dev/ttyUSB0</param>
  <param name="baud_rate">115200</param>
</hardware>
```

## Calibration

### Measure Wheel Parameters

**Wheel Radius:**
1. Mark wheel
2. Roll one complete rotation
3. Measure distance traveled
4. radius = distance / (2 × π)

**Wheel Separation:**
1. Measure distance between wheel centers
2. Account for tire width

### Test Odometry

```bash
# Drive in a square and return to start
# Measure drift to evaluate calibration
```

## Safety Features

### Velocity Timeout

If no cmd_vel received for 0.5 seconds:
- Robot stops automatically
- Prevents runaway

```yaml
cmd_vel_timeout: 0.5  # seconds
```

### Velocity Limits

Hardware enforces maximum velocities:
- Protects motors
- Prevents dangerous speeds

### Emergency Stop

```bash
# Publish zero velocity
ros2 topic pub /cmd_vel geometry_msgs/msg/Twist \
  "{linear: {x: 0.0}, angular: {z: 0.0}}" --once
```

## Tuning

### Improve Straight Line Motion

If robot curves when commanded straight:

1. **Check wheel radius** - Ensure both wheels same size
2. **Calibrate wheel_separation**
3. **Check motor balance**

### Reduce Wheel Slip

```yaml
# Lower max acceleration
max_linear_acceleration: 1.0  # From 2.0
```

### Smooth Motion

```yaml
# Velocity smoothing (if available)
velocity_smoother:
  smoothing_frequency: 20.0
  scale_velocities: true
```

## Integration with Navigation

The controller works seamlessly with Nav2:

```
Nav2 Controller → /cmd_vel → Diff Drive Controller → Wheels
                                      ↓
                                   /odom → Nav2 Localization
```

## Common Issues

### Robot Doesn't Move

**Check:**
```bash
# Is controller active?
ros2 control list_controllers

# Is cmd_vel being received?
ros2 topic echo /cmd_vel
```

### Odometry Drift

**Solutions:**
1. Calibrate wheel parameters
2. Check for wheel slip
3. Use localization (AMCL)

### Jerky Motion

**Solutions:**
```yaml
# Increase update rate
update_rate: 100  # From 50

# Add velocity smoothing
```

## Advanced Topics

### Custom Hardware Interface

Create your own hardware interface for custom motors:

```cpp
// my_hardware.hpp
class MyHardware : public hardware_interface::SystemInterface {
  // Implement read() and write() methods
};
```

### State Estimation

Combine odometry with IMU using `robot_localization`:

```bash
sudo apt install ros-humble-robot-localization
```

## Next Steps

- Learn about [Manual Control](manual-control.md) methods
- Explore [Navigation](navigation.md) integration
- See [Parameter Tuning](tuning.md) for optimization

---

[← Back to Main Documentation](../README.md)