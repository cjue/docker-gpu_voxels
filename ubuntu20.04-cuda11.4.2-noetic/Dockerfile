FROM nvidia/cuda:11.4.2-devel-ubuntu20.04

# ROS

# install bootstrap tools
RUN apt-get update && apt-get install --no-install-recommends -y \
    apt-utils \
    && rm -rf /var/lib/apt/lists/*

# setup timezone
RUN echo 'Etc/UTC' > /etc/timezone && \
    ln -s /usr/share/zoneinfo/Etc/UTC /etc/localtime && \
    apt-get update && \
    apt-get install -q -y --no-install-recommends tzdata && \
    rm -rf /var/lib/apt/lists/*

# install packages
RUN apt-get update && apt-get install -q -y --no-install-recommends \
    dirmngr \
    gnupg2 \
    && rm -rf /var/lib/apt/lists/*

# setup sources.list
RUN echo "deb http://packages.ros.org/ros/ubuntu focal main" > /etc/apt/sources.list.d/ros1-latest.list

# setup keys
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C1CF6E31E6BADE8868B172B4F42ED6FBAB17C654

# setup environment
ENV LANG C.UTF-8
ENV LC_ALL C.UTF-8

ENV ROS_DISTRO noetic

# setup entrypoint
COPY ./ros_entrypoint.sh /

ENTRYPOINT ["/ros_entrypoint.sh"]
CMD ["bash"]

# install bootstrap tools
RUN apt-get update && apt-get install --no-install-recommends -y \
    build-essential \
    python3-rosdep \
    python3-rosinstall \
    python3-vcstools \
    && rm -rf /var/lib/apt/lists/*

# bootstrap rosdep
RUN rosdep init && \
  echo ROS_DISTRO: $ROS_DISTRO
  #rosdep update --rosdistro $ROS_DISTRO

# install ros packages
    
# ROS dekstop full
RUN apt-get update && apt-get install -y --no-install-recommends \
    ros-${ROS_DISTRO}-desktop-full \
    && rm -rf /var/lib/apt/lists/*


# GPU-Voxels

RUN apt-get update && apt-get upgrade -y &&\
    # Install build tools, build dependencies and python
    apt-get install -y \
	python3-pip \
        build-essential \
        cmake \
        ninja-build \
        git \
        wget \
        unzip \
        pkg-config \
        libglm-dev \
        libglew-dev \
        freeglut3-dev \
        libtinyxml-dev \
    && rm -rf /var/lib/apt/lists/*

# make sure "source" works
RUN rm /bin/sh && ln -s /bin/bash /bin/sh

ARG GVL_BRANCH=master
#ARG GVL_BRANCH=docker
# ARG GVL_REPO=https://github.com/fzi-forschungszentrum-informatik/gpu-voxels.git
# ARG GVL_REPO=/home/juelg/gpu_voxels_oss_release/gpu-voxels/.git
ARG GVL_REPO="/local_gpu_voxels.git"

COPY $GVL_REPO /tmp/gpu-voxels.git

ARG GVL_REPO="/tmp/gpu-voxels.git"

RUN cd /opt/ &&\
    echo repo: $GVL_REPO &&\
    git clone $GVL_REPO -b $GVL_BRANCH &&\
    # Create build folder and switch to it
    mkdir /opt/gpu-voxels/build && cd /opt/gpu-voxels/build &&\
    echo ROS_DISTRO: $ROS_DISTRO &&\
    source /opt/ros/$ROS_DISTRO/setup.bash && \
    sed -i '2 s/^/#/' ../packages/CMakeLists.txt &&\
    # Cmake configure
    cmake \
        -GNinja \
        -DCMAKE_BUILD_TYPE=Debug \
        # Install path will be /usr/local/lib (lib is implicit) 
        -DCMAKE_INSTALL_PREFIX=/opt/gpu-voxels/export \
        .. &&\
    # Make
    pwd &&\
    ls -l &&\
    ninja && \
    # Install to /opt/gpu-voxels/ or /usr/local/lib
    ninja install && \
    ldconfig &&\
    ls -lh /opt/gpu-voxels/export

RUN cd /opt/gpu-voxels/build &&\
    echo ROS_DISTRO: $ROS_DISTRO &&\
    source /opt/ros/$ROS_DISTRO/setup.bash && \
    git checkout ../packages/ &&\
    git pull origin master &&\
    # Cmake configure
    cmake \
        -GNinja \
        -DCMAKE_BUILD_TYPE=Debug \
        # Install path will be /usr/local/lib (lib is implicit) 
        -DCMAKE_INSTALL_PREFIX=/opt/gpu-voxels/export \
        .. &&\
    # Make
    pwd &&\
    ls -l &&\
    ninja && \
    # Install to /opt/gpu-voxels/ or /usr/local/lib
    ninja install && \
    ldconfig &&\
    ls -lh /opt/gpu-voxels/export
