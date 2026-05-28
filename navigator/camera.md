# Camera Integration — UC-261 (OV5647)

<img src="/images/camera.jpg" width="400" alt="camera">

The Navigator-I uses a **UC-261 USB camera** with an **OV5647 sensor** connected
via the Raspberry Pi CSI interface, accessed through **libcamera** and the
**camera_ros** ROS 2 bridge.

---

## Hardware Details

| Property | Value |
|----------|-------|
| Camera module | UC-261 |
| Sensor | OV5647 |
| Interface | CSI (Raspberry Pi camera port) |
| Max resolution | 2592×1944 |
| Working resolution | 640×480 |
| Format | YUYV |
| Network fps | ~17 fps (compressed) |
| libcamera device | `/base/soc/i2c0mux/i2c@1/ov5647@36` |

---

## Topics Published

| Topic | Type | Notes |
|-------|------|-------|
| `/camera/image_raw` | `sensor_msgs/Image` | Raw frames — heavy, avoid over network |
| `/camera/camera_info` | `sensor_msgs/CameraInfo` | Calibration info |
| `/camera/image_raw/compressed` | `sensor_msgs/CompressedImage` | Use this in RViz |

---

## Launch

Camera starts automatically as part of hardware bringup:

```bash
ros2 launch navigator_bringup hardware.launch.py
```

To run camera only:

```bash
ros2 run camera_ros camera_node --ros-args \
    -p camera:="/base/soc/i2c0mux/i2c@1/ov5647@36" \
    -p width:=640 \
    -p height:=480 \
    -p format:=YUYV
```

---

## View in RViz

In RViz on your computer:

1. Add → By topic → `/camera/image_raw/compressed` → Image → OK
2. In the Image display properties:
   - **Transport Hint**: `compressed`
3. You should see ~17fps live feed

**Do not use raw `/camera/image_raw` over the network** — it causes severe lag
(1fps) due to uncompressed YUYV bandwidth.

---

## ROS_DOMAIN_ID

Both machines must have the same domain ID:

```bash
# Add to ~/.bashrc on Pi AND your computer
export ROS_DOMAIN_ID=0
export ROS_LOCALHOST_ONLY=0
source ~/.bashrc
```

---

## Calibration (Optional)

Only needed for accurate CV / object detection work.
Not required for SLAM, Nav2, or basic RViz viewing.

### Install calibration tool

```bash
sudo apt install ros-humble-camera-calibration
```

### Run calibration

```bash
ros2 launch navigator_bringup calibrate_camera.launch.py
```

Use an 8×6 checkerboard with 25mm squares. After calibration:

```bash
cd /tmp && tar -xzf calibrationdata.tar.gz
mkdir -p ~/.ros/camera_info
cp /tmp/ost.yaml \
   ~/.ros/camera_info/ov5647__base_soc_i2c0mux_i2c_1_ov5647_36_640x480.yaml
```

### Calibration file location

```
~/.ros/camera_info/ov5647__base_soc_i2c0mux_i2c_1_ov5647_36_640x480.yaml
```

---

## Power Requirements

The OV5647 camera adds ~250mA to the Pi's power draw. With lidar and motors
the total system draws ~3A. Use a **5V/3A USB-C supply** minimum.

Check for undervoltage:

```bash
vcgencmd get_throttled
# 0x0 = fine
# 0x50000 = undervoltage — upgrade your power supply
```

---

## Troubleshooting

### Camera not detected

```bash
# Check libcamera sees it
ros2 run camera_ros camera_node 2>&1 | grep cameras
# Should show: 0: ov5647 (/base/soc/i2c0mux/i2c@1/ov5647@36)
```

### Topics not visible on other computer

```bash
# Verify ROS_DOMAIN_ID matches on both machines
echo $ROS_DOMAIN_ID
echo $ROS_LOCALHOST_ONLY  # must be 0 or unset
```

### Image lagging badly

Switch RViz to compressed transport:
- Topic: `/camera/image_raw/compressed`
- Transport Hint: `compressed`

### Camera node crashes silently

Another process is holding the camera device:

```bash
sudo pkill -f camera_node
sudo pkill -f libcamera
# Then restart
ros2 launch navigator_bringup hardware.launch.py
```

---

[← Back to Main Documentation](../README.md)