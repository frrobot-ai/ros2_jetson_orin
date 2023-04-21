# ROS2 Docker Image

## Building

To build a docker image of ROS2_foxy_ur with Gazebo simulation on Jetson AGX Orin.
 
1. Run the following command directly in this directory:
    ```
    ./BUILD-DOCKER-IMAGE.sh
    ```
2. 2000 years later, run ``` docker images ``` to ensure the build success.

    If it shows a IMAGE called *ros2-jetson-jp5.0.2:foxy_ur_gazebo*, congratulations! Move on to the next step.

3. Run this command to start a container:

    ```
    ./RUN-DOCKER.sh
    ```
    Use ``` docker ps ``` to check the container's information, its name should be *$(YOUR_USER_NAME)_ros2_foxy_ur_gazebo_jetson_1*.

> **NOTE**:
> The container would be stopped if the PC is rebooted, but it would still exist. 
> Just run ```./RUN-DOCKER.sh``` again to start it.

## Running Simulation

### **Attention: Gazebo Segmentation fault (core dumped) PROBLEM occurs on Jetson** (unsolved)

```bash
ros2 launch ur_simulation_gazebo ur_sim_control.launch.py
```

Move robot using test script from  `ur_bringup` package:
```bash
ros2 launch ur_bringup test_joint_trajectory_controller.launch.py
```
