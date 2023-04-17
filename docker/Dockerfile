ARG L4T_MINOR_VERSION=1.0

# https://catalog.ngc.nvidia.com/orgs/nvidia/containers/l4t-jetpack
FROM nvcr.io/nvidia/l4t-jetpack:r35.1.0

#This environment variable is needed to use the streaming features on Jetson inside a container
ENV LOGNAME root
ENV DEBIAN_FRONTEND noninteractive
ENV ROS_DISTRO foxy

#Install dependencies
RUN apt-get update -y && apt-get install -y sudo apt-utils apt-transport-https lsb-release udev usbutils git 

RUN apt-get update -y && apt-get install -y --no-install-recommends wget less cmake curl gnupg2 \
    build-essential python3 python3-pip python3-dev python3-setuptools libusb-1.0-0-dev -y

#This symbolic link is needed to use the streaming features on Jetson inside a container
RUN ln -sf /usr/lib/aarch64-linux-gnu/tegra/libv4l2.so.0 /usr/lib/aarch64-linux-gnu/libv4l2.so

#ROS system setup 
ENV LANG en_US.UTF-8
ENV LC_ALL en_US.UTF-8

RUN apt-get update -y && apt-get install --no-install-recommends -y software-properties-common && \
    add-apt-repository universe && \
    curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key  -o /usr/share/keyrings/ros-archive-keyring.gpg && \
    /bin/bash -c 'echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(source /etc/os-release && echo $UBUNTU_CODENAME)  main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null' && \
    rm -rf /var/lib/apt/lists/* && \
    apt-get clean

RUN apt-get update -y && apt-get install -y ros-$ROS_DISTRO-ros-base && \
    /bin/bash -c 'source /opt/ros/$ROS_DISTRO/setup.bash' && \
    rm -rf /var/lib/apt/lists/*

#Setup ROS Workspace
ENV ROS_WS /opt/ros2_ws

RUN mkdir -p ${ROS_WS}/src && \
    cd ${ROS_WS}/ && \
    apt-get update -y && apt-get install --no-install-recommends python3-rosdep python3-colcon-common-extensions -y

COPY ros_bypass/rosdistro/ /etc/ros/ros_github/rosdistro/
COPY ros_bypass/rep3.py /usr/lib/python3/dist-packages/rosdep2/
COPY ros_bypass/__init__.py /usr/lib/python3/dist-packages/rosdistro/
COPY ros_bypass/20-default.list /etc/ros/rosdep/sources.list.d/

# RUN rosdep init
RUN rosdep update
RUN cd ${ROS_WS}/ && \
    rosdep install --from-paths src --ignore-src -y && \
    colcon build --symlink-install && \
    . install/local_setup.sh     

RUN echo 'source /opt/ros/$ROS_DISTRO/setup.bash' >> /root/.bashrc && \
    echo 'source ${ROS_WS}/install/local_setup.bash' >> /root/.bashrc
    
RUN apt-get autoremove

# Install turtlesim package
RUN apt update && \
    apt install ros-$ROS_DISTRO-turtlesim -y
# Install rqt tools    
RUN apt update && \
    apt install ~nros-$ROS_DISTRO-rqt* -y
# Install rviz2
RUN apt update && \
    sudo apt install ros-$ROS_DISTRO-rviz2 -y
# Install gazebo11
RUN curl -sSL http://get.gazebosim.org | sh && \
    sudo apt install ros-$ROS_DISTRO-gazebo-ros-pkgs -y
    
## UR workspace
# Create the workspace
RUN /bin/bash -c "source /opt/ros/$ROS_DISTRO/setup.bash && \
    mkdir -p ~/ros_ws/src"
# Download source
RUN /bin/bash -c "cd ~/ros_ws/src && \
    git clone -b foxy_ur https://github.com/frrobot-ai/ros2_jetson_orin.git glozzom"

RUN apt update && \
    apt install vim git make python3-pip -y
RUN python3 -m pip install vcstool

RUN /bin/bash -c "cd ~/ros_ws && \
    vcs import src/glozzom --skip-existing --input src/glozzom/Universal_Robots_ROS2_Driver/Universal_Robots_ROS2_Driver.repos"
# Install dependencies
RUN /bin/bash -c "cd ~/ros_ws && \ 
    rosdep update && \
    rosdep install --ignore-src --from-paths src -y -r"
# Build    
RUN /bin/bash -c "cd ~/ros_ws && \
    source /opt/ros/$ROS_DISTRO/setup.bash && \
    colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release && \
    source install/setup.bash"  

RUN echo "source ~/ros_ws/install/setup.bash" >> ~/.bashrc