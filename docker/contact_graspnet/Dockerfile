# syntax=docker/dockerfile:experimental

ARG from=ubuntu:bionic
FROM ${from}

# Change default shell to bash
SHELL ["/bin/bash", "-c"]

# Install packages
# ================
RUN apt-key adv --keyserver keyserver.ubuntu.com --recv-keys A4B469963BF863CC &&\
    apt-get update &&\
    apt-get install -y --no-install-recommends \
        sudo \
        gosu \
        software-properties-common \
        git \
        wget \
        curl \
        nano \
        unzip \
        libgl1-mesa-glx \
        libgl1-mesa-dri \
        apt-transport-https \
        apt-utils \
        dbus-x11 \
        bash-completion \
        gdb \
        ssh \
        iputils-ping \
        gnupg \
        gnupg2 \
        xvfb \
        &&\
    rm -rf /var/lib/apt/lists/*

RUN wget -O - https://apt.kitware.com/keys/kitware-archive-latest.asc 2>/dev/null | apt-key add - && \
    apt-add-repository "deb https://apt.kitware.com/ubuntu/ `lsb_release -cs` main" &&\
    add-apt-repository ppa:ubuntu-toolchain-r/test &&\
    apt-get update &&\
    apt-get install -y --no-install-recommends \
        gcc-8 \
        g++-8 \
        cmake \
        cmake-curses-gui \
        build-essential \
        &&\
    rm -rf /var/lib/apt/lists/*

# Setup locales and timezone
ENV TZ=Europe/Rome
ENV LANG=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8
ENV LANGUAGE=en_US.UTF-8
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update &&\
    apt-get install -y --no-install-recommends locales locales-all tzdata &&\
    rm -rf /var/lib/apt/lists/* &&\
    echo "${TZ}" | tee /etc/timezone &&\
    dpkg-reconfigure --frontend noninteractive tzdata

# Install python3
# ===============
ENV PYTHON_VERSION=3.6
RUN add-apt-repository ppa:deadsnakes/ppa &&\
    apt-get update &&\
    apt-get install -y --no-install-recommends \
        python${PYTHON_VERSION} \
        python${PYTHON_VERSION}-dev \
        libpython${PYTHON_VERSION}-dev \
        python3-pip \
        python3-setuptools \
        python3-wheel \
        python3-pygments \
        python3-numpy \
        &&\
    rm -rf /var/lib/apt/lists/*
RUN python3 -m pip install --upgrade pip

# Install ROS Desktop Full
# ========================

# https://github.com/osrf/docker_images/blob/master/ros/
ENV ROS_DISTRO=melodic

RUN apt-key adv --keyserver keyserver.ubuntu.com \
                --recv-keys F42ED6FBAB17C654 &&\
    echo "deb http://packages.ros.org/ros/ubuntu `lsb_release -cs` main" \
        > /etc/apt/sources.list.d/ros-latest.list &&\
    apt-get update &&\
    apt-get install -y --no-install-recommends \
        ros-${ROS_DISTRO}-ros-base \
        ros-${ROS_DISTRO}-perception \
        ros-${ROS_DISTRO}-geometry2 \
        ros-${ROS_DISTRO}-vision-opencv \
	ros-${ROS_DISTRO}-jsk-tools \
        python-rosdep \
        python-rosinstall \
        python-rosinstall-generator \
        python-vcstools \
        python-wstool \
        python3-catkin-tools \
        python3-catkin-pkg-modules \
        python3-rospkg-modules \
        python3-empy \
        libgtk-3-dev \
        &&\
    rm -rf /var/lib/apt/lists/* &&\
    rosdep init &&\
    rosdep update &&\
    echo "source /opt/ros/${ROS_DISTRO}/setup.bash" >> /etc/bash.bashrc

RUN apt-key adv --keyserver keyserver.ubuntu.com \
                --recv-keys 6AF7F09730B3F0A4 &&\
                apt-get update &&\
                apt-get install ros-${ROS_DISTRO}-ros-numpy

# Set workspace directories
# =========================
ENV WS_DIR=/workspace
ENV CATKIN_WS_DIR=${WS_DIR}/catkin_ws
ENV WS_INSTALL=${WS_DIR}/install
ENV WS_SOURCES=${WS_DIR}/sources
ARG WS_BIN=${WS_SOURCES}/bin

# Create a new runtimeusers group
RUN groupadd -K GID_MIN=900 -K GID_MAX=1100 runtimeusers

# Create catkin_ws with dependencies/ros packages for python3
# ===========================================================
RUN mkdir -p ${CATKIN_WS_DIR}/src && \
    cd ${CATKIN_WS_DIR} &&\
    rosdep update && rosdep install --from-paths src --ignore-src -y -r &&\
    rm -rf /var/lib/apt/lists/* &&\
    catkin config --extend /opt/ros/${ROS_DISTRO} -DPYTHON_EXECUTABLE=/usr/bin/python3 -DCMAKE_BUILD_TYPE=Release && \
    catkin build  -j8 && \
    echo 'source ${CATKIN_WS_DIR}/devel/setup.bash' >> /etc/bash.bashrc &&\
    find ${CATKIN_WS_DIR} -not -group runtimeusers -exec chgrp runtimeusers {} \; -exec chmod g+rw {} \+

# Install ContactGraspNet and related dependencies
# =============================================

# RUN pip3 install vtk==8.1.0

RUN mkdir -p ${WS_SOURCES} &&\
     cd ${WS_SOURCES}

RUN cd ${WS_SOURCES} &&\
    git clone https://github.com/NVlabs/contact_graspnet.git &&\
    git clone https://github.com/hsp-panda/pointnet2.git &&\
    cd contact_graspnet/ &&\
    #pip3 install vtk==9.1.0 &&\
    #pip3 install --ignore-installed -r requirements.txt &&\
    pip install attrdict vtk==8.1.2 &&\
    pip install mayavi \
    trimesh \
    pyrender \
    easydict \
    pyqt5 \
    traitsui \
    python-fcl\ 
    gdown &&\
    bash -c "gdown https://drive.google.com/drive/folders/1tBHKf60K8DLM5arm-Chyf7jxkzOr5zGl --folder -O checkpoints"

###################################

ENV GRASPNET_DIR=${WS_SOURCES}/contact_graspnet

# # Install benchmark-grasping
# # ==========================

RUN cd ${CATKIN_WS_DIR}/src &&\
    echo "!!!!!!!!!" &&\
    git clone https://github.com/nicolas-schreiber/grasping-benchmarks-panda.git &&\
    pip install -e grasping-benchmarks-panda/ &&\
    cd ${CATKIN_WS_DIR} &&\
    rosdep update &&\
    catkin config --extend /opt/ros/${ROS_DISTRO} -DPYTHON_EXECUTABLE=/usr/bin/python3 -DCMAKE_BUILD_TYPE=Release -DGPD_ROS=OFF &&\
    catkin build  -j8 && \
    find ${CATKIN_WS_DIR} -not -group runtimeusers -exec chgrp runtimeusers {} \; -exec chmod g+rw {} \+

# =============
# FIX OWNERSHIP
# =============
RUN find ${WS_DIR} -not -group runtimeusers -exec chgrp runtimeusers {} \; -exec chmod g+rw {} \+ &&\
    find ${CATKIN_WS_DIR} -not -group runtimeusers -exec chgrp runtimeusers {} \; -exec chmod g+rw {} \+

# Nvidia runtime support
ENV NVIDIA_VISIBLE_DEVICES=all
ENV NVIDIA_DRIVER_CAPABILITIES=all

##############################################
# Image initialization
# ====================
ENV USERNAME=docker

# Grasp Planner Definition
ENV GRASP_BENCHMARK_CONTACTGRASPNET=true

# Include a custom bashrc
COPY bashrc /usr/etc/skel/bashrc-dev

COPY setup.sh /usr/sbin/setup.sh
COPY entrypoint.sh /usr/sbin/entrypoint.sh
RUN chmod 755 /usr/sbin/setup.sh
RUN chmod 755 /usr/sbin/entrypoint.sh
ENTRYPOINT ["/usr/sbin/entrypoint.sh"]
CMD ["bash"]
