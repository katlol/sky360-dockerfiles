FROM ubuntu:22.04

ENV TZ=Etc/UTC \
    DEBIAN_FRONTEND=noninteractive \
    OPENCV_VERSION=4.7.0

# Install dependencies
RUN TMPDIR=$(mktemp -d) && cd $TMPDIR && \
    ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone && \
    apt-get update && \
    apt-get install -y  \
    build-essential \
    cmake \
    pkg-config \
    wget \
    git \
    unzip \
    nano \
    curl \
    libgtk-3-dev \
    libavcodec-dev \
    libavformat-dev \
    libswscale-dev \
    libv4l-dev \
    libxvidcore-dev \
    libx264-dev \
    libjpeg-dev \
    libpng-dev \
    libtiff-dev \
    gfortran \
    openexr \
    libatlas-base-dev \
    python3-dev \
    python3-numpy \
    python3-pip \
    libtbb2 \
    libtbb-dev \
    libdc1394-25 \
    libdc1394-dev \
    libopenexr-dev \
    libunwind-dev \
    libgstreamer-plugins-base1.0-dev \
    libgstreamer-opencv1.0-0 \
    libgstreamer1.0-dev \
    liblapacke-dev \
    libva-dev libva-drm2 libva-x11-2 libva-glx2 \
    libhdf5-dev \
    qtbase5-dev \
    qtchooser \
    qt5-qmake \
    qtbase5-dev-tools \
    libvtk9-dev libvtk9-qt-dev \
    usbutils \
    libusb-1.0-0-dev \
    clang \
    libboost-all-dev && \
    curl https://sh.rustup.rs -sSf | sh -s -- -y && \
    ln -s /usr/bin/python3 /usr/bin/python && \
    # cleanup
    apt-get autoclean && apt-get clean \
    && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* $TMPDIR

# OpenCV

RUN TMPDIR=$(mktemp -d) && cd $TMPDIR && \
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
    sh -c 'echo "/usr/local/lib" >> /etc/ld.so.conf.d/opencv.conf' && \
    ldconfig && \
    rm -rf $TMPDIR

ENV PYTHONPATH=$PYTHONPATH:/usr/lib/python3/dist-packages/cv2/python-3.10/

# BGS Library


RUN TMPDIR=$(mktemp -d) && cd $TMPDIR && \
    git clone --recursive https://github.com/Sky360-Repository/bgslibrary.git && \
    cd bgslibrary/build && \
    cmake .. && make -j $(nproc) && cd .. && \
    python setup.py build && python setup.py install && \
    cd $TMPDIR/bgslibrary/gui/qt/build && \
    cmake .. && make -j $(nproc) && cp bgslibrary_gui ../../.. && \
    cd $TMPDIR/bgslibrary/examples/build && \
    cmake .. && make -j $(nproc) && \
    rm -rf $TMPDIR


# QHY SDK

RUN TMPDIR=$(mktemp -d) && cd $TMPDIR && \
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

RUN TMPDIR=$(mktemp -d) && cd $TMPDIR && \
    git clone --recursive https://github.com/Sky360-Repository/sky360lib.git && \
    cd sky360lib/build && \
    cmake -DCMAKE_INSTALL_PREFIX=/usr/local .. && \
    cmake --build . -j$(nproc) && \
    make install && \
    rm -rf $TMPDIR
ENV PYTHONPATH=$PYTHONPATH:/usr/local/lib/python3.10/dist-packages/sky360lib:/opt/sky360lib/build/lib
