FROM ubuntu:15.10
MAINTAINER Mike Orzel <mike.orzel@emergingstack.com>

RUN apt-get -y update && apt-get -y install git bc make dpkg-dev gcc-4.9 g++-4.9 kmod && mkdir -p /usr/src/kernels && mkdir -p /opt/nvidia/nvidia_installers

# Use gcc 4.9 to match the version of gcc used to compile the CoreOS stable kernel where the module will be loaded
RUN update-alternatives --remove gcc /usr/bin/gcc-5.2
RUN update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.9 60 --slave /usr/bin/g++ g++ /usr/bin/g++-4.9

ADD http://developer.download.nvidia.com/compute/cuda/7_0/Prod/local_installers/cuda_7.0.28_linux.run /opt/nvidia/

WORKDIR /usr/src/kernels
RUN git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git linux
WORKDIR linux
RUN git checkout -b stable v`uname -r | sed -e "s/-.*//" | sed -e "s/\.[0]*$//"` && zcat /proc/config.gz > .config && make modules_prepare
RUN sed -i -e "s/`uname -r`+/`uname -r`/" include/generated/utsrelease.h # In case a '+' was added

# Nvidia drivers setup
WORKDIR /opt/nvidia/
RUN chmod +x cuda_7.0.28_linux.run && ./cuda_7.0.28_linux.run -extract=`pwd`/nvidia_installers
WORKDIR /opt/nvidia/nvidia_installers

RUN ./NVIDIA-Linux-x86_64-346.46.run -a -x --ui=none
RUN sed -i "s/read_cr4/__read_cr4/g" NVIDIA-Linux-x86_64-346.46/kernel/nv-pat.c
RUN sed -i "s/write_cr4/__write_cr4/g" NVIDIA-Linux-x86_64-346.46/kernel/nv-pat.c

CMD ./NVIDIA-Linux-x86_64-346.46/nvidia-installer -q -a -n -s --kernel-source-path=/usr/src/kernels/linux/ && insmod /opt/nvidia/nvidia_installers/NVIDIA-Linux-x86_64-346.46/kernel/uvm/nvidia-uvm.ko || cat /var/log/nvidia-installer.log

