FROM b.gcr.io/tensorflow/tensorflow:latest-gpu
MAINTAINER Mike Orzel <mike.orzel@emergingstack.com>

ENV LD_LIBRARY_PATH /usr/lib/x86_64-linux-gnu:/usr/local/nvidia/lib:/usr/local/nvidia/lib64:/usr/local/cuda/lib:/usr/local/cuda/lib64:
RUN ln -s /usr/lib/x86_64-linux-gnu/libcudnn.so.4 /usr/lib/x86_64-linux-gnu/libcudnn.so

# Add some dependent packages we will need for the build process
RUN apt-get -y update && apt-get -y install git bc make dpkg-dev libssl-dev && mkdir -p /usr/src/kernels && mkdir -p /opt/nvidia/nvidia_installers

RUN apt-get -y install software-properties-common python-software-properties

# install gcc 4.9 for newer kernels
RUN add-apt-repository ppa:ubuntu-toolchain-r/test
RUN apt-get update
RUN apt-get install -y gcc-4.9 g++-4.9
RUN update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.9 60 --slave /usr/bin/g++ g++ /usr/bin/g++-4.9

# Download the nvidia cuda package
ADD http://developer.download.nvidia.com/compute/cuda/7.5/Prod/local_installers/cuda_7.5.18_linux.run /opt/nvidia
RUN chmod +x /opt/nvidia/cuda_7.5.18_linux.run

# download the linux kernel source and prepare it for use
WORKDIR /usr/src/kernels
RUN git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git linux
WORKDIR linux

RUN git checkout -b stable v`uname -r | sed -e "s/-.*//" | sed -e "s/\.[0]*$//"` && zcat /proc/config.gz > .config && make modules_prepare
RUN sed -i -e "s/`uname -r | sed -e "s/-.*//" | sed -e "s/\.[0]*$//"`/`uname -r`/" include/generated/utsrelease.h # In case a '+' was added
RUN sed -i -e "s/`uname -r | sed -e "s/-.*//" | sed -e "s/\.[0]*$//"`/`uname -r`/" include/config/kernel.release # In case a '+' was added

# Nvidia drivers setup
WORKDIR /opt/nvidia/
RUN chmod +x cuda_7.5.18_linux.run && ./cuda_7.5.18_linux.run -extract=`pwd`/nvidia_installers
WORKDIR /opt/nvidia/nvidia_installers

RUN ./NVIDIA-Linux-x86_64-352.39.run -a -x --ui=none
# Patch code to be compatible with current coreos kernel
COPY nvprocfs.patch /opt/nvidia/nvidia_installers
RUN patch NVIDIA-Linux-x86_64-352.39/kernel/nv-procfs.c < nvprocfs.patch

RUN ./NVIDIA-Linux-x86_64-352.39/nvidia-installer -q -a -n -s --kernel-source-path=/usr/src/kernels/linux/ --no-kernel-module

# Run jupyter notebook and create a folder for the notebooks
RUN chmod +x /run_jupyter.sh
RUN mkdir /examples
WORKDIR /examples
COPY CNN.ipynb /examples/CNN.ipynb
CMD /run_jupyter.sh
