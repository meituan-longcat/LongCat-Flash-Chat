## AMD GPU Installation and Testing Guide
### Please follow the steps here to install and run LongCat-Flash-Chat models on AMD MI300X GPU.
### Step By Step Guide
#### Step 1
Launch the Rocm-vllm docker:

```shell
docker run -d -it --ipc=host --network=host --privileged --cap-add=CAP_SYS_ADMIN --device=/dev/kfd --device=/dev/dri --device=/dev/mem --group-add video --cap-add=SYS_PTRACE --security-opt seccomp=unconfined -v /:/work -e SHELL=/bin/bash  --name vllm_longcat rocm/vllm-dev:nightly
```

#### Step 2
  Huggingface login

```shell
   huggingface-cli login 
```   

  Install pre-requisites: Please cherry pick the Longcat-flash-chat PR manually before it's merged by upstream vllm. 

```shell
  pip uninstall vllm
  apt-get update
  pip install --upgrade pip
  git clone https://github.com/vllm-project/vllm.git
  cd vllm
  apt -y install gh
  gh pr checkout 23991
  git log
  ```
  commit 3479595edc81a28b1d94ab2219f3ac0d9149744c (HEAD -> LongCatFlash)
  ```
  export PYTORCH_ROCM_ARCH="gfx942"
  python3 setup.py develop
```

### Please wait until the compilation completed.
```
[34/34] Linking HIP shared module _rocm_C.abi3.so
-- Install configuration: "RelWithDebInfo"
-- Installing: /work/home/hc/vllm/build/lib.linux-x86_64-cpython-312/vllm/_moe_C.abi3.so
-- Set non-toolchain portion of runtime path of "/work/home/hc/vllm/build/lib.linux-x86_64-cpython-312/vllm/_moe_C.abi3.so" to ""
-- Install configuration: "RelWithDebInfo"
-- Installing: /work/home/hc/vllm/build/lib.linux-x86_64-cpython-312/vllm/_rocm_C.abi3.so
-- Set non-toolchain portion of runtime path of "/work/home/hc/vllm/build/lib.linux-x86_64-cpython-312/vllm/_rocm_C.abi3.so" to ""
-- Install configuration: "RelWithDebInfo"
-- Installing: /work/home/hc/vllm/build/lib.linux-x86_64-cpython-312/vllm/_C.abi3.so
-- Set non-toolchain portion of runtime path of "/work/home/hc/vllm/build/lib.linux-x86_64-cpython-312/vllm/_C.abi3.so" to ""
copying build/lib.linux-x86_64-cpython-312/vllm/_moe_C.abi3.so -> vllm
copying build/lib.linux-x86_64-cpython-312/vllm/_rocm_C.abi3.so -> vllm
copying build/lib.linux-x86_64-cpython-312/vllm/_C.abi3.so -> vllm
Creating /usr/local/lib/python3.12/dist-packages/vllm.egg-link (link to .)
Adding vllm 0.10.2rc2.dev69+g3479595ed.rocm641 to easy-install.pth file
Installing vllm script to /usr/local/bin
...
Using /usr/local/lib/python3.12/dist-packages
Finished processing dependencies for vllm==0.10.2rc2.dev69+g3479595ed.rocm641

```

#### Step 3
Run the vllm online serving
Sample Command

```shell
VLLM_USE_V1=1 \
VLLM_USE_TRITON_FLASH_ATTN=0 \
SAFETENSORS_FAST_GPU=1 \
vllm serve meituan-longcat/LongCat-Flash-Chat-FP8 \
    --trust-remote-code \
    --enable-expert-parallel \
    --tensor-parallel-size 8 \
    --gpu-memory-utilization 0.98 \
    --no-enable-prefix-caching 
```
