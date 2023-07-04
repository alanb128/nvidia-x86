# nvidia-x86 on balena
Example of using an Nvidia GPU in an x86 device on the balena platform. See the accompanying [blog post](https://www.balena.io/blog/how-to-use-nvidia-gpu-on-x86-device-balenaOS/) for more details!

Note that although these examples should work as-is, the resulting images are quite large and should be optimized for your particular use case. One possibility is to utilize [multistage builds](https://www.balena.io/docs/learn/deploy/build-optimization/#multi-stage-builds) to reduce the size of your containers. Below is a summary of the containers in this project, with all of the details following in the next section.

| Service | Image Size | Description |
| ------------ | ----------- | ----------- |
| gpu | 2.28 GB | main example - downloads, builds and installs gpu kernel modules |
| cuda | 8.60 GB | example container with CUDA toolkit installed |
| app | 7.26 GB | example of app container installing PyTorch |
| nv-pytorch | 14.96 GB | example of using an Nvidia base image for PyTorch |

## How it works
### gpu container
This is the main container in this example and the only one you need to obtain GPU access from within your container or for any other containers in the application. It downloads the kernel source files for our exact OS version and uses them, along with the driver file downloaded from Nvidia to build the required Nvidia kernel modules. Finally, the `entry.sh` file unloads the current Nouveau driver if it's running and loads the Nvidia modules.

This container also provides CUDA compiled application support, though not development support - see the CUDA container example for development use. You could use this image as a base image, build on top of the current example, or use alongside other containers to provide them with gpu access.

Before using this example, you'll need to make sure that you've set the variables at the top of the Dockerfile:
- `VERSION` is the version of balenaOS being used on your device. This needs to be URL-encoded, so a plus sign (+) if present needs to be written as `%2B`.
- `BALENA_MACHINE_NAME` is the device type of your balenaOS from [this list](https://www.balena.io/docs/reference/hardware/devices/)
- `YOCTO_VERSION` is the version of Yocto Linux used to build your version of balenaOS. You can find it by logging into your host OS and typing: `uname -r`
- `NVIDIA_DRIVER_VERSION` is the version of the Nvidia driver you want to download and build using the list found [here]( https://www.nvidia.com/en-us/drivers/unix/) Usually, you can use the "Latest Production Branch Version". Be sure to use the exact same driver version in any other containers that need access to the GPU. 

If this container is set up and running properly, you should see the output below (for your gpu model) in the terminal:
```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 470.86       Driver Version: 470.86       CUDA Version: 11.4     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  Quadro P400         Off  | 00000000:01:00.0 Off |                  N/A |
| 22%   37C    P0    N/A /  N/A |      0MiB /  2000MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+ 
```
### starter container
The starter container example demonstrates how to create a separate container from the gpu container that has access to the GPU. The gpu container must be running first for this container to work properly. You'll need to supply a few variables for the Dockerfile, and they must match the values used in the accomanying gpu container as described above. 
- `YOCTO_VERSION` is the version of Yocto Linux used to build your version of balenaOS. You can find it by logging into your host OS and typing: `uname -r`
- `NVIDIA_DRIVER_VERSION` is the version of the Nvidia driver you want to download and build using the list found [here]( https://www.nvidia.com/en-us/drivers/unix/) Usually, you can use the "Latest Production Branch Version". Be sure to use the eaxct same driver version as in the gpu container.

This is one example of separating the GPU kernel module loading from an application container. Note that all containers needing GPU access must load Nvidia drivers that exactly match the version used in the gpu container. In this case, the same driver from the same Nvidia source as the gpu container is used. (In the app container example below, the distribution's Nvidia drivers are used instead.) If this container is running properly, you should see the same output of `nvidia-smi` as you do from the gpu container. You could then add to this Dockerfile to install your own software (such as CUDA, Deepstream, etc...) that requires GPU access.
 
 ### app container
 The app container is another example of how you can separate the GPU access setup from an application that requires GPU access. In this case, we use the Ubuntu distribution's drivers instead of the Nvidia drivers. However, the version must still match the one used in the gpu container (which is required to be running alongside this one). You can see a list of the available driver versions [here](https://launchpad.net/~graphics-drivers/+archive/ubuntu/ppa). The "app" we install is PyTorch and then a Python example is run to confirm CUDA GPU access. If the container is running properly, you should see the following output, customized for your GPU model:
```
 app  Checking for CUDA device(s) for PyTorch...
 app  ------------------------------------------
 app  Torch CUDA available: True
 app  Torch CUDA current device: 0
 app  Torch CUDA device info: <torch.cuda.device object at 0x7f7c749ed610>
 app  Torch CUDA device count: 1
 app  Torch CUDA device name: Quadro P400
```

### nv-pytorch
This is an example of using a pre-built container from the [Nvidia NGC Catalog](https://catalog.ngc.nvidia.com/). In this case we use their PyTorch container as a base image, install the Nvidia drivers on top of that (using the same version as in the gpu container which is required to be running alongside this) and then run a simple Python script that confirms CUDA GPU access. Note that we are not installing/running the Container Toolkit! If this container is running properly, you should see the following output customized for your GPU model:
```
 nv-pytorch  Checking for CUDA device(s) for PyTorch...
 nv-pytorch  ------------------------------------------
 nv-pytorch  Torch CUDA available: True
 nv-pytorch  Torch CUDA current device: 0
 nv-pytorch  Torch CUDA device info: <torch.cuda.device object at 0x7f7c749ed610>
 nv-pytorch  Torch CUDA device count: 1
 nv-pytorch  Torch CUDA device name: Quadro P400
```
