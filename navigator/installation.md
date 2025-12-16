# Installation Guide

This guide will help you set up your development environment for the Navigator robot.

## Prerequisites

- **Operating System**: Ubuntu 22.04 (recommended)
- **ROS 2 Distribution**: Humble Hawksbill
- **Disk Space**: At least 10 GB free

## Step 1: Install ROS 2 Humble

If you don't have ROS 2 Humble installed:

```bash
# Set locale
sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

# Setup sources
sudo apt install software-properties-common
sudo add-apt-repository universe
sudo apt update && sudo apt install curl -y
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

# Install ROS 2
sudo apt update
sudo apt upgrade
sudo apt install ros-humble-desktop
```

## Step 2: Install Dependencies

Install required ROS 2 packages:

```bash
# Navigation and SLAM
sudo apt install ros-humble-nav2-bringup
sudo apt install ros-humble-slam-toolbox

# Control
sudo apt install ros-humble-ros2-control
sudo apt install ros-humble-ros2-controllers
sudo apt install ros-humble-diff-drive-controller
sudo apt install ros-humble-joint-state-broadcaster

# Gazebo Simulation
sudo apt install ros-humble-ros-gz
sudo apt install ros-humble-gazebo-ros-pkgs

# Additional tools
sudo apt install ros-humble-xacro
sudo apt install ros-humble-robot-state-publisher
sudo apt install ros-humble-joint-state-publisher
sudo apt install python3-colcon-common-extensions
```

## Step 3: Create Workspace

```bash
# Create workspace
mkdir -p ~/navigator_ws/src
cd ~/navigator_ws/src

# Clone the Navigator repository
git clone https://github.com/yourusername/navigator.git

# Return to workspace root
cd ~/navigator_ws
```

## Step 4: Build the Workspace

```bash
# Source ROS 2
source /opt/ros/humble/setup.bash

# Build all packages
colcon build --symlink-install

# Source the workspace
source install/setup.bash
```

## Step 5: Verify Installation

Test that everything is installed correctly:

```bash
# Check if packages are found
ros2 pkg list | grep navigator

# You should see:
# navigator_bringup
# navigator_control
# navigator_description
# navigator_msgs
# navigator_nav
# navigator_sim
```

## Step 6: Setup Environment (Optional but Recommended)

Add these lines to your `~/.bashrc` for automatic sourcing:

```bash
# Open bashrc
nano ~/.bashrc

# Add at the end:
source /opt/ros/humble/setup.bash
source ~/navigator_ws/install/setup.bash
export GAZEBO_MODEL_PATH=$GAZEBO_MODEL_PATH:~/navigator_ws/src/navigator/navigator_sim/models

# Save and reload
source ~/.bashrc
```

## Troubleshooting

### Missing Dependencies

If you encounter missing dependencies during build:

```bash
rosdep install --from-paths src --ignore-src -r -y
```

### Gazebo Issues

If Gazebo doesn't start:

```bash
# Install Gazebo Fortress
sudo apt install ignition-fortress
```

### Build Errors

Clean and rebuild:

```bash
cd ~/navigator_ws
rm -rf build install log
colcon build --symlink-install
```

## Next Steps

- Continue to [Quick Start](quickstart.md) to run your first simulation
- Or check [Package Overview](packages.md) to understand the repository structure

---

[← Back to Main Documentation](../README.md)