# 5. Runtime demonstration
You can use microUSB or Telnet to connect to the console on the ZCU104. The console via microUSB UART has the (dis)advantage of containing the kernel output. This can be impractical as the NVDLA driver generates a lot of messages.

> **_NOTE:_** Default username: `root` and password: `root`.

## 5.1 Loading Module
First, we need to load the kernel module with the NVDLA driver.
```
root@smalldla_zcu104:~# insmod /lib/modules/4.19.0-xilinx-v2019.1/extra/opendla.ko 
```

After that we can run nvdla_runtime with our NVDLA Loadable.
```
root@smalldla_zcu104:~# ./nvdla_runtime --loadable ~/fast-math.nvdla --image ~/nvdla_loadables/lenet-mnist-caffe/Images/0_1.jpg --rawdump
creating new runtime context...
Emulator starting
dlaimg height: 28 x 28 x 1: LS: 128 SS: 0 Size: 3584
(DLA_TEST) Error 0x00000004: Mismatched channel: 1 != 4 (in TestUtils.cpp, function createImageCopy(), line 160)
submitting tasks...
Work Found!
Work Done
execution time = 2301063.000000 s
Shutdown signal received, exiting
Test pass
root@smalldla_zcu104:~# cat output.dimg 
0 124 0 0 0 0 0 0 0 0
root@smalldla_zcu104:~# ./nvdla_runtime --loadable ~/fast-math.nvdla --image ~/nvdla_loadables/lenet-mnist-caffe/Images/0_9.jpg --rawdump
creating new runtime context...
Emulator starting
dlaimg height: 28 x 28 x 1: LS: 128 SS: 0 Size: 3584
(DLA_TEST) Error 0x00000004: Mismatched channel: 1 != 4 (in TestUtils.cpp, function createImageCopy(), line 160)
submitting tasks...
Work Found!
Work Done
execution time = 2296279.000000 s
Shutdown signal received, exiting
Test pass
root@smalldla_zcu104:~# cat output.dimg 
0 0 0 0 0 0 0 0 0 124
```

There is still the same mismatched channel error as at compile time, but it does not affect network output.

> **_NOTE:_** For some reason, nvdla_runtime says the execution time is in seconds, but it is actually in microseconds. So it runs about 2.3 seconds for an image.