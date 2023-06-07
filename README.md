# NVDLA Implementation On ZCU104

## About
This repository serves as a support for my thesis on the implementation of NVDLA, an open source convolutional neural network accelerator from Nvidia, on FPGA. The repository contains a tutorial with the implementation procedure and some of my notes about the implementation.

This project is heavily inspired by Lei Wang's [NVDLA Xilinx FPGA Mapping](https://leiblog.wang/NVDLA-Xilinx-FPGA-Mapping/) project. I'm also using some of his code in this project.

Created at Faculty of Information Technology, Brno University of Technology.

![Faculty of Information Technology, Brno University of Technology logo](images/fit_color.png)

## Content

1. [Introduction](introduction.md)
2. [Hardware Design Guide](hardware.md)
3. [Software Design Guide](software.md)
4. [NVDLA Loadable](loadable.md)
5. [Runtime demonstration](runtime.md)

## File Tree

```
images/     - Directory with all images used in the documentation
nvdla_hw/   - Submodule with forked NVDLA HW
nvdla_ip/   - NVDLA IP core for Vivado
nvdla_sw/   - Submodule with forked NVDLA SW
petalinux/  - OpenDLA module and kernel config
rtl/        - Generated RTL output from NVDLA HW with wrapper from Lei Wang's project
```

## The Following Possible Developments
- Patch NVDLA driver for newer kernels. There is a NVDLA SW [fork](https://github.com/ucb-bar/nvdla-sw) which should support Linux kernel 5.7.
- Try to find another compiler and use it. ~~There is open source [ONNC](https://github.com/ONNC/onnc) compiler which should support compiling ONNX NN for NVDLA. Only the [commercial version](https://github.com/nvdla/sw/issues/217#issuecomment-863889867) support INT8.~~
- Try to run [Tengine](https://github.com/OAID/Tengine).
- Try to use anoter open source CNN accelerator. PipeCNN, Gemmini, TVM?
- Find out how NVDLA Loadables works.

## Reference

<http://nvdla.org/>

<https://github.com/nvdla/hw>

<https://github.com/nvdla/sw>

<https://github.com/LeiWang1999/ZYNQ-NVDLA>

<https://leiblog.wang/NVDLA-Xilinx-FPGA-Mapping/>

<https://docs.xilinx.com/v/u/2019.1-English/ug1144-petalinux-tools-reference-guide>


## Another NVDLA Resources

<https://charleshong3.github.io/projects/nvdla_v_gemmini.pdf>

<https://www.eit.lth.se/sprapport.php?uid=1186>

## Another Interesting CNN FPGA Projects
<https://github.com/LCAI-TIHU/SW>

<https://github.com/doonny/PipeCNN>

<https://github.com/UCLA-VAST/FlexCNN>

<https://tvm.apache.org/>

<https://github.com/SameLight/ITRI-OpenDLA>

<https://github.com/ucb-bar/gemmini>