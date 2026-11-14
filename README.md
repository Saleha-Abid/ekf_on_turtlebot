# TurtleBot3 EKF Sensor Fusion & Trajectory Visualization (ROS 2 + Gazebo)

A ROS 2 simulation package that fuses TurtleBot3 wheel odometry (`/odom`) and IMU measurements (`/imu`) using an Extended Kalman Filter (`robot_localization`) in Gazebo, and publishes continuous `nav_msgs/msg/Path` trajectories for live comparison in RViz2.

---

## Features
- **2D Extended Kalman Filter (`robot_localization`):** Fuses differential-drive linear and angular velocities from `/odom` with yaw orientation and yaw rate from `/imu` to publish `/odometry/filtered` and broadcast `odom -> base_footprint`.
- **Single-Source TF Ownership:** Custom Gazebo model configuration disables direct `DiffDrive` TF broadcasts to prevent transform conflicts with `ekf_filter_node`.
- **Noisy Sensor Simulation:** Injects realistic Gaussian noise into the TurtleBot3 IMU to demonstrate filter smoothing vs. raw odometry drift.
- **Live Path Visualization (`odom_to_path.py`):** Converts `/odom` and `/odometry/filtered` into continuous `nav_msgs/msg/Path` topics (`/odom_path` and `/ekf_path`) for side-by-side trajectory comparison in RViz2.

---

## Prerequisites
- **OS:** Ubuntu 26.04 LTS (or Ubuntu 24.04 LTS)
- **ROS 2 Distribution:** ROS 2 Lyrical Luth (or Jazzy Jalisco)
- **Simulator:** Gazebo Jetty (or Harmonic)

Install the required ROS 2 packages:

    sudo apt update && sudo apt install -y \
      ros-$ROS_DISTRO-ros-gz \
      ros-$ROS_DISTRO-turtlebot3* \
      ros-$ROS_DISTRO-robot-localization \
      ros-$ROS_DISTRO-teleop-twist-keyboard \
      ros-$ROS_DISTRO-rviz2

---

## Repository Structure

    .
    ├── config/
    │   └── ekf.yaml               # robot_localization EKF parameters & sensor masks
    ├── launch/
    │   └── full_sim.launch.py     # Launches Gazebo, bridges, EKF, and path publisher
    ├── models/
    │   └── turtlebot3_waffle/     # Modified TurtleBot3 Waffle SDF (TF disabled + IMU noise)
    ├── scripts/
    │   └── odom_to_path.py        # Converts /odom and /odometry/filtered to nav_msgs/Path
    ├── CMakeLists.txt
    └── package.xml

---

## Build Instructions

1. Clone this repository into your ROS 2 workspace `src/` folder:

       mkdir -p ~/turtlebot3_ekf_ws/src
       cd ~/turtlebot3_ekf_ws/src
       git clone https://github.com/<your-username>/<your-repo-name>.git

2. Build the workspace with `colcon`:

       cd ~/turtlebot3_ekf_ws
       colcon build --symlink-install
       source install/setup.bash

---

## Usage

### 1. Launch Simulation, EKF, and Path Node

    source ~/turtlebot3_ekf_ws/install/setup.bash
    export TURTLEBOT3_MODEL=waffle
    ros2 launch tb3_ekf_apriltag full_sim.launch.py

*(If you renamed the package in `package.xml` and `CMakeLists.txt`, replace `tb3_ekf_apriltag` with your updated package name.)*

### 2. Drive the Robot via Teleop
In a second terminal, launch `teleop_twist_keyboard` with stamped velocity messages enabled (required by the Gazebo bridge):

    ros2 run teleop_twist_keyboard teleop_twist_keyboard --ros-args -p stamped:=true

### 3. Visualize Trajectories in RViz2
In a third terminal, open RViz2:

    rviz2

1. Set **Global Options -> Fixed Frame** to `odom`.
2. Click **Add -> Path**, set **Topic** to `/odom_path`, and set **Color** to **Red** (`255; 0; 0`).
3. Click **Add -> Path**, set **Topic** to `/ekf_path`, and set **Color** to **Green** (`0; 255; 0`).
4. Drive loops in Gazebo to compare raw odometry against the EKF-filtered trajectory.

---

## Sensor Fusion Configuration (`config/ekf.yaml`)

| Input Topic | Fused States | Purpose |
| :--- | :--- | :--- |
| **`/odom`** | Linear velocity ($v_x, v_y$), yaw rate ($\dot{\psi}$) | Provides translational and rotational velocity estimates from wheel encoders |
| **`/imu`** | Yaw orientation ($\psi$), yaw rate ($\dot{\psi}$) | Stabilizes heading orientation and angular rate against wheel slip |
