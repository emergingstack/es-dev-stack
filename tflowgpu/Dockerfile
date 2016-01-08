FROM b.gcr.io/tensorflow/tensorflow:latest-gpu
MAINTAINER Mike Orzel <mike.orzel@emergingstack.com>

# Add some dependent packages we will need for the build process
RUN apt-get -y update && apt-get -y install git bc make dpkg-dev && mkdir -p /usr/src/kernels && mkdir -p /opt/nvidia/nvidia_installers

# Download the nvidia cuda package
ADD http://developer.download.nvidia.com/compute/cuda/7_0/Prod/local_installers/cuda_7.0.28_linux.run /opt/nvidia/
RUN chmod +x /opt/nvidia/cuda_7.0.28_linux.run

# download the linux kernel source and prepare it for use
WORKDIR /usr/src/kernels
RUN git clone git://git.kernel.org/pub/scm/linux/kernel/git/stable/linux-stable.git linux
WORKDIR linux
RUN git checkout -b stable v`uname -r` && zcat /proc/config.gz > .config && make modules_prepare

RUN sed -i -e "s/`uname -r`+/`uname -r`/" include/generated/utsrelease.h # In case a '+' was added
RUN sed -i -e "s/`uname -r`+/`uname -r`/" include/config/kernel.release # In case a '+' was added

# Nvidia drivers setup
WORKDIR /opt/nvidia/
RUN chmod +x cuda_7.0.28_linux.run && ./cuda_7.0.28_linux.run -extract=`pwd`/nvidia_installers
WORKDIR /opt/nvidia/nvidia_installers

RUN ./NVIDIA-Linux-x86_64-346.46.run -a -x --ui=none
RUN sed -i "s/read_cr4/__read_cr4/g" NVIDIA-Linux-x86_64-346.46/kernel/nv-pat.c
RUN sed -i "s/write_cr4/__write_cr4/g" NVIDIA-Linux-x86_64-346.46/kernel/nv-pat.c

RUN ./NVIDIA-Linux-x86_64-346.46/nvidia-installer -q -a -n -s --kernel-source-path=/usr/src/kernels/linux/ --no-kernel-module

# install modules to expected location, cuda will do modprobes in certain situations which require this
WORKDIR /usr/src/kernels/linux
RUN make modules && make modules_install
RUN mv /lib/modules/`uname -r`+ /lib/modules/`uname -r`
WORKDIR /opt/nvidia/nvidia_installers
RUN depmod

# Run jupyter notebook and create a folder for the notebooks
RUN chmod +x /run_jupyter.sh
RUN mkdir /examples
WORKDIR /examples
COPY CNN.ipynb /examples/CNN.ipynb
CMD /run_jupyter.sh
