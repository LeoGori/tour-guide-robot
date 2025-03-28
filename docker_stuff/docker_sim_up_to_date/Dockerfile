ARG base_img
FROM $base_img
LABEL maintainer="Ettore Landini"

ARG ros_distro
ARG yarp_branch="master"
ARG ros2_dev_remote="robotology"
ARG ros2_dev_branch=${yarp_branch}
ARG gazebo_pkg=ros-${ros_distro}-ros-gz
ARG gazebo_version=harmonic

# Non-interactive installation mode
ENV DEBIAN_FRONTEND=noninteractive
ENV DOCKER_UPDATED_ON=06_06_2024

# Install essentials
RUN apt-get update && apt-get install -y apt software-properties-common sudo psmisc lsb-release \
    tmux nano geany vim wget curl build-essential git gitk cmake cmake-curses-gui autoconf xserver-xorg-video-dummy xserver-xorg-legacy \
    net-tools terminator libjpeg-dev ffmpeg apt-transport-https ca-certificates gnupg libace-dev ycm-cmake-modules locales \
    python3-setuptools python3-pip iproute2 python3-tornado python-dev-is-python3 swig lsof iftop iputils-ping gdb bash-completion btop pipx && rm -rf /var/lib/apt/lists/*

RUN pip3 install --break-system-packages numpy bpytop openwakeword

RUN sed -i 's/allowed_users=console/allowed_users=anybody/' /etc/X11/Xwrapper.config
COPY ./common/xorg.conf /etc/X11/xorg.conf

# ROS2 Installation
RUN locale-gen en_US en_US.UTF-8
RUN update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
ENV LANG=en_US.UTF-8
RUN curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
RUN echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null
RUN apt-get update && apt-get upgrade -y && apt-get install -y ros-${ros_distro}-desktop ros-${ros_distro}-navigation2 ros-${ros_distro}-nav2-bringup \
    ros-${ros_distro}-slam-toolbox ros-${ros_distro}-test-msgs ${gazebo_pkg} \
    python3-colcon-common-extensions python3-flake8 python3-flake8-blind-except python3-flake8-builtins \
    python3-flake8-class-newline python3-flake8-comprehensions python3-flake8-deprecated python3-flake8-docstrings \
    python3-flake8-import-order python3-flake8-quotes python3-pytest python3-pytest-cov python3-pytest-repeat \
    python3-pytest-rerunfailures python3-rosdep python3-vcstool && rosdep init && rm -rf /var/lib/apt/lists/*

# Install yarp dependencies
RUN apt-get update && apt-get install -y libode8 libace-dev libboost-filesystem-dev libboost-system-dev \
    libboost-thread-dev libc6 python3 libgsl0-dev libncurses5-dev libsdl1.2-dev subversion git gfortran \
    libxmu-dev libode-dev wget unzip qtbase5-dev qtdeclarative5-dev qtmultimedia5-dev libqt5svg5 \
    libqt5opengl5-dev libopencv-dev freeglut3-dev libtinyxml-dev libblas-dev coinor-libipopt-dev \
    liblapack-dev libmumps-dev qml-module-qtmultimedia qml-module-qtquick-dialogs qml-module-qtquick-controls \
    qml-module-qt-labs-settings libqcustomplot-dev qml-module-qt-labs-folderlistmodel libedit-dev \
    libeigen3-dev libjpeg-dev libsimbody-dev libxml2-dev libjs-underscore portaudio19-dev libgtkmm-2.4-dev \
    libi2c-dev libv4l-dev

# Create user: user1
USER root
RUN useradd -l -u 33334 -G sudo -md /home/user1 -s /bin/bash -p user1 user1 && \
    # passwordless sudo for users in the 'sudo' group
    sed -i.bkp -e 's/%sudo\s\+ALL=(ALL\(:ALL\)\?)\s\+ALL/%sudo ALL=NOPASSWD:ALL/g' /etc/sudoers

# ROS2 Installation (part2)
USER user1
RUN rosdep fix-permissions && rosdep update

# Set ENV
USER user1
RUN sudo mkdir /usr/local/src/robot
ENV robotology_install_user user1
ENV user1_home /home/${robotology_install_user}
ENV robotology_install_folder /usr/local/src/robot
RUN sudo chown -R ${robotology_install_user}:${robotology_install_user} ${robotology_install_folder}

#gazebo models
RUN mkdir -p $user1_home/.gazebo && git clone https://github.com/osrf/gazebo_models $user1_home/.gazebo/models

# Build ycm
USER ${robotology_install_user}
WORKDIR ${robotology_install_folder}
RUN git clone https://github.com/robotology/ycm.git -b master
RUN cd ycm && mkdir build && cd build && \
    cmake .. \
    -DCMAKE_BUILD_TYPE=Release && \
    make -j11
ENV YCM_DIR=${robotology_install_folder}/ycm/build

# Build YARP
USER ${robotology_install_user}
WORKDIR ${robotology_install_folder}
RUN git clone https://github.com/robotology/yarp.git -b ${yarp_branch}
RUN cd yarp && mkdir build && cd build && \
    cmake .. \
    -DCMAKE_BUILD_TYPE=Release \
    -DYARP_COMPILE_libYARP_math=ON \
    -DYARP_COMPILE_GUIS=ON \
    -DYARP_COMPILE_DEVICE_PLUGINS=ON \
    -DENABLE_yarpcar_mjpeg=ON \
    -DENABLE_yarpcar_segmentationimage=ON \
    -DENABLE_yarpcar_portmonitor=ON \
    -DENABLE_yarpmod_portaudio=ON \
    -DENABLE_yarpmod_portaudioPlayer=ON \
    -DENABLE_yarpmod_portaudioRecorder=ON \
    -DENABLE_yarpmod_fakeAnalogSensor=ON \
    -DENABLE_yarpmod_fakeBattery=ON  \
    -DENABLE_yarpmod_fakeDepthCamera=ON \
    -DENABLE_yarpmod_fakeIMU=ON  \
    -DENABLE_yarpmod_fakeLaser=ON  \
    -DENABLE_yarpmod_fakeLocalizer=ON \
    -DENABLE_yarpmod_fakeMicrophone=ON  \
    -DENABLE_yarpmod_fakeMotionControl=ON  \
    -DENABLE_yarpmod_fakeNavigation=ON  \
    -DENABLE_yarpmod_fakeSpeaker=ON  \
    -DENABLE_yarpmod_fakebot=ON \
    -DENABLE_yarpmod_laserFromDepth=ON \
    -DENABLE_yarpmod_laserFromExternalPort=ON \
    -DENABLE_yarpmod_laserFromDepth=ON \
    -DENABLE_yarpmod_laserFromPointCloud=ON \
    -DENABLE_yarpmod_laserFromRosTopic=ON \
    -DENABLE_yarppm_soundfilter_resample=ON \
    -DENABLE_yarppm_bottle_compression_zlib=ON \
    -DENABLE_yarppm_depthimage_compression_zlib=ON \
    -DENABLE_yarppm_image_compression_ffmpeg=ON \
    -DENABLE_yarppm_depthimage_to_mono=ON \
    -DENABLE_yarppm_depthimage_to_rgb=ON && \
    make -j11
ENV YARP_ROOT=${robotology_install_folder}/yarp
ENV YARP_DIR=${robotology_install_folder}/yarp/build

# YARP bindings
# USER ${robotology_install_user}
# WORKDIR ${robotology_install_folder}
# RUN cd yarp && cd bindings &&  mkdir build && cd build && cmake .. -DCREATE_PYTHON=ON && make -j11

# Install YARP completion
RUN sudo ln -s /usr/local/share/bash-completion/completions/yarp /usr/share/bash-completion/completions

# Build icub-main
USER ${robotology_install_user}
WORKDIR ${robotology_install_folder}
RUN git clone https://github.com/robotology/icub-main.git -b devel
RUN cd icub-main && mkdir build && cd build && \
    cmake .. \
    -DCMAKE_BUILD_TYPE=Release \
    -DENABLE_icubmod_cartesiancontrollerserver=ON \
    -DENABLE_icubmod_cartesiancontrollerclient=ON \
    -DICUBMAIN_COMPILE_SIMULATORS=OFF \
    -DENABLE_icubmod_gazecontrollerclient=ON && \
    make -j11
ENV ICUB_ROOT=${robotology_install_folder}/icub-main
ENV ICUB_DIR=${robotology_install_folder}/icub-main/build

# Build robots-configuration devel
USER ${robotology_install_user}
WORKDIR ${robotology_install_folder}
RUN git clone https://github.com/randaz81/robots-configuration.git -b R1SN003_ros2

# Build navigation
USER ${robotology_install_user}
WORKDIR ${robotology_install_folder}
RUN git clone https://github.com/robotology/navigation.git -b master
RUN cd navigation && mkdir build && cd build && /bin/bash -c "source /opt/ros/${ros_distro}/setup.bash; cmake .. -DCMAKE_BUILD_TYPE=Release -DNAVIGATION_USE_NWC=ON -DNAVIGATION_USE_ROS2=ON" && make -j11
ENV navigation_ROOT=${robotology_install_folder}/navigation
ENV navigation_DIR=${robotology_install_folder}/navigation/build


# Gazebo
RUN sudo curl https://packages.osrfoundation.org/gazebo.gpg --output /usr/share/keyrings/pkgs-osrf-archive-keyring.gpg
RUN echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/pkgs-osrf-archive-keyring.gpg] http://packages.osrfoundation.org/gazebo/ubuntu-stable $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/gazebo-stable.list > /dev/null
RUN sudo apt-get update
RUN sudo apt-get install -y gz-${gazebo_version}

# Gz-sim plugins dependencies
WORKDIR /home/${robotology_install_user}
RUN git clone https://github.com/CLIUtils/CLI11
RUN cd CLI11/ && \
    mkdir build && \
    cd build && \
    git submodule update --init && \
    cmake .. && \
    cmake --build . && \
    sudo cmake --install .

# CER
USER ${robotology_install_user}
WORKDIR ${robotology_install_folder}
RUN git clone https://github.com/robotology/cer.git
RUN cd cer && mkdir build && cd build && cmake .. \
            -DCMAKE_BUILD_TYPE=Release \
            -DENABLE_cermod_cerDoubleLidar=ON \
            -DENABLE_cermod_cerOdometry=ON \
            -DENABLE_cermod_tripodMotionControl=ON && make -j11
ENV CER_ROOT=${robotology_install_folder}/cer
ENV CER_DIR=${robotology_install_folder}/cer/build

# YARP-DEVICES-ROS2
WORKDIR ${robotology_install_folder}
RUN git clone https://github.com/${ros2_dev_remote}/yarp-devices-ros2 -b ${ros2_dev_branch}
RUN cd yarp-devices-ros2/ros2_interfaces_ws && /bin/bash -c "source /opt/ros/${ros_distro}/setup.bash; colcon build"
RUN cd yarp-devices-ros2 && mkdir build && cd build && /bin/bash -c "source /opt/ros/${ros_distro}/setup.bash; source ${robotology_install_folder}/yarp-devices-ros2/ros2_interfaces_ws/install/setup.bash; cmake .. -DYARP_ROS2_USE_SYSTEM_map2d_nws_ros2_msgs=ON -DYARP_ROS2_USE_SYSTEM_yarp_control_msgs=ON; make -j11"

# CER-Sim stuff
USER ${robotology_install_user}
WORKDIR ${robotology_install_folder}
RUN git clone https://github.com/robotology/cer-sim -b ros2
RUN cd cer-sim/colcon_ws && /bin/bash -c "source /opt/ros/${ros_distro}/setup.bash; colcon build; source ${robotology_install_folder}/cer-sim/colcon_ws/install/local_setup.bash"

# Build icub-contrib-common
USER ${robotology_install_user}
WORKDIR ${robotology_install_folder}
RUN git clone https://github.com/robotology/icub-contrib-common.git
RUN cd icub-contrib-common && mkdir build && cd build && cmake .. -DCMAKE_INSTALL_PREFIX=${robotology_install_folder}/iCubContrib && make -j11 install
ENV ICUBcontrib_DIR=${robotology_install_folder}/iCubContrib/build

# Gz-sim Plugins
USER ${robotology_install_user}
WORKDIR ${robotology_install_folder}
RUN cd yarp/build && sudo make install
RUN git clone https://github.com/robotology/gz-sim-yarp-plugins
RUN cd gz-sim-yarp-plugins && mkdir build && cd build && \
cmake .. && make -j11

# r1-models
USER ${robotology_install_user}
WORKDIR ${robotology_install_folder}
RUN mkdir installed
RUN git https://github.com/icub-tech-iit/r1-models
RUN cd r1-models && cmake -S . -B build -DCMAKE_INSTALL_PREFIX=${robotology_install_user}/installed && \
    cmake --build build && cmake --build build --target install

# Set environmental variables
RUN echo "PS1='\[\e]0;\u \w\a\]\[\033[01;32m\]\u\[\033[00m\] \[\033[01;34m\]\w\[\033[00m\] \$ '" >> /home/${robotology_install_user}/.bashrc
ENV GZ_SIM_YARP_PLUGINS_SOURCE_DIR=${robotology_install_user}/gz-sim-yarp-plugins
ENV GZ_SIM_YARP_PLUGINS_DIR=${GZ_SIM_YARP_PLUGINS_SOURCE_DIR}/build
ENV GZ_SIM_SYSTEM_PLUGIN_PATH=${GZ_SIM_SYSTEM_PLUGIN_PATH}:${GZ_SIM_YARP_PLUGINS_DIR}/lib
ENV R1_MODELS_SOURCE_DIR=${robotology_install_user}/r1-models
ENV GZ_SIM_RESOURCE_PATH=${GZ_SIM_RESOURCE_PATH}:${robotology_install_user}/installed/share:${robotology_install_user}/installed/share/R1Mk3/robots
ENV TOUR_GUIDE_ROBOT_SOURCE_DIR=${robotology_install_folder}/tour-guide-robot
ENV ROBOT_CODE=${robotology_install_folder}
ENV CER_SIM_ROOT_DIR=${ROBOT_CODE}/cer-sim
ENV PATH=$PATH:$YARP_DIR/bin:$navigation_DIR/bin:$CER_DIR/bin:$ICUB_DIR/bin:$ICUB_DIR:${robotology_install_folder}/iCubContrib/bin:${robotology_install_folder}/Groot/build:${robotology_install_folder}/tour-guide-robot/build/bin:${TOUR_GUIDE_ROBOT_SOURCE_DIR}/app/navigation2/scripts:${TOUR_GUIDE_ROBOT_SOURCE_DIR}/app/headSynchronizer/scripts
ENV DISPLAY=:1
ENV LD_LIBRARY_PATH=${robotology_install_folder}/yarp/build/lib/yarp/
ENV GAZEBO_PLUGIN_PATH=$GazeboYARPPlugins_DIR/lib
ENV GAZEBO_MODEL_PATH=${robotology_install_folder}/cer-sim/gazebo/
ENV YARP_COLORED_OUTPUT=1
ENV QT_X11_NO_MITSHM=1
ENV PYTHONPATH=$PYTHONPATH:${robotology_install_folder}/yarp/bindings/build/lib/python3/
ENV YARP_CLOCK=/clock
#ENV CYCLONEDDS_URI=${robotology_install_folder}/tour-guide-robot/app/navigation2/conf/cyclone_dds_settings.xml

# Manage yarp port
EXPOSE 10000/tcp 10000/udp

# Launch bash from ${robotology_install_folder}
USER ${robotology_install_user}
RUN sudo chown -R ${robotology_install_user}:${robotology_install_user} /opt/ros && sudo chown -R ${robotology_install_user}:${robotology_install_user} /home/${robotology_install_user}/.ros

WORKDIR ${robotology_install_folder}
RUN git clone https://github.com/elandini84/yarp-web-teleop.git && cd yarp-web-teleop && mkdir build && cd build && cmake ..
RUN git clone https://github.com/elandini84/libfvad && cd libfvad && autoreconf -i && ./configure && sudo make install
RUN git clone https://github.com/hsp-iit/tour-guide-robot -b ${ros_distro}
RUN cd tour-guide-robot && mkdir build && cd build && cmake .. -DCMAKE_BUILD_TYPE=Release && make -j11

ENV WEB_TELEOP_DIR=${robotology_install_folder}/yarp-web-teleop/build
ENV YARP_DATA_DIRS=$YARP_DIR/share/yarp:$ICUB_DIR/share/iCub:$navigation_DIR/share/navigation:${TOUR_GUIDE_ROBOT_SOURCE_DIR}/build/share/tour-guide-robot:$CER_DIR/share/CER:$ICUBcontrib_DIR/share/ICUBcontrib:${robotology_install_folder}/yarp-devices-ros2/build/share/yarp:${robotology_install_folder}/yarp-devices-ros2/build/share/yarp-devices-ros2:${WEB_TELEOP_DIR}/share/yarp-web-teleop

# R1 NETWORK CONFIG
RUN yarp conf 172.17.0.1 10000

# Possibly temporary ROS2 network middleware change
RUN sudo apt-get update && sudo apt-get install -y cyclonedds-dev cyclonedds-tools \
ros-${ros_distro}-cyclonedds-dbgsym ros-${ros_distro}-cyclonedds ros-${ros_distro}-rmw-cyclonedds-cpp \
ros-${ros_distro}-rmw-cyclonedds-cpp-dbgsym libcycloneddsidl0 && \
sudo apt-get clean && sudo rm -rf /var/lib/apt/lists/*
ENV RMW_IMPLEMENTATION=rmw_cyclonedds_cpp

# Set ROS2 environment
RUN echo "#!/bin/bash" >> /home/user1/.entrypoint.sh
RUN echo "source /opt/ros/${ros_distro}/setup.bash" >> /home/user1/.entrypoint.sh
RUN echo "source ${robotology_install_folder}/yarp-devices-ros2/ros2_interfaces_ws/install/local_setup.bash" >> /home/user1/.entrypoint.sh
RUN echo 'exec "$@"' >> /home/user1/.entrypoint.sh
RUN chmod +x /home/user1/.entrypoint.sh

WORKDIR ${user1_home}
#RUN sudo updatedb
ENTRYPOINT [ "/home/user1/.entrypoint.sh" ]
CMD ["bash"]
