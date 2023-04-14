# https://github.com/dusty-nv/jetson-containers
FROM dustynv/ros:humble-pytorch-l4t-r35.1.0

#This symbolic link is needed to use the streaming features on Jetson inside a container
RUN ln -sf /usr/lib/aarch64-linux-gnu/tegra/libv4l2.so.0 /usr/lib/aarch64-linux-gnu/libv4l2.so

#Setup ROS Workspace
ENV ROS_WS ~/ros2_ws

RUN mkdir -p ${ROS_WS}/src && \
    cd ${ROS_WS}/ && \
    apt-get update -y && apt-get install --no-install-recommends python3-rosdep python3-colcon-common-extensions -y

RUN cd ${ROS_WS}/src && \
    git clone -b ${ROS_DISTRO}_ur https://github.com/frrobot-ai/ros2_jetson_orin.git glozzom

RUN git clone https://github.com/ros/rosdistro.git /etc/ros/ros_github/rosdistro/
COPY ros_bypass/rep3.py /usr/lib/python3/dist-packages/rosdep2/
COPY ros_bypass/__init__.py /usr/lib/python3/dist-packages/rosdistro/
COPY ros_bypass/20-default.list /etc/ros/rosdep/sources.list.d/

# Install ur_driver 
RUN apt-get update && \
    apt-get install ros-${ROS_DISTRO}-ur-robot-driver -y

# RUN rosdep init
RUN rosdep update
RUN cd ${ROS_WS}/ && \
    rosdep install --from-paths src --ignore-src -y -r

RUN cd ${ROS_WS}/ && \
    /bin/bash -c 'source /opt/ros/${ROS_DISTRO}/setup.bash' && \
    colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release && \
    . install/local_setup.sh     

RUN echo 'source /opt/ros/${ROS_DISTRO}/setup.bash' >> /root/.bashrc && \
    echo 'source ${ROS_WS}/install/local_setup.bash' >> /root/.bashrc
    
RUN apt-get autoremove

WORKDIR ${ROS_WS}
