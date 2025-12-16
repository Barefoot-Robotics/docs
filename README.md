# Navigator Robot Documentation

Welcome to the Navigator Robot documentation. This guide will help you get started with building, simulating, and deploying your Navigator robot.

## 📚 Documentation Sections

### Getting Started
- [Installation Guide](docs/installation.md) - Set up your development environment
- [Quick Start](docs/quickstart.md) - Get your robot running in 5 minutes
- [Package Overview](docs/packages.md) - Understanding the Navigator packages

### Hardware & Description
- [Robot Description](docs/robot-description.md) - URDF models and robot specifications
- [Hardware Setup](docs/hardware-setup.md) - Physical robot assembly and configuration

### Simulation
- [Gazebo Simulation](docs/simulation.md) - Running Navigator in Gazebo
- [Simulation Worlds](docs/worlds.md) - Available environments and creating custom worlds

### Navigation
- [SLAM Mapping](docs/slam.md) - Creating maps with SLAM Toolbox
- [Autonomous Navigation](docs/navigation.md) - Using Nav2 for autonomous navigation
- [Localization](docs/localization.md) - AMCL and localization setup

### Control
- [Robot Control](docs/control.md) - ros2_control configuration and differential drive
- [Manual Control](docs/manual-control.md) - Teleoperation and manual driving

### Advanced Topics
- [Custom Messages](docs/messages.md) - Navigator custom ROS messages
- [Launch Files](docs/launch-files.md) - Understanding and customizing launch files
- [Tuning Parameters](docs/tuning.md) - Optimizing navigation and control parameters

### Troubleshooting
- [Common Issues](docs/troubleshooting.md) - Solutions to common problems
- [FAQ](docs/faq.md) - Frequently asked questions

## 🚀 Quick Links

**First Time Users**: Start with [Installation Guide](docs/installation.md) → [Quick Start](docs/quickstart.md)

**Simulation Users**: Jump to [Gazebo Simulation](docs/simulation.md)

**Hardware Users**: Check [Hardware Setup](docs/hardware-setup.md)

**Navigation Users**: See [SLAM Mapping](docs/slam.md) and [Autonomous Navigation](docs/navigation.md)

## 📦 Repository Structure

```
navigator/
├── navigator_bringup/      # Launch files for bringing up the robot
├── navigator_control/      # ros2_control configuration
├── navigator_description/  # URDF, meshes, robot model
├── navigator_msgs/         # Custom ROS messages
├── navigator_nav/          # Navigation and SLAM configuration
├── navigator_sim/          # Gazebo simulation
└── docs/                   # Documentation files
```

## 🤝 Contributing

Found an issue or want to improve the documentation? Contributions are welcome! Please submit a pull request or open an issue.

## 📄 License

This project is licensed under the MIT License.

## 🔗 Additional Resources

- [ROS 2 Documentation](https://docs.ros.org/)
- [Nav2 Documentation](https://navigation.ros.org/)
- [Gazebo Documentation](https://gazebosim.org/)

---

**Need help?** Check the [Troubleshooting](docs/troubleshooting.md) section or open an issue on GitHub.