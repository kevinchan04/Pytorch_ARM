# Install PyTorch v1.6 or v1.7 and torchvision on Raspberry Pi 3b+

This is the tutorial to teach you how to compile Pytorch on ARM68v8 (aarch64) CPU that also on Raspberry Pi. My device are

```
Raspberry Pi 3b+
ubuntu 18.04 server 64-bit
gcc / g++ in version 7.5 
```

# PyTorch

## set swap space

```bash
sudo apt-get install dphys-swapfile
sudo vim /etc/dphys-swapfile

# activate swapfile service
sudo service dphys-swapfile start

# check swap space
free -m
```

![Install%20PyTorch%20v1%206%20or%20v1%207%20and%20torchvision%20on%20Ra%20083c73b038274875bfe072bd290eed38/Untitled.png](Install%20PyTorch%20v1%206%20or%20v1%207%20and%20torchvision%20on%20Ra%20083c73b038274875bfe072bd290eed38/Untitled.png)

Uncomment and modify `CONF_MAXSWAP` and `CONF_SWAPSIZE` to `4096` if you are using raspberry pi 3b+ only with 1GB RAM.

## create virtual environment (optional, recommend)

```bash
# install virtual environment
sudo pip3 install -U virtualenv

# create your virtual env that names pytorch1.7
virtualenv -p python3 ~/my_envs/pytorch1.7

# activate your virtual env
source ~/my_envs/pytorch1.7/bin/activate
```

## install required packets

```bash
sudo apt-get update
sudo apt-get install libopenblas-dev cython3 libatlas-base-dev m4 libblas-dev cmake -y
```

## install numpy and pyyaml

```bash
pip3 install numpy pyyaml -y
```

## clone Pytorch source code

```bash
git clone https://github.com/pytorch/pytorch.git
cd pytorch
```

## select the version you want to install

```bash
# check the version 
git branch -a

# switch to the branch
git checkout v1.7.0
git submodule update --init --recursive 

# to avoid the problem from protobuf
git submodule update --remote third_party/protobuf
```

## set temporary environment variables

```bash
# simplest, recommend
export NO_CUDA=1 
export NO_DISTRIBUTED=1 
export NO_MKLDNN=1
```

```bash
# complex
export USE_CUDA=0 
export USE_MKLDNN=0 
export USE_NNPACK=0 
export USE_QNNPACK=0 
export USE_NUMPY=1 
export USE_DISTRIBUTED=0 
export NO_CUDA=1 
export NO_DISTRIBUTED=1 
export NO_MKLDNN=1 
export NO_NNPACK=1 
export NO_QNNPACK=1 
export ONNX_ML=1
```

If your raspberry pi RAM is lower than 1GB, some tutorials told you to reduce the parallel compiling jobs on CPU by `export MAX_JOBS=1`. However, it will increase the compile time. So, I think adding swap space to 4GB can handle this problem and I used 4 cores on CPU to compile Pytorch.

## modify XNNPACK source code

```bash
nano aten/src/ATen/native/quantized/cpu/qnnpack/src/q8gemm/8x8-dq-aarch64-neon.S
```

[Can't compile pytorch from source · Issue #34040 · pytorch/pytorch](https://github.com/pytorch/pytorch/issues/34040)

![Install%20PyTorch%20v1%206%20or%20v1%207%20and%20torchvision%20on%20Ra%20083c73b038274875bfe072bd290eed38/Untitled%201.png](Install%20PyTorch%20v1%206%20or%20v1%207%20and%20torchvision%20on%20Ra%20083c73b038274875bfe072bd290eed38/Untitled%201.png)

![Install%20PyTorch%20v1%206%20or%20v1%207%20and%20torchvision%20on%20Ra%20083c73b038274875bfe072bd290eed38/Untitled%202.png](Install%20PyTorch%20v1%206%20or%20v1%207%20and%20torchvision%20on%20Ra%20083c73b038274875bfe072bd290eed38/Untitled%202.png)

## start compiling

```bash
python3 setup.py bdist_wheel
```

## install after compiling

```bash
cd dist

# the whl filename depends on your python/torch/cpu
pip3 install ./torch-1.6.0a0+b31f58d-cp36-cp36m-linux_aarch64.whl
```

If you use python 3.6 on raspberry pi, and your architecture is `aarch64` , you can download my compiled file directly.

```bash
# check your system architecture
uname -a
```

# Torchvision

```bash
sudo apt-get install libjpeg-dev libavcodec-dev libavformat-dev libswscale-dev
pip3 install pillow

git clone https://github.com/pytorch/vision.git
cd vision
git checkout v0.7.0

python setup.py bdist_wheel
```

```bash
cd dist

# the whl filename depends on your python/torch/cpu
pip3 install ./torchvision-0.7.0a0+78ed10c-cp36-cp36m-linux_aarch64.whl
```

## When you meet problems while compiling

```bash
# clean the build process
make clean
python setup.py clean
```

Please note that in Ubuntu 18.04 you should use gcc/g++ in version larger than 6.0


# Reference

How to install Pytorch on raspberry pi?

[Install pyTorch in Raspberry Pi 4 (or any other)](https://gist.github.com/akaanirban/621e63237e63bb169126b537d7a1d979)

[在树莓派上安装Pytorch (Installing Pytorch on Raspberry Pi) ---- 一个比较详细的版本_油炸花生米34的博客-CSDN博客](https://blog.csdn.net/weixin_39965127/article/details/102686314)

[在树莓派上安装PyTorch 1.0](https://zhuanlan.zhihu.com/p/57938855)

[PyTorch packages for ARM64](https://mathinf.eu/pytorch/arm64/)

[树莓派上编译安装pytorch1.4](https://cloud.tencent.com/developer/article/1625850)

How to fix bugs when you met during compile?

[Cannot build Caffe2 while installing Pytorch on Raspbian (buster) · Issue #22564 · pytorch/pytorch](https://github.com/pytorch/pytorch/issues/22564#issuecomment-509029165)

[Can't compile pytorch from source · Issue #34040 · pytorch/pytorch](https://github.com/pytorch/pytorch/issues/34040)
