# ROS2 Docker Image

To build a docker image of ROS2 on Jetson AGX Orin
 
1. run the following command directly in this directory:
    ```
    ./BUILD-DOCKER-IMAGE.sh
    ```
2. 2000 years later, run ``` docker images ``` to ensure the build success.

    If it shows a IMAGE called *ros2-foxy-jetson-jp5.0.2*, congratulations! Move on to the next step.

3. Run this command to start a container:

    ```
    ./RUN-DOCKER.sh
    ```
    Use ``` docker ps ``` to check the container's information, its name should be *$(YOUR_USER_NAME)_glozzom_1*.
