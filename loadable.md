# 4. NVDLA Loadable

NVDLA has its own compiled neural network format called NVDLA Loadable. It can be created using `nvdla_compiler`, which is capable of compiling Caffe neural networks with some [limitations](https://github.com/fmasar/nvdla_sw/blob/master/CompilerFeatures.md).

> **_NOTE:_** For precompiled NVDLA Loadables, you can refer to [Lei Wang's repository](https://github.com/LeiWang1999/nvdla_loadables).

## 4.1 INT8 Quantization
Most neural network models use floating point arithmetic, but the nv_small configuration of NVDLA only supports fixed point arithmetic, specifically int8. So we need to generate a scales factors for nvdla_compiler.

> **_NOTE:_** See [Low precision support in NVDLA](https://github.com/fmasar/nvdla_sw/blob/master/LowPrecision.md) for more details.

The official way to generate scales factors is with [TensorRT](https://github.com/NVIDIA/TensorRT/tree/release/5.1/samples/opensource/sampleINT8). For some reason there is no long example how to generate scales factors with Python in the latest version of TensorRT. But there is a docker with older TensorRT version (7.2.2) that we can use.

> **_NOTE:_** For some reason there are no Python examples on the TensorRT Github branch with version 7.2, so I will refer to the branch with version 8.0.

### 4.1.1 Getting Docker Container
Pull Docker container and run it.
```
docker pull nvcr.io/nvidia/tensorrt:20.12-py3
docker run -ti --gpus all nvcr.io/nvidia/tensorrt:20.12-py3 /bin/bash
```

### 4.1.2 Getting MNIST Database
In the docker container, download and extract the MNIST database. We will use it to run LeNet and read the weight value for quantization.
```
wget -O /usr/src/tensorrt/data/mnist/train-images-idx3-ubyte.gz http://yann.lecun.com/exdb/mnist/train-images-idx3-ubyte.gz
wget -O /usr/src/tensorrt/data/mnist/t10k-images-idx3-ubyte.gz http://yann.lecun.com/exdb/mnist/t10k-images-idx3-ubyte.gz
wget -O /usr/src/tensorrt/data/mnist/t10k-labels-idx1-ubyte.gz http://yann.lecun.com/exdb/mnist/t10k-labels-idx1-ubyte.gz

gzip -d /usr/src/tensorrt/data/mnist/train-images-idx3-ubyte.gz
gzip -d /usr/src/tensorrt/data/mnist/t10k-images-idx3-ubyte.gz
gzip -d /usr/src/tensorrt/data/mnist/t10k-labels-idx1-ubyte.gz
```

### 4.1.3 Getting LeNet Model
You can train your own LeNet model, but for testing purposes it is much faster to use a pre-trained model instead of learning and running Caffe. I chose a model from [Lei Wang's repository](https://github.com/LeiWang1999/nvdla_loadables).
```
git clone https://github.com/LeiWang1999/nvdla_loadables
cp ~/nvdla_loadables/lenet-mnist-caffe/lenet/lenet.prototxt /usr/src/tensorrt/data/mnist/
cp ~/nvdla_loadables/lenet-mnist-caffe/lenet/lenet.caffemodel /usr/src/tensorrt/data/mnist/
```

> **_NOTE:_** TensorRT also includes a pre-trained LeNet Caffe model, but it's for 64 images on the input at the same time instead of just one image input. I tried it and it worked, but the execution time on NVDLA was 64 times longer than LeNet from Lei Wang's repository. Also, as far as I know, NVDLA does not support multiple inputs.


### 4.1.4 Quantization
Example code with LeNet quantization is in the `/opt/tensorrt/samples/python/int8_caffe_mnist` directory.
```
cd /opt/tensorrt/samples/python/int8_caffe_mnist
```

There are two Python files, one called `sampler.py` which is the main file and the other called `calibrator.py`. We need to edit our Caffe prototxt and model name in the `sampler.py` file. Change two variables as shown below.
```
DEPLOY_PATH = "lenet.prototxt"
MODEL_PATH = "lenet.caffemodel"
```

Once the variables are set correctly, we can run the quantization itself.
```
python sample.py
```

The file `mnist_calibration.cache` is created in the same folder with the scales factors. Copy this file into the virtual machine with Ubuntu 18.04.

## 4.2 NVDLA Loadable
In the Ubuntu 18.04 virtual machine, open the `mnist_calibration.cache` file. You should see something like this.
```
TRT-7202-EntropyCalibration2
data: 3c008912
conv1: 3c60dd38
pool1: 3c60dd38
conv2: 3d895e44
pool2: 3d895e44
(Unnamed Layer* 4) [Fully Connected]_output: 3d65992f
ip1: 3d65ff18
ip2: 3e11094c
prob: 3c083ffd
```
There is an unnamed layer, probably due to a bug in TensorRT. Replace `(Unnamed Layer* 4) [Fully Connected]_output` with `relu1` and save the file.

Then convert the calibration cache to a json file that can be read by the `nvdla_compiler`.
```
python <nvdla sw>/umd/utils/calibdata/calib_txt_to_json.py mnist_calibration.cache mnist_calibration.json
```

Once the file is converted, everything is ready for compilation.
```
$ ./nvdla_compiler --prototxt lenet.prototxt --caffemodel lenet.caffemodel --configtarget nv_small --cprecision int8 --calibtable mnist_calibration.json 
creating new wisdom context...
opening wisdom context...
parsing caffe network...
libnvdla<3> mark prob
Marking total 1 outputs
parsing calibration table...
attaching parsed network to the wisdom...
compiling profile "fast-math"... config "nv_small"...
libnvdla<2> Prototxt #chnls (C = 1) != Profile #chnls for input (NVDLA_IMG_A8B8G8R8: C = 4). Preferring #chnls from Profile for compiling.
closing wisdom context...
```
There is a warning that should be fixed in a later version of the manual. You can ignore it for now.

A NVDLA Loadable file called `fast-math.nvdla` should be created in the same folder. And that is all for compiling Caffe to NVDLA Loadable.

> **_NOTE:_** There is an option to select the target platform `--configtarget` from one of the options `<opendla-full|opendla-large|opendla-small>`, or so the `nvdla_compiler` help says. But you will get the error `libnvdla<1> Invalid target config` if you try any of them. The correct options are `nv_full`, `nv_large` and `nv_small`.