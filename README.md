# jetson-ffmpeg
L4T Multimedia API for ffmpeg, works with nvcr.io/nvidia/l4t-jetpack:r36.4.0 (runtime nvidia)

mount directories from tegra to container or copy on build /usr/lib/aarch64-linux-gnu/tegra -> /usr/lib/aarch64-linux-gnu/tegra

apt install pkg-config libx264-dev libx265-dev g++ gcc git build-essential cmake

**1.build and install library**

    git clone https://github.com/Tantael/jetson-ffmpeg-fixed.git
    cd jetson-ffmpeg-fixed
    mkdir build
    cd build
    cmake ..
    make
    sudo make install
    sudo ldconfig
	
**2.patch ffmpeg and build**

    git clone git://source.ffmpeg.org/ffmpeg.git -b release/4.2 --depth=1
    cd ffmpeg
    wget https://github.com/jocover/jetson-ffmpeg/raw/master/ffmpeg_nvmpi.patch
    git apply --whitespace=fix ffmpeg_nvmpi.patch
    ./configure --enable-static \
    --enable-shared \
    --enable-nonfree \
    --enable-nvmpi \
    --enable-gpl \
    --disable-opencl \
    --extra-cflags="-I/usr/local/cuda-12.2/include -I/usr/local/include" \
    --extra-ldflags="-L/usr/local/cuda-12.2/lib64 -L/usr/local/lib -L/usr/lib/aarch64-linux-gnu" \
    --enable-libx264 \
    --enable-libx265 \
    --prefix=/usr/local \
    make -j 12

**3.using**

### Supports Decoding
  - MPEG2
  - H.264/AVC
  - HEVC
  - VP8
  - VP9
  
**example**

    ffmpeg -c:v h264_nvmpi -i input_file -f null -
	
### Supports Encoding
  - H.264/AVC
  - HEVC

**example**

    ffmpeg -i input_file -c:v h264_nvmpi <output.mp4>

### Open CV that works with this 
os.environ["OPENCV_FFMPEG_CAPTURE_OPTIONS"] = "hwaccel;cuvid|video_codec;h264_cuvid|vsync;0"

export OPENCV_TAG=4.10.0
export OPENCV_DIR=./opencv
export OPENCV_BUILD_DIR=${OPENCV_DIR}/build
mkdir -p ${OPENCV_DIR}
cd ${OPENCV_DIR} && git clone --depth 1 --branch ${OPENCV_TAG} https://github.com/opencv/opencv.git
mkdir -p ${OPENCV_BUILD_DIR}
cd ${OPENCV_BUILD_DIR} && cmake ../opencv \
    -D BUILD_LIST=core,gapi,imgcodecs,imgproc,python3,videoio \
    -D BUILD_PERF_TESTS=OFF \
    -D BUILD_TESTS=OFF \
    -D BUILD_TIFF=ON \
    -D PYTHON_DEFAULT_EXECUTABLE=$(which python3) \
    -D WITH_CAP_IOS=OFF \
    -D WITH_EIGEN=OFF \
    -D WITH_FFMPEG=ON \
    -D WITH_GTK=OFF \
    -D WITH_OBSENSOR=OFF \
    -D WITH_OPENCL=ON \
    -D WITH_OPENGL=ON \
    -D WITH_PTHREADS_PF=OFF \
    -D WITH_TBB=OFF \
    -D WITH_VTK=OFF \
    -D WITH_WIN32UI=OFF

cmake --build . -j 12
make install
