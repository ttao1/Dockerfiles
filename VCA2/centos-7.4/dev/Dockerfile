
FROM centos:7.4.1708 AS build
WORKDIR /home

# COMMON BUILD TOOLS
RUN yum install -y -q bzip2 make autoconf libtool git wget ca-certificates pkg-config gcc gcc-c++ bison flex patch epel-release yum-devel libcurl-devel zlib-devel;

# Install cmake
ARG CMAKE_VER=3.13.1
ARG CMAKE_REPO=https://cmake.org/files
RUN wget -O - ${CMAKE_REPO}/v${CMAKE_VER%.*}/cmake-${CMAKE_VER}.tar.gz | tar xz && \
    cd cmake-${CMAKE_VER} && \
    ./bootstrap --prefix="/usr" --system-curl && \
    make -j8 && \
    make install

# Install automake, use version 1.14 on CentOS
ARG AUTOMAKE_VER=1.14
ARG AUTOMAKE_REPO=https://ftp.gnu.org/pub/gnu/automake/automake-${AUTOMAKE_VER}.tar.xz
RUN wget -O - ${AUTOMAKE_REPO} | tar xJ && \
    cd automake-${AUTOMAKE_VER} && \
    ./configure --prefix=/usr --libdir=/usr/lib64 --disable-doc && \ 
    make -j8 && \
    make install

# Build NASM
ARG NASM_VER=2.13.03
ARG NASM_REPO=https://www.nasm.us/pub/nasm/releasebuilds/${NASM_VER}/nasm-${NASM_VER}.tar.bz2
RUN  wget ${NASM_REPO} && \
     tar -xaf nasm* && \
     cd nasm-${NASM_VER} && \
     ./autogen.sh && \
     ./configure --prefix="/usr" --libdir=/usr/lib64 && \
     make -j8 && \
     make install

# Build YASM
ARG YASM_VER=1.3.0
ARG YASM_REPO=https://www.tortall.net/projects/yasm/releases/yasm-${YASM_VER}.tar.gz
RUN  wget -O - ${YASM_REPO} | tar xz && \
     cd yasm-${YASM_VER} && \
     sed -i "s/) ytasm.*/)/" Makefile.in && \
     ./configure --prefix="/usr" --libdir=/usr/lib64 && \
     make -j8 && \
     make install

# Build ogg
ARG OGG_VER=1.3.3
ARG OGG_REPO=https://downloads.xiph.org/releases/ogg/libogg-${OGG_VER}.tar.xz

RUN wget -O - ${OGG_REPO} | tar xJ && \
    cd libogg-${OGG_VER} && \
    ./configure --prefix="/usr" --libdir=/usr/lib64 --enable-shared && \
    make -j8 && \
    make install DESTDIR=/home/build && \
    make install

# Build vorbis
ARG VORBIS_VER=1.3.6
ARG VORBIS_REPO=https://downloads.xiph.org/releases/vorbis/libvorbis-${VORBIS_VER}.tar.xz

RUN wget -O - ${VORBIS_REPO} | tar xJ && \
    cd libvorbis-${VORBIS_VER} && \
    ./configure --prefix="/usr" --libdir=/usr/lib64 --enable-shared && \
    make -j8 && \
    make install DESTDIR=/home/build && \
    make install

# Build mp3lame
ARG MP3LAME_VER=3.100
ARG MP3LAME_REPO=https://sourceforge.net/projects/lame/files/lame/${MP3LAME_VER}/lame-${MP3LAME_VER}.tar.gz

RUN wget -O - ${MP3LAME_REPO} | tar xz && \
    cd lame-${MP3LAME_VER} && \
    ./configure --prefix="/usr" --libdir=/usr/lib64 --enable-shared --enable-nasm && \
    make -j8 && \
    make install DESTDIR=/home/build && \
    make install

# Build fdk-aac
ARG FDK_AAC_VER=v0.1.6
ARG FDK_AAC_REPO=https://github.com/mstorsjo/fdk-aac/archive/${FDK_AAC_VER}.tar.gz

RUN wget -O - ${FDK_AAC_REPO} | tar xz && mv fdk-aac-${FDK_AAC_VER#v} fdk-aac && \
    cd fdk-aac && \
    autoreconf -fiv && \
    ./configure --prefix="/usr" --libdir=/usr/lib64 --enable-shared && \
    make -j8 && \
    make install DESTDIR=/home/build && \
    make install


# Build opus
ARG OPUS_VER=1.2.1
ARG OPUS_REPO=https://archive.mozilla.org/pub/opus/opus-${OPUS_VER}.tar.gz

RUN wget -O - ${OPUS_REPO} | tar xz && \
    cd opus-${OPUS_VER} && \
    ./configure --prefix="/usr" --libdir=/usr/lib64 --enable-shared && \
    make -j8 && \
    make install DESTDIR=/home/build && \
    make install

# Build vpx
ARG VPX_VER=tags/v1.7.0
ARG VPX_REPO=https://chromium.googlesource.com/webm/libvpx.git

RUN git clone ${VPX_REPO} && \
    cd libvpx && \
    git checkout ${VPX_VER} && \
    ./configure --prefix="/usr" --libdir=/usr/lib64 --enable-shared --disable-examples --disable-unit-tests --enable-vp9-highbitdepth --as=nasm && \
    make -j8 && \
    make install DESTDIR=/home/build && \
    make install


# Build AOM
ARG AOM_VER=b6f1767eedbaddeb1ff5aa409a710ef61078640e
ARG AOM_REPO=https://aomedia.googlesource.com/aom

RUN  git clone ${AOM_REPO} && \
     mkdir aom/aom_build && \
     cd aom/aom_build && \
     git checkout ${AOM_VER} && \
     cmake -DBUILD_SHARED_LIBS=ON -DENABLE_NASM=ON -DENABLE_TESTS=OFF -DENABLE_DOCS=OFF -DCMAKE_INSTALL_PREFIX="/usr" -DLIB_INSTALL_DIR=/usr/lib64 .. && \
     make -j8 && \
     make install DESTDIR="/home/build" && \
     make install

# Build x264
ARG X264_VER=stable
ARG X264_REPO=https://github.com/mirror/x264

RUN  git clone ${X264_REPO} && \
     cd x264 && \
     git checkout ${X264_VER} && \
     ./configure --prefix="/usr" --libdir=/usr/lib64 --enable-shared && \
     make -j8 && \
     make install DESTDIR="/home/build" && \
     make install


# Build x265
ARG X265_VER=2.9
ARG X265_REPO=https://github.com/videolan/x265/archive/${X265_VER}.tar.gz

RUN  yum install -y -q numactl-devel libpciaccess-devel

RUN  wget -O - ${X265_REPO} | tar xz && mv x265-${X265_VER} x265 && \
     cd x265/build/linux && \
     cmake -DBUILD_SHARED_LIBS=ON -DENABLE_TESTS=OFF -DCMAKE_INSTALL_PREFIX=/usr -DLIB_INSTALL_DIR=/usr/lib64 ../../source && \
     make -j8 && \
     make install DESTDIR="/home/build" && \
     make install

# Fetch SVT-HEVC
ARG SVT_HEVC_VER=v1.4.1
ARG SVT_HEVC_REPO=https://github.com/intel/SVT-HEVC

RUN yum install -y -q patch centos-release-scl && \
    yum install -y -q devtoolset-7

RUN git clone ${SVT_HEVC_REPO} && \
    cd SVT-HEVC/Build/linux && \
    git checkout ${SVT_HEVC_VER} && \
    mkdir -p ../../Bin/Release && \
    ( source /opt/rh/devtoolset-7/enable && \
    cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_INSTALL_LIBDIR=lib64 -DCMAKE_ASM_NASM_COMPILER=yasm ../.. && \
    make -j8 && \
    make install DESTDIR=/home/build && \
    make install )


# Fetch SVT-AV1
ARG SVT_AV1_VER=v0.7.0
ARG SVT_AV1_REPO=https://github.com/OpenVisualCloud/SVT-AV1

RUN git clone ${SVT_AV1_REPO} && \
    cd SVT-AV1/Build/linux && \
    git checkout ${SVT_AV1_VER} && \
    mkdir -p ../../Bin/Release && \
    ( source /opt/rh/devtoolset-7/enable && \
    cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_INSTALL_LIBDIR=lib64 -DCMAKE_ASM_NASM_COMPILER=yasm ../.. && \
    make -j8 && \
    make install DESTDIR=/home/build && \
    make install )

#Remove build residue from SVT-AV1 build -- temp fix for bug
RUN if [ -d "build/home/" ]; then rm -rf build/home/; fi


# Fetch SVT-VP9
ARG SVT_VP9_VER=d18b4acf9139be2e83150e318ffd3dba1c432e74
ARG SVT_VP9_REPO=https://github.com/OpenVisualCloud/SVT-VP9

RUN git clone ${SVT_VP9_REPO} && \
    cd SVT-VP9/Build/linux && \
    git checkout ${SVT_VP9_VER} && \
    mkdir -p ../../Bin/Release && \
  ( source /opt/rh/devtoolset-7/enable && \
    cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_INSTALL_LIBDIR=lib64 -DCMAKE_ASM_NASM_COMPILER=yasm ../.. && \
    make -j8 && \
    make install DESTDIR=/home/build && \
    make install )

#include(transform360.m4)
# Fetch gmmlib
ARG GMMLIB_VER=intel-gmmlib-18.3.0
ARG GMMLIB_REPO=https://github.com/intel/gmmlib/archive/${GMMLIB_VER}.tar.gz

RUN wget -O - ${GMMLIB_REPO} | tar xz && mv gmmlib-${GMMLIB_VER} gmmlib;


#include(libdrm.m4)
# Build libva
ARG LIBVA_VER=2.4.0
ARG LIBVA_REPO=https://github.com/intel/libva/archive/${LIBVA_VER}.tar.gz

RUN yum install -y -q libX11-devel mesa-libGL-devel which libdrm-devel

RUN wget -O - ${LIBVA_REPO} | tar xz && \
    cd libva-${LIBVA_VER} && \
    ./autogen.sh --prefix=/usr --libdir=/usr/lib64 && \
    make -j8 && \
    make install DESTDIR=/home/build && \
    make install;


# Build media driver
ARG MEDIA_DRIVER_VER=intel-media-kbl-19.1.1
ARG MEDIA_DRIVER_REPO=https://github.com/VCDP/media-driver/archive/${MEDIA_DRIVER_VER}.tar.gz

RUN yum install -y -q libX11-devel mesa-libGL-devel libpciaccess-devel libXext-devel

RUN wget -O - ${MEDIA_DRIVER_REPO} | tar xz && mv media-driver-${MEDIA_DRIVER_VER} media-driver && \
    mkdir -p media-driver/build && \
    cd media-driver/build && \
    cmake -DBUILD_TYPE=release -DBUILD_ALONG_WITH_CMRTLIB=1 -DMEDIA_VERSION="2.0.0" -DBS_DIR_GMMLIB=/home/gmmlib/Source/GmmLib -DBS_DIR_COMMON=/home/gmmlib/Source/Common -DBS_DIR_INC=/home/gmmlib/Source/inc -DBS_DIR_MEDIA=/home/media-driver -Wno-dev -DCMAKE_INSTALL_PREFIX=/usr .. && \
    make -j8 && \
    make install DESTDIR=/home/build && \
    make install



# Build Intel(R) Media SDK
ARG MSDK_VER=MSS-KBL-2019-R1-HF1
ARG MSDK_REPO=https://github.com/Intel-Media-SDK/MediaSDK/archive/${MSDK_VER}.tar.gz

RUN wget -O - ${MSDK_REPO} | tar xz && mv MediaSDK-${MSDK_VER} MediaSDK && \
    mkdir -p MediaSDK/build && \
    cd MediaSDK/build && \
    cmake -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_INSTALL_INCLUDEDIR=include -DBUILD_SAMPLES=OFF -DENABLE_OPENCL=OFF -Wno-dev .. && \
    make -j8 && \
    make install DESTDIR=/home/build && \
    make install;

ARG PYTHON_VER=3.6.6
ARG PYTHON_REPO=https://www.python.org/ftp/python/${PYTHON_VER}/Python-${PYTHON_VER}.tgz

RUN yum install -y zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel \
    readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
RUN wget -O - ${PYTHON_REPO} | tar xz && \
    cd Python-${PYTHON_VER} && \
    ./configure --prefix=/usr && \
    make && \
    make install && \
    make install DESTDIR=/home/build
RUN yum install -y -q python-yaml



ARG OPENCV_VER=4.1.0
ARG OPENCV_REPO=https://github.com/opencv/opencv/archive/${OPENCV_VER}.tar.gz


RUN wget ${OPENCV_REPO} && \
    tar -zxvf ${OPENCV_VER}.tar.gz && \
    cd opencv-${OPENCV_VER} && \
    mkdir build && \
    cd build && \
    cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr/local -D BUILD_EXAMPLES=OFF -D BUILD_PERF_TESTS=OFF -D BUILD_DOCS=OFF -D BUILD_TESTS=OFF .. && \
    make -j $(nproc) && \
    make install DESTDIR=/home/build && \
    make install

# Build DLDT-Inference Engine
ARG DLDT_VER=2019_R3
ARG DLDT_REPO=https://github.com/opencv/dldt.git

RUN yum install -y -q boost-devel glibc-static glibc-devel libstdc++-static libstdc++-devel libstdc++ libgcc libusbx-devel openblas-devel;

RUN git clone -b ${DLDT_VER} ${DLDT_REPO} && \
    cd dldt && \
    git submodule init && \
    git submodule update --recursive && \
    cd inference-engine && \
    mkdir build && \
    cd build && \
    cmake  -DCMAKE_INSTALL_PREFIX=/opt/intel/dldt -DLIB_INSTALL_PATH=/opt/intel/dldt -DENABLE_MKL_DNN=ON -DENABLE_CLDNN=ON -DENABLE_SAMPLES=OFF -DENABLE_OPENCV=OFF .. && \
    make -j $(nproc) && \
    rm -rf ../bin/intel64/Release/lib/libgtest* && \
    rm -rf ../bin/intel64/Release/lib/libgmock* && \
    rm -rf ../bin/intel64/Release/lib/libmock* && \
    rm -rf ../bin/intel64/Release/lib/libtest*

ARG libdir=/opt/intel/dldt/inference-engine/lib/intel64

RUN mkdir -p /opt/intel/dldt/inference-engine/include && \
    cp -r dldt/inference-engine/include/* /opt/intel/dldt/inference-engine/include && \
    mkdir -p ${libdir} && \
    cp -r dldt/inference-engine/bin/intel64/Release/lib/* ${libdir} && \
    mkdir -p /opt/intel/dldt/inference-engine/src && \
    cp -r dldt/inference-engine/src/* /opt/intel/dldt/inference-engine/src/ && \
    mkdir -p /opt/intel/dldt/inference-engine/share && \
    cp -r dldt/inference-engine/build/share/* /opt/intel/dldt/inference-engine/share/ && \
    mkdir -p /opt/intel/dldt/inference-engine/external/ && \
    cp -r dldt/inference-engine/temp/tbb /opt/intel/dldt/inference-engine/external/

RUN mkdir -p build/opt/intel/dldt/inference-engine/include && \
    cp -r dldt/inference-engine/include/* build/opt/intel/dldt/inference-engine/include && \
    mkdir -p build${libdir} && \
    cp -r dldt/inference-engine/bin/intel64/Release/lib/* build${libdir} && \
    mkdir -p build/opt/intel/dldt/inference-engine/src && \
    cp -r dldt/inference-engine/src/* build/opt/intel/dldt/inference-engine/src/ && \
    mkdir -p build/opt/intel/dldt/inference-engine/share && \
    cp -r dldt/inference-engine/build/share/* build/opt/intel/dldt/inference-engine/share/ && \
    mkdir -p build/opt/intel/dldt/inference-engine/external/ && \
    cp -r dldt/inference-engine/temp/tbb build/opt/intel/dldt/inference-engine/external/

RUN for p in /usr /home/build/usr /opt/intel/dldt/inference-engine /home/build/opt/intel/dldt/inference-engine; do \
        pkgconfiglibdir="$p/lib64" && \
        mkdir -p "${pkgconfiglibdir}/pkgconfig" && \
        pc="${pkgconfiglibdir}/pkgconfig/dldt.pc" && \
        echo "prefix=/opt" > "$pc" && \
        echo "libdir=${libdir}" >> "$pc" && \
        echo "includedir=/opt/intel/dldt/inference-engine/include" >> "$pc" && \
        echo "" >> "$pc" && \
        echo "Name: DLDT" >> "$pc" && \
        echo "Description: Intel Deep Learning Deployment Toolkit" >> "$pc" && \
        echo "Version: 5.0" >> "$pc" && \
        echo "" >> "$pc" && \
        echo "Libs: -L\${libdir} -linference_engine" >> "$pc" && \
        echo "Cflags: -I\${includedir}" >> "$pc"; \
    done;

ENV InferenceEngine_DIR=/opt/intel/dldt/inference-engine/share
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/intel/dldt/inference-engine/lib:/opt/intel/dldt/inference-engine/external/tbb/lib:${libdir}

# DLDT IE C API
ARG DLDT_C_API_REPO=https://raw.githubusercontent.com/VCDP/FFmpeg-patch/master/thirdparty/dldt-c-api/source/dldt-c_api_v2-1.0.tar.gz

ARG c_api_libdir="/usr/local/lib64"
RUN wget -O - ${DLDT_C_API_REPO} | tar xz && \
    cd dldt-c_api-1.0 && \
    mkdir -p build && cd build && \
    cmake -DENABLE_AVX512F=OFF .. && \
    make -j8 && \
    make install && \
    make install DESTDIR=/home/build
ENV PKG_CONFIG_PATH=${c_api_libdir}/pkgconfig:$PKG_CONFIG_PATH

#install Model Optimizer in the DLDT for Dev
ARG PYTHON_TRUSTED_HOST
ARG PYTHON_TRUSTED_INDEX_URL
#install MO dependencies
#RUN pip3 install numpy scipy
RUN git clone https://github.com/google/protobuf.git && \
    cd protobuf && \
    git submodule update --init --recursive && \
    ./autogen.sh && \
    ./configure && \
    make && \
    make install && \
    make install DESTDIR=/home/build
#RUN apt-get update && apt-get install -y sudo
#installing dependency libs to mo_libs directory to avoid issues with updates to Python version
RUN cd dldt/model-optimizer && \
if [ "x$PYTHON_TRUSTED_HOST" = "x" ] ; \
then pip3 install --target=/home/build/mo_libs -r requirements.txt && \
pip3 install -r requirements.txt; \
else pip3 install --target=/home/build/mo_libs -r requirements.txt -i $PYTHON_TRUSTED_INDEX_URL --trusted-host $PYTHON_TRUSTED_HOST && \
pip3 install -r requirements.txt -i $PYTHON_TRUSTED_INDEX_URL --trusted-host $PYTHON_TRUSTED_HOST; \
fi

#Copy over Model Optimizer to same directory as Inference Engine
RUN cp -r dldt/model-optimizer /opt/intel/dldt/model-optimizer
RUN cp -r dldt/model-optimizer /home/build/opt/intel/dldt/model-optimizer



# Build the gstremaer core
ARG GST_VER=1.16.0
ARG GST_REPO=https://gstreamer.freedesktop.org/src/gstreamer/gstreamer-${GST_VER}.tar.xz

RUN  yum install -y -q glib2-devel-2.56.1 gettext-devel gobject-introspection gobject-introspection-devel python-gobject-base
RUN  wget -O - ${GST_REPO} | tar xJ && \
     cd gstreamer-${GST_VER} && \
     ./autogen.sh \
        --prefix=/usr \
        --libdir=/usr/lib64 \
        --libexecdir=/usr/lib64 \
        --enable-shared \
        --enable-introspection \
        --disable-examples  \
        --disable-gtk-doc && \
     make -j $(nproc) && \
     make install DESTDIR=/home/build && \
     make install;

# Build the gstremaer plugin base
ARG GST_PLUGIN_BASE_REPO=https://gstreamer.freedesktop.org/src/gst-plugins-base/gst-plugins-base-${GST_VER}.tar.xz

RUN  yum install -y -q libXv-devel libvisual-devel libtheora-devel glib2-devel alsa-lib-devel cdparanoia-devel mesa-libGL-devel pango-devel

RUN  yum install -y -q libXrandr-devel


RUN  wget -O - ${GST_PLUGIN_BASE_REPO} | tar xJ && \
     cd gst-plugins-base-${GST_VER} && \
     ./autogen.sh \
        --prefix=/usr \
        --libdir=/usr/lib64 \
        --libexecdir=/usr/lib64 \
        --enable-introspection \
        --enable-shared \
        --disable-examples  \
        --disable-gtk-doc && \
     make -j $(nproc) && \
     make install DESTDIR=/home/build && \
     make install


# Build the gstremaer plugin good set
ARG GST_PLUGIN_GOOD_REPO=https://gstreamer.freedesktop.org/src/gst-plugins-good/gst-plugins-good-${GST_VER}.tar.xz

RUN  yum install -y -q libsoup-devel libjpeg-devel

RUN  wget -O - ${GST_PLUGIN_GOOD_REPO} | tar xJ && \
     cd gst-plugins-good-${GST_VER} && \
     ./autogen.sh \
        --prefix=/usr \
        --libdir=/usr/lib64 \
        --libexecdir=/usr/lib64 \
        --enable-shared \
        --disable-examples  \
        --disable-gtk-doc && \
     make -j $(nproc) && \
     make install DESTDIR=/home/build && \
     make install


# Build the gstreamer plugin bad set
ARG GST_PLUGIN_BAD_REPO=https://gstreamer.freedesktop.org/src/gst-plugins-bad/gst-plugins-bad-${GST_VER}.tar.xz
ARG GST_PATCHES_RELEASE_VER=v0.1
ARG GST_PATCHES_RELEASE_URL=https://github.com/VCDP/gstreamer-patch/archive/${GST_PATCHES_RELEASE_VER}.tar.gz
RUN wget -O - ${GST_PATCHES_RELEASE_URL} | tar xz

RUN  yum localinstall -y --nogpgcheck https://download1.rpmfusion.org/free/el/rpmfusion-free-release-7.noarch.rpm && yum install -y -q openssl-devel librtmp-devel && yum remove -y rpmfusion-free-release

RUN  wget -O - ${GST_PLUGIN_BAD_REPO} | tar xJ && \
     cd gst-plugins-bad-${GST_VER} && \
     find /home/gstreamer-patch-${GST_PATCHES_RELEASE_VER#*v}/ -type f -name '*.patch' -print0 | sort -z | xargs -t -0 -n 1 patch -p1 -i && \
     ./autogen.sh \
        --prefix=/usr \
        --libdir=/usr/lib64 \
        --libexecdir=/usr/lib64 \
        --enable-shared \
        --disable-examples  \
        --disable-gtk-doc \ 
        --disable-shm \
        --disable-mxf && \
     make -j $(nproc) && \
     make install DESTDIR=/home/build && \
     make install

# Build the gstremaer plugin ugly set
ARG GST_PLUGIN_UGLY_REPO=https://gstreamer.freedesktop.org/src/gst-plugins-ugly/gst-plugins-ugly-${GST_VER}.tar.xz

RUN  wget -O - ${GST_PLUGIN_UGLY_REPO} | tar xJ; \
     cd gst-plugins-ugly-${GST_VER}; \
     ./autogen.sh \
        --prefix=/usr \
        --libdir=/usr/lib64 \
        --libexecdir=/usr/lib64 \
        --enable-shared \
        --disable-examples  \
        --disable-gtk-doc && \
     make -j $(nproc) && \
     make install DESTDIR=/home/build && \
     make install

# Build gst-libav
ARG GST_PLUGIN_LIBAV_REPO=https://gstreamer.freedesktop.org/src/gst-libav/gst-libav-${GST_VER}.tar.xz

RUN yum install -y -q zlib-devel openssl-devel

RUN wget -O - ${GST_PLUGIN_LIBAV_REPO} | tar xJ && \
    cd gst-libav-${GST_VER} && \
    ./autogen.sh \
        --prefix="/usr" \
        --libdir=/usr/lib64 \
        --enable-shared \
        --enable-gpl \
        --disable-gtk-doc && \
    make -j $(nproc) && \
    make install DESTDIR=/home/build && \
    make install


# Build gstremaer plugin for svt

#RUN cd SVT-HEVC/gstreamer-plugin && \
#    cmake . && \
#    make -j8 && \
#    make install DESTDIR=/home/build && \
#    make install

RUN cd SVT-VP9/gstreamer-plugin && \
    cmake . && \
    make -j8 && \
    make install DESTDIR=/home/build && \
    make install

RUN cd SVT-AV1/gstreamer-plugin && \
    cmake . && \
    make -j8 && \
    make install DESTDIR=/home/build && \
    make install

# Build gstremaer plugin vaapi
ARG GST_PLUGIN_VAAPI_REPO=https://gstreamer.freedesktop.org/src/gstreamer-vaapi/gstreamer-vaapi-${GST_VER}.tar.xz

# https://gitlab.freedesktop.org/gstreamer/gstreamer-vaapi/merge_requests/45
ARG GST_PLUGIN_VAAPI_REPO_DISPLAY_LOCK_PATCH_HASH=b219f6095f3014041896714dd88e7d90ee3d72dd
ARG GST_PLUGIN_VAAPI_REPO_GIT=https://gitlab.freedesktop.org/gstreamer/gstreamer-vaapi.git

RUN  yum install -y -q libXrandr-devel

#RUN  git clone https://gitlab.freedesktop.org/gstreamer/gstreamer-vaapi.git -b 1.14 --depth 10 && \
#     cd gstreamer-vaapi && git reset --hard ${GST_PLUGIN_VAAPI_REPO_DISPLAY_LOCK_PATCH_HASH} && \
RUN  wget -O - ${GST_PLUGIN_VAAPI_REPO} | tar xJ && \
     cd gstreamer-vaapi-${GST_VER} && \
     ./autogen.sh \
        --prefix=/usr \
        --libdir=/usr/lib64 \
        --libexecdir=/usr/lib64 \
        --enable-shared \
        --disable-examples \
        --disable-gtk-doc  && \
     make -j $(nproc) && \
     make install DESTDIR=/home/build && \
     make install


# Fetch FFmpeg source
ARG FFMPEG_VER=n4.1
ARG FFMPEG_REPO=https://github.com/FFmpeg/FFmpeg/archive/${FFMPEG_VER}.tar.gz
ARG FFMPEG_1TN_PATCH_REPO=https://patchwork.ffmpeg.org/patch/11625/raw
ARG FFMPEG_THREAD_PATCH_REPO=https://patchwork.ffmpeg.org/patch/11035/raw

ARG FFMPEG_PATCHES_RELEASE_VER=0.1
ARG FFMPEG_PATCHES_RELEASE_URL=https://github.com/VCDP/CDN/archive/v${FFMPEG_PATCHES_RELEASE_VER}.tar.gz
ARG FFMPEG_PATCHES_PATH=/home/CDN-${FFMPEG_PATCHES_RELEASE_VER}
RUN wget -O - ${FFMPEG_PATCHES_RELEASE_URL} | tar xz

ARG FFMPEG_MA_RELEASE_VER=0.3
ARG FFMPEG_MA_RELEASE_URL=https://github.com/VCDP/FFmpeg-patch/archive/v${FFMPEG_MA_RELEASE_VER}.tar.gz
ARG FFMPEG_MA_PATH=/home/FFmpeg-patch-${FFMPEG_MA_RELEASE_VER}
RUN wget -O - ${FFMPEG_MA_RELEASE_URL} | tar xz

RUN yum install -y -q libass-devel freetype-devel SDL2-devel libxcb-devel libvdpau-devel texinfo zlib-devel openssl-devel

RUN wget -O - ${FFMPEG_REPO} | tar xz && mv FFmpeg-${FFMPEG_VER} FFmpeg && \
    cd FFmpeg && \
    find ${FFMPEG_PATCHES_PATH}/FFmpeg_patches -type f -name '0001*.patch' -print0 | sort -z | xargs -t -0 -n 1 patch -p1 -i && \
    wget -O - ${FFMPEG_1TN_PATCH_REPO} | patch -p1 && \
    wget -O - ${FFMPEG_THREAD_PATCH_REPO} | patch -p1 && \
    find ${FFMPEG_MA_PATH}/patches -type f -name '*.patch' -print0 | sort -z | xargs -t -0 -n 1 patch -p1 -i;

# Patch FFmpeg source for SVT-HEVC
RUN cd /home/FFmpeg && \
    patch -p1 < ../SVT-HEVC/ffmpeg_plugin/0001-lavc-svt_hevc-add-libsvt-hevc-encoder-wrapper.patch;

# Patch FFmpeg source for SVT-AV1
RUN cd /home/FFmpeg; \
    patch -p1 < ../SVT-AV1/ffmpeg_plugin/0001-Add-ability-for-ffmpeg-to-run-svt-av1-with-svt-hevc.patch;

# Compile FFmpeg
RUN cd /home/FFmpeg && \
    ./configure --prefix="/usr" --libdir=/usr/lib64 --extra-libs="-lpthread -lm" --enable-shared --enable-gpl --enable-libass --enable-libfreetype  --enable-openssl --enable-nonfree --enable-libdrm --enable-libmfx  --enable-libfdk-aac --enable-libmp3lame --enable-libopus --enable-libvorbis --enable-libvpx --enable-libx264 --enable-libx265 --enable-libaom --enable-libsvthevc --enable-libsvtav1 --enable-libinference_engine_c_api  && \
    make -j8 && \
    make install DESTDIR="/home/build"



FROM centos:7.4.1708
LABEL Description="This is the image for FFMPEG and GSTREAMER application development on CentOS 7.4"
LABEL Vendor="Intel Corporation"
WORKDIR /home

# Prerequisites
RUN yum install -y epel-release;  \
    yum localinstall -y --nogpgcheck https://download1.rpmfusion.org/free/el/rpmfusion-free-release-7.noarch.rpm; \
    yum install -y -q   libxcb SDL2 libass numactl libvdpau  glib2-2.56.1 pango gobject-introspection libxcb mesa-libGL libXrandr libpng12 libXv libvisual mesa-libGL pango glib2 alsa-lib cdparanoia libtheora libpciaccess libX11 mesa-dri-drivers mesa-libGL libdrm  libsoup libjpeg-turbo librtmp python-yaml ; \
    yum remove -y -q epel-release; \
    yum remove -y -q rpmfusion-free-release; \
    rm -rf /var/cache/yum/*;

# Install
COPY --from=build /home/build /
ENV LIBVA_DRIVERS_PATH=/usr/lib64/dri
ENV LIBVA_DRIVER_NAME=iHD
ENV GST_VAAPI_ALL_DRIVERS=1
ENV DISPLAY=:0.0
ARG libdir=/opt/intel/dldt/inference-engine/lib/intel64
ARG c_api_libdir="/usr/local/lib64"
ENV LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/intel/dldt/inference-engine/lib:/opt/intel/dldt/inference-engine/external/tbb/lib:${libdir}:${c_api_libdir}
ENV PKG_CONFIG_PATH=${c_api_libdir}/pkgconfig:$PKG_CONFIG_PATH
ENV InferenceEngine_DIR=/opt/intel/dldt/inference-engine/shareENV PYTHONPATH=${PYTHONPATH}:/mo_libs

