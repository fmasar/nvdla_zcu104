# 1. Project Introduction
## 1.1 Abstract
This project is about the implementation of NVDLA on an FPGA, more precisely on the ZCU104. NVDLA is an open source hardware accelerator for convolutional neural networks (CNN) from Nvidia. It is a general-purpose accelerator that is capable of running neural networks saved in the nvdla loadable format.

> **_NOTE:_** Development of NVDLA has probably been discontinued. There is no official information about the cancellation of the project, but the last change to the NVDLA repository was on 30 September 2019. There is also no release with promising ONNC support for the compiler, so only Caffe is supported.

## 1.2 Used Hardware
- [Zynq UltraScale+ MPSoC ZCU104 Evaluation Kit](https://www.xilinx.com/products/boards-and-kits/zcu104.html)
- an SD card (16 GB)
- Computer with Nvidia GPU with CUDA support (GTX 1050 Ti)

## 1.3 Used Software
### 1.3.1 For Hardware Design
The RTL generation and hardware design in the Vivado:
- Manjaro Linux x86_64
- Vivado 2022.1

Installed software required by RTL generation:
```
cpp/gcc/g++  - gcc (GCC) 12.2.1 20230201
perl         - revision 5 version 36 subversion 0
java         - openjdk 20.0.1 2023-04-18
python       - Python 3.10.10
clang        - version 15.0.7
```

### 1.3.2 For Software Design
NVDLA SW and PetaLinux compilation:
- Ubuntu 18.04.6
- PetaLinux 2019.1

The output of the command `apt list --installed` with all installed packages is available [here](/ubuntu18_04_installed_packages.txt).

Installed packages required by PetaLinux:
```
gcc              - 4:7.4.0-1ubuntu2.3
gcc-multilib     - 4:7.4.0-1ubuntu2.3
gawk             - 1:4.1.4+dfsg-1build1
make             - 4.1-9.1ubuntu1
git              - 1:2.17.1-1ubuntu0.18
socat            - 1.7.3.2-2ubuntu2
xterm            - 330-1ubuntu2.2
autoconf         - 2.69-11
build-essential  - 12.4ubuntu1
zlib1g-dev       - 1:1.2.11.dfsg-0ubuntu2.2
python           - 2.7.15~rc1-1
ncurses-bin      - 6.1-1ubuntu1.18.04.1
texinfo          - 6.5.0.dfsg.1-2
net-tools        - 1.60+git20161116.90da8a0-1ubuntu1
chrpath          - 0.16-2
libtool          - 2.4.6-2
zlib1g-dev       - 1:1.2.11.dfsg-0ubuntu2.2
ncurses-term     - 6.1-1ubuntu1.18.04.1
libncurses5-dev  - 6.1-1ubuntu1.18.04.1
```

Packages for UMD compilation:
```
g++-aarch64-linux-gnu       - 4:7.4.0-1ubuntu2.3
g++-7-aarch64-linux-gnu     - 7.5.0-3ubuntu1~18.04cross1
gcc-aarch64-linux-gnu       - 4:7.4.0-1ubuntu2.3
gcc-7-aarch64-linux-gnu     - 7.5.0-3ubuntu1~18.04cross1
binutils-aarch64-linux-gnu  - 2.30-21ubuntu1~18.04.9
```

> **_NOTE:_** The first four packages are incompatible with the `gcc-multilib` package required by PetaLinux. These packages are not listed in the package summary mentioned above.

### 1.3.3 For TensorRT
- Nvidia Driver Version: 530.41.03
- CUDA Version: 12.1
- Docker version: 23.0.4
- nvidia-docker package: 2.13.0-1 - [from AUR](https://aur.archlinux.org/packages/nvidia-docker) 

## 1.4 Limitations
### 1.4.1 Hardware Limitations
The RTL for NVDLA was generate using the with [nv_small.spec](https://github.com/fmasar/nvdla_hw/blob/nv_small/spec/defs/nv_small.spec) specification, hence there are the relevant [restrictions](http://nvdla.org/hw/v1/hwarch.html#small-nvdla-implementation-example).

### 1.4.2 Software Limitations
The NVDLA KMD part, which contains the Linux kernel module, is only compatible (after a small modification) with kernels up to 4.19. This means that you cannot use newer PetaLinux than 2019. It also means that you can't use the official Ubuntu release for ZCU104, because the oldest version is Ubuntu 20.04 LTS arm64 with kernel 5.4. However, you can use Linux kernel from PetaLinux and (somehow) replace the original root filesystem with an older version of Ubuntu. I have not tried this so there is no how to guide in this documentation.

Limitations/supported features for NVDLA compiler are available [here](https://github.com/fmasar/nvdla_sw/blob/master/CompilerFeatures.md).
