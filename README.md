# es-dev-stack
An on-premises, bare-metal solution for deploying GPU-powered applications in containers

**Blog Post with deployment details:**

http://www.emergingstack.com/2016/01/10/Nvidia-GPU-plus-CoreOS-plus-Docker-plus-TensorFlow.html 

### Prerequisites

- CoreOS-compatible dedicated machine with vanilla CoreOS installed
- Current-generation Nvidia GPU (tested with TitanX)

### To Build

**Nvidia Drivers Installation Image**

```
$ cd es-dev-stack/corenvidiadrivers
```

```
$ docker build -t cuda .
```

**GPU-enabled TensorFlow Image**

```
$ cd es-dev-stack/tflowgpu
```

```
$ docker build -t tflowgpu .
```

### To Run

**Stage 1 - Install Nvidia Drivers & Register GPU Devices (One-Time)**

```
# docker run -it --privileged cuda
```

```
# ./mkdevs.sh
```

**Stage 2 - TensorFlow Docker Container with mapped GPU devices**

```
$ docker run --device /dev/nvidia0:/dev/nvidia0 --device /dev/nvidia1:/dev/nvidia1 --device /dev/nvidiactl:/dev/nvidiactl --device /dev/nvidia-uvm:/dev/nvidia-uvm -it -p 8888:8888 --privileged tflowgpu
```
