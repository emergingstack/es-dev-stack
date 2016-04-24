FROM ubuntu:14.04
MAINTAINER Mike Orzel <mike.orzel@emergingstack.com>

RUN apt-get -y update && apt-get -y install git bc make dpkg-dev libssl-dev && mkdir -p /usr/src/kernels && mkdir -p /opt/nvidia/nvidia_installers

RUN apt-get -y install software-properties-common python-software-properties

# install gcc 4.9 for newer kernels
RUN add-apt-repository ppa:ubuntu-toolchain-r/test
RUN apt-get update
RUN apt-get install -y gcc-4.9 g++-4.9
RUN update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.9 60 --slave /usr/bin/g++ g++ /usr/bin/g++-4.9

ADD http://developer.download.nvidia.com/compute/cuda/7.5/Prod/local_installers/cuda_7.5.18_linux.run /opt/nvidia
#ADD http://developer.download.nvidia.com/compute/cuda/7_0/Prod/local_installers/cuda_7.0.28_linux.run /opt/nvidia/

WORKDIR /usr/src/kernels
RUN git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git linux
WORKDIR linux
RUN git checkout -b stable v`uname -r | sed -e "s/-.*//" | sed -e "s/\.[0]*$//"` && zcat /proc/config.gz > .config && make modules_prepare
RUN sed -i -e "s/`uname -r | sed -e "s/-.*//" | sed -e "s/\.[0]*$//"`/`uname -r`/" include/generated/utsrelease.h # In case a '+' was added

# Nvidia drivers setup
WORKDIR /opt/nvidia/
RUN chmod +x cuda_7.5.18_linux.run && ./cuda_7.5.18_linux.run -extract=`pwd`/nvidia_installers
WORKDIR /opt/nvidia/nvidia_installers

RUN ./NVIDIA-Linux-x86_64-352.39.run -a -x --ui=none
#RUN sed -i "s/read_cr4/__read_cr4/g" NVIDIA-Linux-x86_64-346.46/kernel/nv-pat.c
#RUN sed -i "s/write_cr4/__write_cr4/g" NVIDIA-Linux-x86_64-346.46/kernel/nv-pat.c

COPY nvprocfs.patch /opt/nvidia/nvidia_installers
RUN patch NVIDIA-Linux-x86_64-352.39/kernel/nv-procfs.c < nvprocfs.patch

CMD ./NVIDIA-Linux-x86_64-352.39/nvidia-installer -q -a -n -s --kernel-source-path=/usr/src/kernels/linux/ && insmod /opt/nvidia/nvidia_installers/NVIDIA-Linux-x86_64-352.39/kernel/uvm/nvidia-uvm.ko

