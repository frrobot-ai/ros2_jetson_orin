ARG L4T_MINOR_VERSION=1.0

# https://catalog.ngc.nvidia.com/orgs/nvidia/containers/l4t-jetpack
FROM nvcr.io/nvidia/l4t-jetpack:r35.1.0

# After using an arg in a `FROM` line, the arg is lo
# ARG L4T_MINOR_VERSION=1.0
# ARG ZED_SDK_MAJOR=3
# ARG ZED_SDK_MINOR=7
# ARG JETPACK_MAJOR=5
# ARG JETPACK_MINOR=0

#This environment variable is needed to use the streaming features on Jetson inside a container
ENV LOGNAME root
ENV DEBIAN_FRONTEND noninteractive

#Install dependencies
RUN apt-get update -y && apt-get install -y sudo apt-utils apt-transport-https lsb-release udev usbutils git 

#Install ZED SDK
RUN apt-get update -y && apt-get install -y --no-install-recommends wget less cmake curl gnupg2 \
    build-essential python3 python3-pip python3-dev python3-setuptools libusb-1.0-0-dev -y && \
    sudo pip install protobuf && \
    echo "# R35 (release), REVISION: ${L4T_MINOR_VERSION}" > /etc/nv_tegra_release && \
    # wget -q --no-check-certificate -O ZED_SDK_Linux_JP.run https://download.stereolabs.com/zedsdk/${ZED_SDK_MAJOR}.${ZED_SDK_MINOR}/jp${JETPACK_MAJOR}${JETPACK_MINOR}/jetsons && \
    # chmod +x ZED_SDK_Linux_JP.run ; ./ZED_SDK_Linux_JP.run silent skip_tools && \
    # rm -rf /usr/local/zed/resources/* && \
    # rm -rf ZED_SDK_Linux_JP.run && \
    rm -rf /var/lib/apt/lists/*

#This symbolic link is needed to use the streaming features on Jetson inside a container
RUN ln -sf /usr/lib/aarch64-linux-gnu/tegra/libv4l2.so.0 /usr/lib/aarch64-linux-gnu/libv4l2.so

#ROS system setup 
ENV LANG en_US.UTF-8
ENV LC_ALL en_US.UTF-8
ENV ROS_DISTRO humble

RUN apt-get update -y && apt-get install --no-install-recommends -y software-properties-common && \
    add-apt-repository universe && \
    curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key  -o /usr/share/keyrings/ros-archive-keyring.gpg && \
    /bin/bash -c 'echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(source /etc/os-release && echo $UBUNTU_CODENAME)  main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null' && \
    rm -rf /var/lib/apt/lists/* && \
    apt-get clean

RUN apt-get update -y && apt-get install -y ros-${ROS_DISTRO}-ros-base && \
    /bin/bash -c 'source /opt/ros/${ROS_DISTRO}/setup.bash' && \
    rm -rf /var/lib/apt/lists/*

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
