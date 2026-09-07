# Lastmile Robotics Assignment 04
## Real-World Indoor Navigation & State Monitoring

### Project Overview

This project implements a ROS 2 based indoor mobile robot simulation using a real-world 3D environment scan.

The main objective is to process the provided 3D point cloud, integrate the environment into Gazebo, place a mobile robot inside the environment, generate a 2D navigation map, and develop a custom ROS 2 node called `robot_state_manager` for robot state monitoring and waypoint handling.

## Objectives

- Process the provided real-world 3D point cloud.
- Convert the environment into a simulation-compatible 3D mesh.
- Load the environment into Gazebo as a static and collidable object.
- Maintain real-world scale.
- Place the robot at `(0, 0, 0)`.
- Generate a 2D occupancy map for navigation.
- Configure localization and Nav2.
- Develop a custom ROS 2 node for telemetry and waypoint commands.

## Task 1: Environment Integration

The provided environment was supplied as a 3D point cloud. The point cloud did not directly contain polygon faces, so it was processed to create an approximate 3D surface mesh.

The resulting mesh was exported as:

```text
environment.obj
```

The mesh was integrated into Gazebo as a static and collidable environment.

### Work Done

- Processed the supplied 3D point cloud.
- Generated a 3D mesh.
- Created the Gazebo environment model.
- Added the environment to the Gazebo world.
- Maintained the required real-world scale.
- Added the mobile robot at `(0, 0, 0)`.

### Robot

A differential-drive robot was integrated into the simulation with:

- Base
- Left wheel
- Right wheel
- Caster
- LiDAR sensor
- Differential-drive controller
- Odometry

The robot receives velocity commands through `/cmd_vel` and publishes odometry through `/odom`.

## Task 2: Map Generation & Localization

The original environment is 3D, while the navigation system uses a 2D representation. Therefore, a 2D occupancy map was generated from the 3D point cloud.

### Map Files

```text
maps/
├── map.pgm
└── map.yaml
```

Map resolution:

```text
0.05 m/pixel
```

### Localization

The localization framework uses the standard transform structure:

```text
map
 |
 v
odom
 |
 v
base_link
```

LiDAR data is available through `/scan` and odometry is available through `/odom`.

Nav2 configuration and localization launch files are included in the package.

## Task 3: Custom ROS 2 Node

A custom Python ROS 2 node named `robot_state_manager` was developed.

### 1. Position Publisher

The node subscribes to:

```text
/odom
```

and extracts the robot position:

```text
x
y
z
```

It publishes the position to:

```text
/robot/current_xyz
```

at 10 Hz.

### 2. Goal Receiver

The node listens for waypoint coordinates on:

```text
/robot/next_waypoint
```

using:

```text
geometry_msgs/Point
```

### 3. Nav2 Interface

When a waypoint is received, `robot_state_manager` sends the target to Nav2 using the `NavigateToPose` action.

Action server:

```text
/navigate_to_pose
```

The goal is represented as a `PoseStamped` in the `map` frame.

### 4. Velocity Monitoring

The node subscribes to:

```text
/cmd_vel
```

and monitors linear and angular velocity.

Example thresholds:

```text
Linear velocity  > 0.8 m/s
Angular velocity > 1.0 rad/s
```

If a value exceeds its threshold, a warning is printed.

## ROS 2 Topics

| Topic | Message Type | Purpose |
|---|---|---|
| `/odom` | `nav_msgs/Odometry` | Robot odometry |
| `/scan` | `sensor_msgs/LaserScan` | LiDAR data |
| `/cmd_vel` | `geometry_msgs/Twist` | Robot velocity command |
| `/robot/current_xyz` | `std_msgs/Float64MultiArray` | Robot `[x,y,z]` position |
| `/robot/next_waypoint` | `geometry_msgs/Point` | Target waypoint |

## ROS 2 Action

```text
/navigate_to_pose
```

Action type:

```text
nav2_msgs/action/NavigateToPose
```

## Package Structure

```text
lastmile_assignment04/
├── config/
│   └── nav2_params.yaml
├── launch/
│   ├── simulation.launch.py
│   ├── localization.launch.py
│   ├── navigation.launch.py
│   └── full_system.launch.py
├── maps/
│   ├── map.pgm
│   └── map.yaml
├── models/
│   ├── environment/
│   │   ├── meshes/
│   │   │   └── environment.obj
│   │   ├── model.config
│   │   └── model.sdf
│   └── lastmile_robot/
│       ├── model.config
│       └── model.sdf
├── urdf/
│   └── robot.urdf
├── worlds/
│   └── lastmile_world.world
├── lastmile_assignment04/
│   ├── __init__.py
│   └── robot_state_manager.py
├── package.xml
├── setup.py
├── setup.cfg
└── README.md
```

## Software Used

- Ubuntu
- ROS 2 Humble
- Gazebo Classic
- RViz2
- Nav2
- AMCL
- Python 3
- CloudCompare
- URDF
- SDF

## Installation and Build

Source ROS 2:

```bash
source /opt/ros/humble/setup.bash
```

Go to the workspace:

```bash
cd ~/lastmile_final_ws
```

Build:

```bash
colcon build --symlink-install
```

Source the workspace:

```bash
source install/setup.bash
```

Verify:

```bash
ros2 pkg list | grep lastmile_assignment04
```

Expected:

```text
lastmile_assignment04
```

## Running the Project

### Start Simulation

```bash
ros2 launch lastmile_assignment04 simulation.launch.py
```

### Start Localization

In another terminal:

```bash
source /opt/ros/humble/setup.bash
source ~/lastmile_final_ws/install/setup.bash
ros2 launch lastmile_assignment04 localization.launch.py
```

### Start Navigation

```bash
source /opt/ros/humble/setup.bash
source ~/lastmile_final_ws/install/setup.bash
ros2 launch lastmile_assignment04 navigation.launch.py
```

### Complete System

```bash
source /opt/ros/humble/setup.bash
source ~/lastmile_final_ws/install/setup.bash
ros2 launch lastmile_assignment04 full_system.launch.py
```

## Useful Verification Commands

Check odometry:

```bash
ros2 topic echo /odom
```

Check LiDAR:

```bash
ros2 topic echo /scan
```

Check robot position:

```bash
ros2 topic echo /robot/current_xyz
```

Check velocity commands:

```bash
ros2 topic echo /cmd_vel
```

Check navigation actions:

```bash
ros2 action list | grep navigate
```

## Sending a Waypoint

Example:

```bash
ros2 topic pub --once /robot/next_waypoint geometry_msgs/msg/Point "{x: 2.0, y: 1.0, z: 0.0}"
```

Expected flow:

```text
/robot/next_waypoint
        |
        v
robot_state_manager
        |
        v
NavigateToPose
        |
        v
Nav2
        |
        v
/cmd_vel
        |
        v
Robot
```

## Verification Performed

### Environment

- 3D point cloud processed.
- 3D mesh generated.
- Environment loaded into Gazebo.
- Environment configured as static and collidable.
- Robot placed at the origin.

### Robot

- Differential-drive robot loaded.
- LiDAR sensor working.
- `/scan` available.
- `/odom` available.
- `/cmd_vel` successfully controls the robot.

### Custom State Manager

- `/odom` subscription implemented and tested.
- `/robot/current_xyz` publishing implemented and tested.
- Position publishing rate set to 10 Hz.
- `/robot/next_waypoint` subscription implemented.
- Nav2 `NavigateToPose` Action Client implemented.
- `/cmd_vel` monitoring implemented.
- Velocity threshold warnings implemented.

## Current Status

### Task 1 — Environment Integration
**Completed**

The 3D environment was processed and integrated into Gazebo with the robot.

### Task 2 — Map Generation & Localization
**Partially completed**

The 2D map, LiDAR integration, Nav2 configuration and localization framework were implemented. However, the complete localization and autonomous navigation pipeline was not consistently demonstrated end-to-end.

### Task 3 — Telemetry & Command Node
**Mostly completed**

The custom `robot_state_manager` was implemented with position publishing, waypoint reception, Nav2 Action Client integration and velocity monitoring. The final autonomous waypoint-to-motion behavior was not consistently demonstrated.

## Limitations

The main limitation is the final end-to-end autonomous Nav2 navigation. During testing, the robot responded correctly to direct `/cmd_vel` commands and its odometry and telemetry changed accordingly. However, Nav2 did not consistently generate `/cmd_vel` for manually selected navigation goals.

Therefore, this implementation is a working prototype rather than a fully production-ready autonomous navigation system.

## Learning Outcomes

Through this assignment, I gained practical experience in:

- 3D point cloud processing.
- 3D environment reconstruction.
- Gazebo simulation.
- ROS 2 package development.
- URDF and SDF.
- Differential-drive robot simulation.
- LiDAR simulation.
- Odometry.
- 2D occupancy-grid maps.
- Localization concepts.
- Nav2.
- ROS 2 publishers and subscribers.
- ROS 2 Action Clients.
- Robot telemetry.
- Velocity monitoring.
- Integration of perception, simulation and navigation components.

## Conclusion

This project demonstrates a ROS 2 indoor robotics workflow starting from a real-world 3D environment scan and progressing toward robot simulation, mapping, localization, navigation and state monitoring.

The main workflow is:

```text
Real-World 3D Point Cloud
          |
          v
Point Cloud Processing
          |
          v
3D Environment Mesh
          |
          v
Gazebo Simulation
          |
          v
Mobile Robot
     /           LiDAR       Odometry
    |             |
    v             v
  /scan          /odom
          |
          v
      2D Map
          |
          v
 Localization / Nav2
          |
          v
 robot_state_manager
       /             v         v
Telemetry    Waypoints
```

This project helped me understand how real-world 3D environment data can be integrated with ROS 2 simulation and how a custom ROS 2 node can connect robot state monitoring with navigation commands.
