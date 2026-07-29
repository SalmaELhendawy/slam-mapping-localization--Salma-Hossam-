## SLAM Mapping & Localization Demo (slam_toolbox) ##
This repository contains a complete ROS 2 package for mapping an environment 
with **SLAM Toolbox** (Async Mapping Mode) and subsequently localizing a **TurtleBot3 Burger** within the generated map 
using serialized Pose Graph localization.

1-Step-by-Step Setup Instructions -Phase 1 (Mapping):
 1. Environment Setup
* Launch the TurtleBot3 simulation:
 from ETGAH platform launch [ turtlebot_world ]

2. Create Workspace & Package Structure
Set up workspace and create the package:
```
mkdir -p ~/workspaces/slam_ws/src
cd ~/workspaces/slam_ws/src
ros2 pkg create --build-type ament_cmake slam_toolbox_demo
cd slam_toolbox_demo
mkdir config launch map
```
3. Configuration & Launch Setup
Create config/slam_toolbox_online_async.yaml configured for asynchronous SLAM (mode: mapping, frames: map ➔ odom ➔ base_footprint, scan topic: /scan).

Create launch/slam_toolbox_online_async.launch.py to run the lifecycle node async_slam_toolbox_node.

Update CMakeLists.txt to install package directories

4. Build & Run Mapping
Build and source the workspace:
cd ~/workspaces/slam_ws
colcon build
source install/setup.bash
Launch SLAM Toolbox:
```
ros2 launch slam_toolbox_demo slam_toolbox_online_async.launch.py
```
5. RViz Visualization & Teleoperation
Open RViz (rviz2), set Fixed Frame to map, and add displays: TF, RobotModel, LaserScan (/scan), and Map (/map).

Drive the robot slowly to build the full map:
using teleop 
```
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```
6. Map Saving
   ```
   cd ~/workspaces/slam_ws/src/slam_toolbox_demo/map
   ros2 run nav2_map_server map_saver_cli -f turtlebot3_world_map
   ```
2/2-Step-by-Step Setup Instructions -Phase 2 (LOCALIZATION):
. Configuration & Launch Setup
* Create `config/slam_toolbox_localization.yaml`:
  * Set `mode: localization`
  * Set initial pose estimate: `map_start_pose: [0.0, 0.0, 0.0]`
* Create `launch/localization.launch.py` to run the lifecycle node `localization_slam_toolbox_node`.
* Ensure `posegraph` is included in `CMakeLists.txt` for installation:
  ```cmake
  install(
    DIRECTORY launch config map posegraph
    DESTINATION share/${PROJECT_NAME}
  )
  ```
3. Build & Run Localization
Rebuild and source the workspace:
Bash
```
cd ~/workspaces/slam_ws
colcon build
source install/setup.bash
```
Launch SLAM Toolbox in localization mode:
```
ros2 launch slam_toolbox_demo localization.launch.py
```
4. RViz Setup & Initial Pose Testing
Open RViz (rviz2), set Fixed Frame to map, and add displays: TF, RobotModel, LaserScan (/scan), Map (/map), and MarkerArray (/slam_toolbox/graph_visualization to view saved robot path).

Test Wrong Pose: Use 2D Pose Estimate to place the robot in an incorrect location. Observe the mismatch between laser scans and wall boundaries.

Test Correct Pose: Set the correct position using 2D Pose Estimate. Confirm that laser scans align perfectly with the map.

5. Verification & Teleoperation
Drive the robot using teleop:
```
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```
Expected Result: The global map remains completely static (does not change or grow), and only the robot pose updates continuously within the pre-built environment.

##  How to Test Your Nodes
1. Testing Mapping Node
Launch the TurtleBot3 simulation in Gazebo.
Launch the mapping node

2.Check active topics and published frames:
```
ros2 topic list
ros2 topic echo /scan --once
ros2 topic echo /odom --once
```
Verify TF transformation tree structure:
```
ros2 run tf2_tools view_frames
```
3. Testing Localization Node
Launch the localization node:
```
ros2 launch slam_toolbox_demo localization.launch.py
```
Verify that the node successfully loads the serialized pose graph from disk without missing file errors.
Test initial pose sensitivity in RViz:
Apply a Wrong 2D Pose Estimate to observe laser scan mismatch.
Apply a Correct 2D Pose Estimate to observe immediate scan-to-map alignment.

3-### Expected Output
**Active TF Tree:** A continuous transform chain: `map` ➔ `odom` ➔ `base_footprint` ➔ `base_link` ➔ `base_scan`.
* **Mapping Phase:** The map dynamically expands and updates in RViz as the robot explores new areas.
* **Wrong 2D Pose:** LiDAR laser lines mismatch and do not line up with the saved map walls.
* **Correct 2D Pose:** LiDAR laser lines overlap perfectly with the saved map walls.
* **Localization Drive:** The global map remains static while only the robot pose updates accurately inside it.
* **Odometry Data (`/odom`):** Continuous coordinate updates with valid positions and orientations without drift.

4-###🎬 Demo

Below are the video demonstrations showing the system running in simulation, terminal execution, and RViz visual updates for both mapping and localization phases:

### 1. SLAM Mapping Phase
*(Video demonstration showing the robot exploring the environment and dynamically generating the map in RViz)*  

https://github.com/user-attachments/assets/4b0cf493-3181-44a5-b5d4-43ede3e2a734

*final map screenshot:
<img width="512" height="314" alt="mapping screnshot" src="https://github.com/user-attachments/assets/ce0c2207-9e6d-4369-bc11-98f829cc075c" />


### 2. Localization Phase: Incorrect Initial Pose Test

https://github.com/user-attachments/assets/72f03d49-0f22-43e4-b7be-3b5f3801064a


### 3. Localization Phase: Correct Initial Pose:
https://github.com/user-attachments/assets/2bd3231d-89dc-4d03-8fa0-fd54aabf35b1


