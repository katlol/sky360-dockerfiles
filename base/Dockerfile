FROM debian:testing-slim AS builder

ENV TZ=Etc/UTC \
    DEBIAN_FRONTEND=noninteractive \
    OPENCV_VERSION=4.7.0

# Install dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
      lsb-release build-essentials git cmake pkg-config wget unzip openssh-client ca-certificates && \
      # FIXME add all common dependencies here
    rm -rf /var/lib/apt/lists/* && apt-get clean && apt-get autoclean

# OpenCV

FROM builder AS opencv

RUN TMPDIR=$(mktemp -d) && cd $TMPDIR && \
    # FIXME add all dependencies to build opencv here
    wget -qO opencv-$OPENCV_VERSION.zip https://github.com/opencv/opencv/archive/$OPENCV_VERSION.zip && \
    wget -qO opencv_contrib-$OPENCV_VERSION.zip https://github.com/opencv/opencv_contrib/archive/$OPENCV_VERSION.zip && \
    unzip opencv-$OPENCV_VERSION.zip && \
    unzip opencv_contrib-$OPENCV_VERSION.zip && \
    rm -f opencv-$OPENCV_VERSION.zip && \
    rm -f opencv_contrib-$OPENCV_VERSION.zip && \
    cd opencv-$OPENCV_VERSION && mkdir build && cd build && \
    cmake -D CMAKE_BUILD_TYPE=RELEASE \
          -D CMAKE_INSTALL_PREFIX=/usr/local \
          -D OPENCV_GENERATE_PKGCONFIG=YES \
          -D ENABLE_PRECOMPILED_HEADERS=OFF \
          -D WITH_LIBV4L=ON \
          -D WITH_QT=ON \
          -D WITH_OPENGL=ON \
          -D WITH_OPENCL=ON \
          -D WITH_JAVA=OFF \
          -D WITH_CUDA=OFF \
          -D BUILD_opencv_java=OFF \
          -D BUILD_opencv_python=OFF \
          -D BUILD_opencv_python2=OFF \
          -D BUILD_opencv_python3=ON \
          -D PYTHON3_EXECUTABLE=$(which python3) \
          -D PYTHON3_INCLUDE_DIR=$(python3 -c "from distutils.sysconfig import get_python_inc; print(get_python_inc())") \
          -D PYTHON3_PACKAGES_PATH=$(python3 -c "from distutils.sysconfig import get_python_lib; print(get_python_lib())") \
          -D INSTALL_PYTHON_EXAMPLES=OFF \
          -D BUILD_NEW_PYTHON_SUPPORT=ON \
          -D ENABLE_CXX11=ON \
          -D WITH_TBB=ON \
          -D BUILD_TESTS=OFF \
          -D INSTALL_TESTS=OFF \
          -D BUILD_PERF_TESTS=OFF \
          -D BUILD_EXAMPLES=OFF \
          -D INSTALL_C_EXAMPLES=OFF \
          -D OPENCV_ENABLE_NONFREE=ON \
          -D OPENCV_EXTRA_MODULES_PATH=../../opencv_contrib-$OPENCV_VERSION/modules \
          -D OpenGL_GL_PREFERENCE=LEGACY \
      .. && \
    make -j $(nproc) && \
    make install && \
    # FIXME instead of make install, create a tarball to copy to the final image?
    sh -c 'echo "/usr/local/lib" >> /etc/ld.so.conf.d/opencv.conf' && \
    ldconfig && \
    rm -rf $TMPDIR


# FIXME what is the correct way to add opencv to python path?
#ENV PYTHONPATH=$PYTHONPATH:/usr/lib/python3/dist-packages/cv2/python-3.10/


# BGS Library

FROM builder AS bgslibrary
# FIXME Copy opencv from previous stage
#COPY --from=opencv /usr/local/ /usr/local/ ???

RUN TMPDIR=$(mktemp -d) && cd $TMPDIR && \
# FIXME add all dependencies to build bgslibrary here
    git clone --depth=1 --recursive https://github.com/Sky360-Repository/bgslibrary.git && \
    mkdir -p bgslibrary/build && \
    cd bgslibrary/build && \
    cmake .. && make -j $(nproc) && cd .. && \
    python setup.py build && python setup.py install && \
    cd $TMPDIR/bgslibrary/gui/qt/build && \
    cmake .. && make -j $(nproc) && cp bgslibrary_gui ../../.. && \
    cd $TMPDIR/bgslibrary/examples/build && \
    cmake .. && make -j $(nproc) && \
    rm -rf $TMPDIR


# QHY SDK
FROM builder AS qhy
# FIXME copy any requirements from previous stages
RUN TMPDIR=$(mktemp -d) && cd $TMPDIR && \
# FIXME add all dependencies to build qhy here
    AMD64_LINK=https://www.qhyccd.com/file/repository/publish/SDK/230228/sdk_linux64_23.02.28.tgz && \
    ARM64_LINK=https://www.qhyccd.com/file/repository/publish/SDK/230228/sdk_Arm64_23.02.28.tgz && \
    # detect
    if [ "$(uname -m)" = "x86_64" ]; then \
        DOWNLOAD_LINK=$AMD64_LINK; \
    else \
        DOWNLOAD_LINK=$ARM64_LINK; \
    fi && \
    # download
    wget -q $DOWNLOAD_LINK -O $TMPDIR/sdk.tgz && \
    # extract
    tar -xf $TMPDIR/sdk.tgz -C $TMPDIR && \
    # install
    cd $TMPDIR/sdk_* && \
    bash install.sh && \
    # cleanup
    rm -rf $TMPDIR


# Sky360Lib
FROM builder AS sky360lib
# FIXME copy any requirements from previous stages

RUN TMPDIR=$(mktemp -d) && cd $TMPDIR && \
    # FIXME add all dependencies to build sky360lib here
    git clone --depth=1 --recursive https://github.com/Sky360-Repository/sky360lib.git && \
    mkdir -p sky360lib/build && \
    cd sky360lib/build && \
    cmake -DCMAKE_INSTALL_PREFIX=/usr/local .. && \
    cmake --build . -j$(nproc) && \
    make install && \
    rm -rf $TMPDIR
ENV PYTHONPATH=$PYTHONPATH:/usr/local/lib/python3.10/dist-packages/sky360lib:/opt/sky360lib/build/lib


### from https://github.com/Sky360-Repository/dockerfiles/blob/master/current/ros2/Dockerfile-base

# ROS
FROM builder AS ros2
# FIXME copy any requirements from previous stages
# FIXME maybe this should be the final image?
# is there any reason to have a separate image for ros2?

RUN apt-get update && apt-get install -y lsb-release && \
    curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" > /etc/apt/sources.list.d/ros2.list && \
# vulcanexus
    curl -sSL https://raw.githubusercontent.com/eProsima/vulcanexus/main/vulcanexus.key -o /usr/share/keyrings/vulcanexus-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/vulcanexus-archive-keyring.gpg] http://repo.vulcanexus.org/debian $(lsb_release -cs) main" > /etc/apt/sources.list.d/vulcanexus.list && \
    apt-get update && apt-get install -y \
      locales \
      curl \
      gnupg2 \
      lsb-release \
      sudo \
      tzdata \
    && \
    locale-gen en_GB.UTF-8 && \
    update-locale LC_ALL=en_GB.UTF-8 LANG=en_GB.UTF-8 && \
    dpkg-reconfigure --frontend noninteractive tzdata && \
    # Install ROS2
    apt-get install -y \
      ros-humble-ros-base \
      ros-humble-rosbridge-server \
      python3-argcomplete && \
    # Install vulcanexus-humble-base
    apt-get install -y \
      vulcanexus-humble-base && \
    rm -rf /var/lib/apt/lists/* && apt-get clean && apt-get autoclean
    ## BEGIN Prometheus Stuff

# Install prometheus
FROM builder AS prometheus
# FIXME copy any requirements from previous stages
RUN TMPDIR=$(mktemp -d) && cd $TMPDIR && \
    # FIXME add all dependencies to build prometheus here
    git clone --depth=1 --recursive https://github.com/eProsima/prometheus-cpp.git && \
    mkdir -p prometheus-cpp/build && \
    cd prometheus-cpp/build && \
    cmake .. -DBUILD_SHARED_LIBS=ON -DENABLE_PUSH=OFF -DENABLE_COMPRESSION=OFF && \
    cmake --build . --parallel 4 && \
    cmake --install . && \
    # ^ FIXME instead of make install, create a tarball to copy to the final image?
    rm -rf $TMPDIR

## END Prometheus Stuff
FROM debian:slim
COPY --from=opencv /usr/local/ /usr/local/
COPY --from=bgslibrary /usr/local/ /usr/local/
COPY --from=qhy /usr/local/ /usr/local/
COPY --from=sky360lib /usr/local/ /usr/local/
COPY --from=ros2 /usr/ /usr/
COPY --from=prometheus /usr/local/ /usr/local/
# ^ FIXME the paths above are not correct, but I don't know how to fix them
# one should copy only the required files, not the whole /usr/local
# maybe it should be make installed in a temporary directory and then copied to the final image?
# or maybe not installed at all, but just copied to the final image?
# FIXME FIXME
# run dependencies
RUN apt-get update && apt-get install -y \
      locales \
      curl \
      gnupg2 \
      lsb-release \
      sudo \
      tzdata \
    && \
    locale-gen en_GB.UTF-8 && \
    update-locale LC_ALL=en_GB.UTF-8 LANG=en_GB.UTF-8 && \
    dpkg-reconfigure --frontend noninteractive tzdata && \
    rm -rf /var/lib/apt/lists/* && apt-get clean && apt-get autoclean

ENV ROS_DISTRO=humble \
    AMENT_PREFIX_PATH=/opt/ros/humble \
    COLCON_PREFIX_PATH=/opt/ros/humble \
    LD_LIBRARY_PATH=/opt/ros/humble/lib \
    PATH=/opt/ros/humble/bin:$PATH \
    PYTHONPATH=$PYTHONPATH:/opt/ros/humble/lib/python3.10/site-packages \
    ROS_PYTHON_VERSION=3 \
    ROS_VERSION=2 \
    DEBIAN_FRONTEND= \
    LANG=en_GB.UTF-8
