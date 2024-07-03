
# Targeted Systems

- Ubuntu / Debian Systems "like Ubuntu 22.04"

- To check your system version, run the following command:

```bash
uname -m && cat /etc/*release
```

You should also check if your GPU supports ROCm by running the following command:

```bash
clinfo | grep "gfx"
```
This will probably return two values, one being the dedicated GPU and the other being the integrated GPU.
If one of the result values matches any of the following, then your GPU is supported:

- gfx1030
- gfx1100
- gfx900
- gfx906
- gfx908 
- gfx90a
- gfx940
- gfx941
- gfx942

# Prerequisites

Install the following packages:

```bash
sudo apt-get update
sudo apt install "linux-headers-$(uname -r)" "linux-modules-extra-$(uname -r)"
```

And set video permissions to yourself by running the following command:

```bash
sudo usermod -a -G render,video $LOGNAME
```

To set the permissions for all future users, run the following command:

```bash
echo 'ADD_EXTRA_GROUPS=1' | sudo tee -a /etc/adduser.conf
echo 'EXTRA_GROUPS=video' | sudo tee -a /etc/adduser.conf
echo 'EXTRA_GROUPS=render' | sudo tee -a /etc/adduser.conf
``` 
* you may want to re-login to apply the changes.

# Installing ROCm

As given on the AMD ROCm website (https://rocm.docs.amd.com/projects/install-on-linux/en/latest/tutorial/install-overview.html), 
ROCm supports installing from the package manager, or the `amdgpu-install` script.

For me, I first ran the `amdgpu-install` script, but I did not get it working correctly.
But try it out, it might work for you.

## Installing using the package manager

Installation using the package manager is slightly more complicated.

First, add the package signing key using the following command:

> [!IMPORTANT]
> The GPG key may change, so make sure to check the ROCm website for the latest key. This also applies to when you want to update ROCm.

```bash
# Make the directory if it doesn't exist yet.
# This location is recommended by the distribution maintainers.
sudo mkdir --parents --mode=0755 /etc/apt/keyrings

# Download the key, convert the signing-key to a full
# keyring required by apt and store in the keyring directory
wget https://repo.radeon.com/rocm/rocm.gpg.key -O - | \
    gpg --dearmor | sudo tee /etc/apt/keyrings/rocm.gpg > /dev/null
```

Next, add the AMDGPU repository to your system:

```bash
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/amdgpu/6.1.2/ubuntu jammy main" \
    | sudo tee /etc/apt/sources.list.d/amdgpu.list
sudo apt update
```
and also add the ROCm repository:

```bash
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/rocm.gpg] https://repo.radeon.com/rocm/apt/6.1.2 jammy main" \
    | sudo tee --append /etc/apt/sources.list.d/rocm.list
echo -e 'Package: *\nPin: release o=repo.radeon.com\nPin-Priority: 600' \
    | sudo tee /etc/apt/preferences.d/rocm-pin-600
sudo apt update
```

Finally, install the kernel driver and the ROCm package:
> [!IMPORTANT]
> You should reboot your system after installing the kernel driver.

```bash
sudo apt install amdgpu-dkms
sudo reboot
```

```bash
sudo apt install rocm
```

## Post-installation steps
The two last steps include configuring the system linker and adding ROCm to the PATH.

```bash
sudo tee --append /etc/ld.so.conf.d/rocm.conf <<EOF
/opt/rocm/lib
/opt/rocm/lib64
EOF
sudo ldconfig
```
To add ROCm to the PATH, run the following command:
```bash
export PATH=$PATH:/opt/rocm-6.1.3/bin
```
> [!WARNING]
> You should add the PATH to your `.bashrc` or `.zshrc` file to make the changes permanent.
> Make sure the path is correct, as it may change with the ROCm version.

You can now reload the shell by running the following command:

```bash
source ~/.bashrc
```

and check if the kernel-mode driver is installed correctly by running the following command:

```bash
dkms status
```

If the output is similar to the following, then the kernel-mode driver is installed correctly:

```bash
amdgpu/6.7.0-1787201.22.04, 5.15.0-113-generic, amd64: installed
```
To verify the ROCm installation, run the following command:

```bash
/opt/rocm-6.1.3/bin/rocminfo
/opt/rocm-6.1.3/bin/clinfo
```

> [!WARNING]
> Again, check if your version is the same (6.1.3), or different, as it may change with the ROCm version.
> Use the `tab` key to autocomplete the path.


## Installing using the `amdgpu-install` script

To download the `amdgpu-install` script, run the following command:

```bash
sudo apt update
wget https://repo.radeon.com/amdgpu-install/6.1.2/ubuntu/jammy/amdgpu-install_6.1.60102-1_all.deb
sudo apt install ./amdgpu-install_6.1.60102-1_all.deb
```
You can now install the ROCm workload run it by typing `amdgpu-install --usecase=rocm`.
This will install `dkms` (A kernel mode driver), and the ROCm packages used for machine learning.

A full list of all workloads can be found here:

<details>
  <summary>List of all ROCm Workloads</summary>
  
  ```bash
    If --usecase option is not present, the default selection is "graphics,opencl,hip"

    Available use cases:
    dkms            (to only install the kernel mode driver)
    - Kernel mode driver (included in all usecases)
    graphics        (for users of graphics applications)
    - Open source Mesa 3D graphics and multimedia libraries
    multimedia      (for users of open source multimedia)
    - Open source Mesa 3D multimedia libraries
    multimediasdk   (for developers of open source multimedia)
    - Open source Mesa 3D multimedia libraries
    - Development headers for multimedia libraries
    workstation     (for users of legacy WS applications)
    - Open source multimedia libraries
    - Closed source (legacy) OpenGL
    rocm            (for users and developers requiring full ROCm stack)
    - OpenCL (ROCr/KFD based) runtime
    - HIP runtimes
    - Machine learning framework
    - All ROCm libraries and applications
    rocmdev         (for developers requiring ROCm runtime and
                    profiling/debugging tools)
    - HIP runtimes
    - OpenCL runtime
    - Profiler, Tracer and Debugger tools
    rocmdevtools    (for developers requiring ROCm profiling/debugging tools)
    - Profiler, Tracer and Debugger tools
    amf             (for users of AMF based multimedia)
    - AMF closed source multimedia library
    lrt             (for users of applications requiring ROCm runtime)
    - ROCm Compiler and device libraries
    - ROCr runtime and thunk
    opencl          (for users of applications requiring OpenCL on Vega or later
                    products)
    - ROCr based OpenCL
    - ROCm Language runtime
    openclsdk       (for application developers requiring ROCr based OpenCL)
    - ROCr based OpenCL
    - ROCm Language runtime
    - development and SDK files for ROCr based OpenCL
    hip             (for users of HIP runtime on AMD products)
    - HIP runtimes
    hiplibsdk       (for application developers requiring HIP on AMD products)
    - HIP runtimes
    - ROCm math libraries
    - HIP development libraries
    openmpsdk       (for users of openmp/flang on AMD products)
    - OpenMP runtime and devel packages
    mllib           (for users executing machine learning workloads)
    - MIOpen hip/tensile libraries
    - Clang OpenCL
    - MIOpen kernels
    mlsdk           (for developers executing machine learning workloads)
    - MIOpen development libraries
    - Clang OpenCL development libraries
    - MIOpen kernels
    asan            (for users of ASAN enabled ROCm packages)
    - ASAN enabled OpenCL (ROCr/KFD based) runtime
    - ASAN enabled HIP runtimes
    - ASAN enabled Machine learning framework
    - ASAN enabled ROCm libraries
    
  ```
</details>

This should be all for installation using the script!

## Installing tensorflow-rocm for python

> [!WARNING]
> Please keep your other tensorflow installs safe by using a virtual environment.

As of 2024-04-03, the tensorflow-rocm package is not yet updated to the latest fixes, which causes issues on some newer GPUs.

I recommend installing a later version of tensorflow-rocm which can be found here:

http://ml-ci.amd.com:21096/job/tensorflow/job/release-rocmfork-r214-rocm-enhanced/job/release-build-whl/

Download the `.whl` appropriate to your python version (cp39, cp310, cp311 corresponding to python 3.9, 3.10 and 3.11) and install it using pip.

## Using tensorflow-rocm

Import tensorflow as usual and check if the GPU is detected by running the following code:

```python
import tensorflow as tf
print("Num GPUs Available: ", len(tf.config.list_physical_devices('GPU')))
```

If the output is `Num GPUs Available:  1`, then the GPU is detected correctly.

Also, I recommend limiting GPU memory growth by running the following code:

```python

# Limit GPU memory usage to prevent Cinnamon crashes
gpus = tf.config.experimental.list_physical_devices('GPU')
if gpus:
    try:
        # Set memory growth to True to allocate memory as needed
        for gpu in gpus:
            tf.config.experimental.set_memory_growth(gpu, True)
        
        # Optionally, set a fixed memory limit (e.g., 4GB)
        tf.config.experimental.set_virtual_device_configuration(
            gpus[0],
            [tf.config.LogicalDeviceConfiguration(memory_limit=4096)]
        )
    except RuntimeError as e:
        print(e)
```