FROM balenalib/genericx86-64-ext-ubuntu:focal

# Ubuntu 20

WORKDIR /usr/src

ENV DEBIAN_FRONTEND noninteractive

# Set variables for the Yocto version of the OS
ENV YOCTO_VERSION=5.15.54
ENV YOCTO_KERNEL=${YOCTO_VERSION}-yocto-standard

# Set variables to download proper NVIDIA driver
ENV NVIDIA_DRIVER_VERSION= 530.41.03
ENV NVIDIA_DRIVER=NVIDIA-Linux-x86_64-${NVIDIA_DRIVER_VERSION}

# Install some prereqs
RUN install_packages git wget unzip build-essential libelf-dev bc libssl-dev bison flex software-properties-common libglvnd-dev

WORKDIR /usr/src/nvidia

# Download and compile NVIDIA driver
RUN \
    curl -fsSL -O https://us.download.nvidia.com/XFree86/Linux-x86_64/$NVIDIA_DRIVER_VERSION/$NVIDIA_DRIVER.run && \
    chmod +x ./${NVIDIA_DRIVER}.run && \
    ./${NVIDIA_DRIVER}.run --extract-only && \
    ./${NVIDIA_DRIVER}/nvidia-installer \
    --ui=none \
    --no-questions \
    --no-drm \
    --no-x-check \
    --no-systemd \
    --no-kernel-module \
    --no-distro-scripts \
    --install-compat32-libs \
    --no-nouveau-check \
    --no-rpms \
    --no-backup \
    --no-abi-note \
    --no-check-for-alternate-installs \
    --no-libglx-indirect \
    --install-libglvnd \
    --x-prefix=/tmp/null \
    --x-module-path=/tmp/null \
    --x-library-path=/tmp/null \
    --x-sysconfig-path=/tmp/null \
    --skip-depmod \
    --expert 
    
WORKDIR /usr/src/app

COPY *.sh ./

CMD ["bash", "start.sh"]
