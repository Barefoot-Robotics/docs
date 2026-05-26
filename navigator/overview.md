# Navigator Robot - Overview

![Screenshot](/images/overview.png)

## Introduction

The Navigator is a cutting-edge autonomous mobile robot platform designed for research and development in robotics, particularly focused on SLAM (Simultaneous Localization and Mapping) and autonomous navigation. Built on ROS 2 Humble, it provides a robust, extensible foundation for developing and testing navigation algorithms in both simulated and real-world environments.

## Key Features

### Hardware Capabilities
- **Differential Drive System**: Two-wheeled design with precise motor control
- **360° Lidar Sensor**: Real-time environmental scanning with 10m range
- **RGB Camera**: 640×480 resolution for visual perception
- **IMU Sensor**: Inertial measurement for enhanced odometry
- **Compact Design**: 180mm × 120mm × 60mm footprint

### Software Stack
- **ROS 2 Humble**: Latest long-term support ROS distribution
- **Nav2 Integration**: Full autonomous navigation stack
- **SLAM Toolbox**: Real-time mapping and localization
- **ros2_control**: Standardized hardware interface
- **Gazebo Fortress**: High-fidelity physics simulation

### Autonomous Capabilities
- **SLAM Mapping**: Create accurate maps of unknown environments
- **Path Planning**: Intelligent global and local path generation
- **Obstacle Avoidance**: Dynamic obstacle detection and avoidance
- **Localization**: Precise pose estimation using AMCL
- **Waypoint Navigation**: Follow predefined paths autonomously


## Use Cases

### Research & Development
- Algorithm development and testing
- Multi-robot coordination experiments
- Human-robot interaction studies
- Machine learning dataset collection

### Education
- Robotics curriculum development
- Student project platform
- ROS 2 learning and training
- Competition preparation

### Industrial Applications
- Warehouse automation research
- Indoor navigation systems
- Autonomous delivery prototypes
- Facility inspection robots

## Technical Specifications

![drawing](</images/drawing.png>)

### Physical Dimensions
- **Length**: 180 mm
- **Width**: 120 mm
- **Height**: 60 mm (base) + 40 mm (lidar)
- **Weight**: ~5 kg (fully assembled)
- **Wheel Diameter**: 72 mm
- **Wheel Base**: 195 mm

### Performance Specifications
- **Max Speed**: 1.0 m/s (configurable)
- **Max Angular Velocity**: 2.0 rad/s
- **Battery Life**: 2-4 hours (typical operation)
- **Payload Capacity**: 2 kg
- **Operating Temperature**: 0°C to 40°C

## Software Packages

The Navigator platform consists of six main ROS 2 packages:

1. **navigator_bringup**: System startup and launch files
2. **navigator_control**: Motor control and ros2_control configuration
3. **navigator_description**: URDF models and robot description
4. **navigator_msgs**: Custom message definitions
5. **navigator_nav**: Navigation and SLAM configuration
6. **navigator_sim**: Gazebo simulation environment


## Getting Started

### Quick Links
- [Installation Guide](installation.md) - Set up your development environment
- [Quick Start](quickstart.md) - Get running in 5 minutes
- [Simulation Guide](simulation.md) - Learn Gazebo simulation
- [SLAM Tutorial](slam.md) - Create your first map
- [Navigation Guide](navigation.md) - Autonomous navigation setup

### Community & Support
- **GitHub Repository**: [github.com/barefootrobotics/navigator](https://github.com/barefootrobotics/navigator)
- **Documentation**: [docs.barefootrobotics.com](https://docs.barefootrobotics.com)
- **Email Support**: info@barefootrobotics.com
- **Discord Community**: Coming soon!

## Design Philosophy

### Open Source
Navigator is fully open source, allowing you to:
- Modify and extend functionality
- Contribute improvements
- Learn from the codebase
- Share with the community

### Modularity
Each component is designed to be:
- Independently replaceable
- Well-documented
- Following ROS 2 standards
- Easily customizable

### Extensibility
The platform supports:
- Custom sensor integration
- Additional hardware modules
- Third-party algorithms
- Multi-robot systems

## Performance Benchmarks

### Navigation Accuracy
- **Position Error**: < 5 cm (in mapped environments)
- **Orientation Error**: < 3 degrees
- **Goal Achievement Rate**: > 95%
- **Path Efficiency**: > 90% (vs optimal path)

### Mapping Quality
- **Map Resolution**: 5 cm per cell
- **Loop Closure Success**: > 90%
- **Map Consistency**: High (< 2% drift)
- **Processing Time**: Real-time (< 100ms per scan)

### System Performance
- **CPU Usage**: 30-40% (Intel i5 equivalent)
- **Memory Usage**: < 2 GB RAM
- **Network Latency**: < 10ms (local)
- **Response Time**: < 50ms (emergency stop)

## Acknowledgments

The Navigator platform is built upon the incredible work of the open-source robotics community, including:
- ROS 2 Development Team
- Nav2 Contributors
- SLAM Toolbox Developers
- Gazebo Simulation Team

## License

Navigator is released under the Apache 2.0 License, promoting open collaboration and innovation in robotics research and development.

---

**Ready to get started?** Check out our [Installation Guide](installation.md) or jump straight to the [Quick Start](quickstart.md)!

**Have questions?** Visit our [FAQ](faq.md) or [Troubleshooting Guide](troubleshooting.md).