---
layout: post
title: Python deep learning setup on GPU
subtitle: Get up and running in less than an hour
background: '/img/posts/03-deeplearning.jpg'
---

## What is deep learning.

This is a neural network:

![NN](https://timedotcom.files.wordpress.com/2015/01/mcdonalds-burger.jpg?w=753&quality=85){:height="100%" width="100%"}

This is deep learning:

![DL](http://files.doobybrain.com/wp-content/uploads/2007/10/big-burger.jpg){:height="100%" width="100%"}

Then why is this all the rage when we've seen Neural Networks from the 70's?

Answer is, we now have mouths and stomachs that can handle the second burger with relative ease. So yeah, is not only that Gogle guys are crazy smart, we've always have smart people in Computer Science.

**We've never had this hardware.**

Of course this is grossly oversimplified and I should burn in hell for this comparison. If you want to dive deep, get a [whitepaper](https://arxiv.org/pdf/1702.07800.pdf).

The most common language to exploit DL capabilities is Python, with the Tensorflow (TF) framework. On this post we'll set up an environment to work with tensorflow on GPU.

In order for TF to work you'll need to install some libraries on your OS (CUDA for GPU computation and cuDNN for Deep Learning capabilities on GPU), and do some easy variable and python management.

# Step 0: Issues using TF

#### Issue #1: Compatibility and Version Hell.

Each TF version is compiled for a specific version of both CUDA and cuDNN. No backwards compatibility. Let me try to rephrase that. Each TF version is ONLY compatible with ONE combination of CUDA and cuDNN drivers. And no, you cannot use CUDA 10.1 if TF is only for 10.0

This is the tested build configurations:

https://www.tensorflow.org/install/source#tested_build_configurations

Make sure you know which version of Tensorflow do you need. For this I'd install TF 1.14 as it is the requirement of the faceswap project: https://github.com/deepfakes/faceswap

#### Issue #2: Nvidia Drivers on machines that do not have nvidia drivers AKA Nouveau

Some linux distros come with Nouveau prepacked instead of the proprietary Nvidia drivers. This is a good thing (FOSS and all that), but disabling it is a bit of a pain I want to experience as few times as possible. Be careful if your environment does not have the Nvidia drivers. 

You can check that with:

```
lsmod | grep nouveau
```

I won't cover how to install the nvidia driver. You can check if you have it with (for instance) `nvidia-smi`. Find a guide on how to install the driver [here](https://www.advancedclustering.com/act_kb/installing-nvidia-drivers-rhel-centos-7/).

#### Issue/rant #3: It looks way harder than it is.

Installing Cuda / cuDNN is extremely simple, and can be done on the same machine with multiple versions! Those two super fancy things are just a collection of libraries. There seems to be an obsesion to keep Deep Learning related topics complicated, they are not. </rant>


# Step 1: Install CUDA / cuDNN

Once you know the Tensorflow version you're going to install, you need to make sure which CUDA version is compatible with it. In my case, since I'm using TF 1.14, I'll need CUDA 10.0.
You can select your target CUDA version [here](https://developer.nvidia.com/cuda-toolkit-archive).

Download the toolkit 

![Download](https://i.imgur.com/KGqwB1W.png){:height="100%" width="100%"}

Install the toolkit with:

```
sudo sh cuda_xxxxxxxx.run --toolkit --silent
```

This installs CUDA library files under `/usr/local/cuda-<version>`. You can safely delete the installer.

For cuDNN, first check the compatibility matrix for both [Tensorflow](https://www.tensorflow.org/install/source#tested_build_configurations) and [CUDA](https://docs.nvidia.com/deeplearning/sdk/cudnn-support-matrix/index.html).

Once you get the version you need to install (in my case cuDNN 7.4), download that version from https://developer.nvidia.com/rdp/cudnn-archive. You need to be registered as a Nvidia developer for it (don't worry is free, and no spam).

The download is a tar.gz package. You need to unpack it and move some files to your CUDA directory.
```
tar -xzvf cudnn-10.0-linux-x64-v7.tgz
sudo cp cuda/include/cudnn.h /usr/local/cuda-<version>/include
sudo cp cuda/lib64/libcudnn* /usr/local/cuda-<version>/lib64/
```

### Multi CUDA / cuDNN

You can definitelly have multiple CUDA cuDNN versions cohexisting on the same machine. There is a couple of caveats you need to consider though:

- When installing CUDA it will create a link `/usr/local/cuda` which will point to the last CUDA version installed. If you want to keep a version you installed before as the default, just change that symlink.
- You need to copy the cuDNN libraries to that version too.

There is something else I like to do, which is creating a virtualenv for your project to isolate dependencies. You could have different TF versions that way and you'll always keep your deps isolated from the rest. It's very common for a group of people to share a GPU enabled instance as those are VERY expensive.

# Step 2: Install Tensorflow

I strongly recommend using a virtualenv for your DL project (well, for almost any project you're starting).

Create the virtualenv with:

```
pip install virtualenv
virtualenv deeplearning
```

If you prefer conda to manage your virtual environments, please feel free to use it.

And install your required version of TF. It is very important you use the `-gpu` and not the one without the prefix, that will make use of your CPUs, not your GPUs. 
You need to be careful with this for TF versions before 1.14, they unified the packages for `tensorflow>=1.15`.


```
pip install tensorflow==1.14
```

# Step 3: Make Tensorflow aware of CUDA and GPUs

Sounds hard? just TWO ENVIRONMENT VARIABLES:

- `LD_LIBRARY_PATH` so it knows where to find the .so files (libraries) for the CUDA / cuDNN. Like for any other PATH variable it's a colon separated list of folders where you can find the files:

```
export LD_LIBRARY_PATH=export LD_LIBRARY_PATH=/usr/local/cuda-<version>/lib64:/usr/local/cuda-<version>/extras/CUPTI/lib64:$LD_LIBRARY_PATH
```

- `CUDA_VISIBLE_DEVICES` the list of GPU ids that your session can use.

```
# SINGLE GPU
export CUDA_VISIBLE_DEVICES=0

# GPU LIST
export CUDA_VISIBLE_DEVICES=0,1,3
```

You can set this on your environment or be smarter and modify your virtualenv activation / deactivation scripts. This will mean that every time you activate an environment, you don't neet to worry again about this.

At least for the LD_LIBRARY_PATH I highly recommend it:

```
# FILE  ~/deeplearning/bin/activate

[...]
deactivate(){
	[...]
	export LD_LIBRARY_PATH=$DEFAULT_LD_LIBRARY_PATH
	unset CUDA_VISIBLE_DEVICES
	[...]
}

[...]
# I like to put this at the end
export DEFAULT_LD_LIBRARY_PATH=$LD_LIBRARY_PATH
export LD_LIBRARY_PATH=/usr/local/cuda-<version>/lib64:/usr/local/cuda-<version>/extras/CUPTI/lib64:$LD_LIBRARY_PATH
```


# Step 4: Check tensorflow:

Use this code after you activate your virtualenv. You should at least have the variables for the LD_LIBRARY_PATH exported, either on your virtualenv or just as an `export` command. Invoke python's REPL with `python` and try running this:


```
a = tf.constant(2)
b = tf.constant(3)

with tf.Session() as sess:
    print("a=2, b=3")
    print("Addition with constants: %i" % sess.run(a+b))
    print("Multiplication with constants: %i" % sess.run(a*b))
```



The result should look like this (I used `export CUDA_VISIBLE_DEVICES=0` before running):

```
>>> # Launch the default graph.
... with tf.Session() as sess:
...     print("a=2, b=3")
...     print("Addition with constants: %i" % sess.run(a+b))
...     print("Multiplication with constants: %i" % sess.run(a*b))
...
2019-10-08 08:19:54.438188: I tensorflow/stream_executor/platform/default/dso_loader.cc:42] Successfully opened dynamic library libcuda.so.1
2019-10-08 08:19:54.489470: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1640] Found device 0 with properties:
name: Tesla V100-PCIE-16GB major: 7 minor: 0 memoryClockRate(GHz): 1.38
pciBusID: 0000:af:00.0
2019-10-08 08:19:54.489696: I tensorflow/stream_executor/platform/default/dso_loader.cc:42] Successfully opened dynamic library libcudart.so.10.0
2019-10-08 08:19:54.491453: I tensorflow/stream_executor/platform/default/dso_loader.cc:42] Successfully opened dynamic library libcublas.so.10.0
2019-10-08 08:19:54.492924: I tensorflow/stream_executor/platform/default/dso_loader.cc:42] Successfully opened dynamic library libcufft.so.10.0
2019-10-08 08:19:54.493266: I tensorflow/stream_executor/platform/default/dso_loader.cc:42] Successfully opened dynamic library libcurand.so.10.0
2019-10-08 08:19:54.494979: I tensorflow/stream_executor/platform/default/dso_loader.cc:42] Successfully opened dynamic library libcusolver.so.10.0
2019-10-08 08:19:54.496391: I tensorflow/stream_executor/platform/default/dso_loader.cc:42] Successfully opened dynamic library libcusparse.so.10.0
2019-10-08 08:19:54.500327: I tensorflow/stream_executor/platform/default/dso_loader.cc:42] Successfully opened dynamic library libcudnn.so.7
2019-10-08 08:19:54.509655: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1763] Adding visible gpu devices: 0
2019-10-08 08:19:54.510414: I tensorflow/core/platform/cpu_feature_guard.cc:142] Your CPU supports instructions that this TensorFlow binary was not compiled to use: AVX2 AVX512F FMA
2019-10-08 08:19:54.984804: I tensorflow/compiler/xla/service/service.cc:168] XLA service 0x5571c22e0f20 executing computations on platform CUDA. Devices:
2019-10-08 08:19:54.984883: I tensorflow/compiler/xla/service/service.cc:175]   StreamExecutor device (0): Tesla V100-PCIE-16GB, Compute Capability 7.0
2019-10-08 08:19:54.994560: I tensorflow/core/platform/profile_utils/cpu_utils.cc:94] CPU Frequency: 2300000000 Hz
2019-10-08 08:19:55.006894: I tensorflow/compiler/xla/service/service.cc:168] XLA service 0x5571c236f110 executing computations on platform Host. Devices:
2019-10-08 08:19:55.006955: I tensorflow/compiler/xla/service/service.cc:175]   StreamExecutor device (0): <undefined>, <undefined>
2019-10-08 08:19:55.014353: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1640] Found device 0 with properties:
name: Tesla V100-PCIE-16GB major: 7 minor: 0 memoryClockRate(GHz): 1.38
pciBusID: 0000:XX:00.0
2019-10-08 08:19:55.014453: I tensorflow/stream_executor/platform/default/dso_loader.cc:42] Successfully opened dynamic library libcudart.so.10.0
2019-10-08 08:19:55.014494: I tensorflow/stream_executor/platform/default/dso_loader.cc:42] Successfully opened dynamic library libcublas.so.10.0
2019-10-08 08:19:55.014533: I tensorflow/stream_executor/platform/default/dso_loader.cc:42] Successfully opened dynamic library libcufft.so.10.0
2019-10-08 08:19:55.014571: I tensorflow/stream_executor/platform/default/dso_loader.cc:42] Successfully opened dynamic library libcurand.so.10.0
2019-10-08 08:19:55.014610: I tensorflow/stream_executor/platform/default/dso_loader.cc:42] Successfully opened dynamic library libcusolver.so.10.0
2019-10-08 08:19:55.014648: I tensorflow/stream_executor/platform/default/dso_loader.cc:42] Successfully opened dynamic library libcusparse.so.10.0
2019-10-08 08:19:55.014686: I tensorflow/stream_executor/platform/default/dso_loader.cc:42] Successfully opened dynamic library libcudnn.so.7
2019-10-08 08:19:55.034804: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1763] Adding visible gpu devices: 0
2019-10-08 08:19:55.034941: I tensorflow/stream_executor/platform/default/dso_loader.cc:42] Successfully opened dynamic library libcudart.so.10.0
2019-10-08 08:19:55.045107: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1181] Device interconnect StreamExecutor with strength 1 edge matrix:
2019-10-08 08:19:55.045135: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1187]      0
2019-10-08 08:19:55.045153: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1200] 0:   N
2019-10-08 08:19:55.065327: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1326] Created TensorFlow device (/job:localhost/replica:0/task:0/device:GPU:0 with 15022 MB memory) -> physical GPU (device: 0, name: Tesla V100-PCIE-16GB, pci bus id: 0000:XX:00.0, compute capability: 7.0)
a=2, b=3
```

If you get something including something similar to this line:

```
tensorflow/core/common_runtime/gpu/gpu_device.cc:1663] Cannot dlopen some GPU libraries. Skipping registering GPU devices...
```

It means that Tensorflow fell back to use CPU.

-----------------

## Common errors

```
2019-10-08 07:19:26.640811: I tensorflow/stream_executor/platform/default/dso_loader.cc:42] Successfully opened dynamic library libcuda.so.1
2019-10-08 07:19:26.799847: I tensorflow/core/common_runtime/gpu/gpu_device.cc:1640] Found device 0 with properties:
name: Tesla V100-PCIE-16GB major: 7 minor: 0 memoryClockRate(GHz): 1.38
pciBusID: 0000:af:00.0
2019-10-08 07:19:26.800215: I tensorflow/stream_executor/platform/default/dso_loader.cc:53] Could not dlopen library 'libcudart.so.10.0'; dlerror: libcudart.so.10.0: cannot open shared object file: No such file or directory
2019-10-08 07:19:26.800389: I tensorflow/stream_executor/platform/default/dso_loader.cc:53] Could not dlopen library 'libcublas.so.10.0'; dlerror: libcublas.so.10.0: cannot open shared object file: No such file or directory
2019-10-08 07:19:26.800547: I tensorflow/stream_executor/platform/default/dso_loader.cc:53] Could not dlopen library 'libcufft.so.10.0'; dlerror: libcufft.so.10.0: cannot open shared object file: No such file or directory
2019-10-08 07:19:26.800700: I tensorflow/stream_executor/platform/default/dso_loader.cc:53] Could not dlopen library 'libcurand.so.10.0'; dlerror: libcurand.so.10.0: cannot open shared object file: No such file or directory
2019-10-08 07:19:26.800849: I tensorflow/stream_executor/platform/default/dso_loader.cc:53] Could not dlopen library 'libcusolver.so.10.0'; dlerror: libcusolver.so.10.0: cannot open shared object file: No such file or directory
2019-10-08 07:19:26.800993: I tensorflow/stream_executor/platform/default/dso_loader.cc:53] Could not dlopen library 'libcusparse.so.10.0'; dlerror: libcusparse.so.10.0: cannot open shared object file: No such file or directory
2019-10-08 07:19:26.812246: I tensorflow/stream_executor/platform/default/dso_loader.cc:42] Successfully opened dynamic library libcudnn.so.7
2019-10-08 07:19:26.812288: W tensorflow/core/common_runtime/gpu/gpu_device.cc:1663] Cannot dlopen some GPU libraries. Skipping registering GPU devices...
2019-10-08 07:19:26.813180: I tensorflow/core/platform/cpu_feature_guard.cc:142] Your CPU supports instructions that this TensorFlow binary was not compiled to use: AVX2 AVX512F FMA
```

Can happen when you don't have CUDA installed properly or `LD_LIBRARY_PATH` is not pointing to the right path. Check that variable and make sure you export it either on your session or with the virtualenv activate script.

Also make sure you have permissions to read (a+r) all files on the cuda path. If you don't change them with a:

```
sudo chmod a+r /usr/local/cuda-<version> -R
```


## Resource management with Tensorflow:

Unfortunatelly resource management on GPUs is still a lot messier than on the CPU counterparts, there is no queue anywhere and your jobs won't run smoothly if you don't dedicate a GPU per job.

That is the `CUDA_VISIBLE_DEVICES` variable job. This variable is not mandatory, if you don't set it, you'll see all available GPUs. BUT, being able to launch processes on only one GPU is a simple yet powerful way to distribute resources.

For example, you can have user A, B working on a server with 8 GPUs, you can assign 4 GPUs for each user, and they won't have a problem sharing resources. Furthermore, those users can launch multiple jobs in parallel if they declare which GPU are they launching on before doing so.

To check which GPUs are in use and at which percentage you can use `nvidia-smi`.

```
# nvidia-smi
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 418.67       Driver Version: 418.67       CUDA Version: 10.0     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Tesla V100-PCIE...  On   | 00000000:XX:00.0 Off |                    0 |
| N/A   28C    P0    24W / 250W |     11MiB / 16130MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   1  Tesla V100-PCIE...  On   | 00000000:XX:00.0 Off |                    0 |
| N/A   33C    P0    79W / 250W |   9670MiB / 16130MiB |     60%      Default |
+-------------------------------+----------------------+----------------------+
|   2  Tesla V100-PCIE...  On   | 00000000:XX:00.0 Off |                    0 |
| N/A   42C    P0    55W / 250W |   5666MiB / 16130MiB |     35%      Default |
+-------------------------------+----------------------+----------------------+
|   3  Tesla V100-PCIE...  On   | 00000000:xX:00.0 Off |                    0 |
| N/A   26C    P0    24W / 250W |     11MiB / 16130MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   4  Tesla V100-PCIE...  On   | 00000000:XX:00.0 Off |                    0 |
| N/A   42C    P0    55W / 250W |   5666MiB / 16130MiB |     30%      Default |
+-------------------------------+----------------------+----------------------+
|   5  Tesla V100-PCIE...  On   | 00000000:XX:00.0 Off |                    0 |
| N/A   42C    P0    55W / 250W |   4902MiB / 16130MiB |     65%      Default |
+-------------------------------+----------------------+----------------------+
|   6  Tesla V100-PCIE...  On   | 00000000:XX:00.0 Off |                    0 |
| N/A   42C    P0    55W / 250W |   1603MiB / 16130MiB |     10%      Default |
+-------------------------------+----------------------+----------------------+
|   7  Tesla V100-PCIE...  On   | 00000000:XX:00.0 Off |                    0 |
| N/A   42C    P0    55W / 250W |   5666MiB / 16130MiB |     35%      Default |
+-------------------------------+----------------------+----------------------+

```

With this I know GPUs 0 and 3 are free, so before I launch my job I'd do

```
export CUDA_VISIBLE_DEVICES=0

# or 

export CUDA_VISIBLE_DEVICES=3
```

So my jobs don't get interrupted.
