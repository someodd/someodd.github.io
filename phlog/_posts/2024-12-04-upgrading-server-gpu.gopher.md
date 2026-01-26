---
date: 2024-12-04
tags:
- sysadmin
- linux
- hardware
- ai
- debian
title: Server upgrade! 4x RAM + GPU! Including CUDA+ollama!

---


This is both a server announcement and a guide!

I quadrupled my server memory and installed a GPU.

Quadro P620

## Lenovo ST50 "Unqualified DIMM"

## Making the most out of my Nvidia p620 (?)

### Getting the GPU working

Time to install the appropriate drivers.

```
sudo apt update
sudo apt install nvidia-driver
```

I got some warnings about some driver bugs with certain Linux kernel, but after
confirming I wasn't affected by checking my kernel version `uname -r` I just
accepted/moved on. My display stopped displaying during the install process.

After reboot, check if it's working:

```
nvidia-smi
```

I got:

```
baudrillard@simulacra ~ % nvidia-smi
Wed Dec  4 16:18:35 2024       
+---------------------------------------------------------------------------------------+
| NVIDIA-SMI 535.183.01             Driver Version: 535.183.01   CUDA Version: 12.2     |
|-----------------------------------------+----------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |         Memory-Usage | GPU-Util  Compute M. |
|                                         |                      |               MIG M. |
|=========================================+======================+======================|
|   0  Quadro P620                    On  | 00000000:02:00.0  On |                  N/A |
| 34%   44C    P8              N/A /  N/A |    188MiB /  2048MiB |      0%      Default |
|                                         |                      |                  N/A |
+-----------------------------------------+----------------------+----------------------+
                                                                                         
+---------------------------------------------------------------------------------------+
| Processes:                                                                            |
|  GPU   GI   CI        PID   Type   Process name                            GPU Memory |
|        ID   ID                                                             Usage      |
|=======================================================================================|
|    0   N/A  N/A     10606      G   /usr/lib/xorg/Xorg                          185MiB |
+---------------------------------------------------------------------------------------+
```

Can check OpenGL, if you're using an actual display (maybe works in xrdp, too):

```
sudo apt update
sudo apt install mesa-utils
glxinfo | grep OpenGL
```

#### Caveats

##### xrdp

After I installed the GPU I noticed xrdp stopped working!

IDK why but I rebooted or something and now it's working, so just try
connecting a few times, restarting, etc.

### Testing `ollama`

I checked these pages to make sure my GPU is supported:

* https://developer.nvidia.com/cuda-gpus
* https://github.com/ollama/ollama/blob/main/docs/gpu.md

I saw the P620 is supported on both pages, so pretty sure!

I installed the `nvidia-cuda-toolkit`:

```
sudo apt install nvidia-cuda-toolkit
```

Verify CUDA installation:

```
nvcc --version
```

Although this prompted me about how `gcc-11` has a serious bug that should not be shipped in `trixie`. I just accepted the risk.

I'm going to install `nvtop` to see if my GPU is being used by `ollama` (I ran
`ollama run llama3`. While `nvtop` was running I asked `ollama` to "write a
story that's mathematically complex" and I saw my GPU usage shoot up. I was also able to see by running
`nvidia-smi`, you could also do something like `watch -n 0.5 nvidia-smi`.

All looks pretty good to me!

## Final notes

Darn, that fan is sure annoying on that little low-profile P620.

Original content in gopherspace: [gopher://gopher.someodd.zip:70/0/phlog/upgrading-server-gpu.gopher.txt](gopher://gopher.someodd.zip:70/0/phlog/upgrading-server-gpu.gopher.txt)
