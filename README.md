# Install Driverless AI on Ubuntu w GPUs

## Environment

- Ubuntu with GPUs, Min Mem is 64GB
- Ubuntu 18.04, g4dn.4xlarge
- We will install CUDA 11 and during this install it will install NVIDIA >= 440.82 Drivers and 
- We will install cuDNN >= 7.2.1 (for TensorFlow on GPUs)
- We will install OpenCL (for LightGBM on GPUs)
- We will install Driverless AI DEB

## Install CUDA 11 Toolkit and Drivers

### Pre-installation Actions

- Verify the system has a CUDA-capable GPU.
- Verify the system is running a supported version of Linux. `Ubuntu 18.04.z (z <= 4)`
- Verify the system has gcc installed. `type gcc`, `gcc version 8.2.0`
- Verify the system has the correct kernel headers and development packages installed. ?, `4.15.0`
- Download the NVIDIA CUDA Toolkit.
- Handle conflicting installation methods.

2.1\. Verify You Have a CUDA-Capable GPU

To verify that your GPU is CUDA-capable, go to your distribution's equivalent of System Properties:

~~~bash
lspci | grep -i nvidia
~~~

If you do not see any settings, update the PCI hardware database that Linux maintains by entering `update-pciids` (generally found in `/sbin`) at the command line and rerun the previous `lspci` command.

If your graphics card is from NVIDIA and it is listed in https://developer.nvidia.com/cuda-gpus, your GPU is CUDA-capable.

2.2\. Verify You Have a Supported Version of Linux

Determine which distribution and release number you're running:

~~~bash
uname -m && cat /etc/*release
~~~

2.3\. Verify the System Has gcc Installed

Verify the version of gcc installed on your system:

~~~bash
gcc --version
~~~

If an error message displays, you need to install the development tools from your Linux distribution or obtain a version of `gcc` and its accompanying toolchain from the Web.

2.4\. Verify the System has the Correct Kernel Headers and Development Packages Installed

Get the version of the kernel your system is running:

~~~bash
uname -r
~~~

Install the kernel headers and development packages for the currently running kernel:

~~~bash
sudo apt-get -y update
sudo apt-get -y install linux-headers-$(uname -r)
~~~

2.5\. Check if there are conflicting installation methods

Before installing CUDA, any previously installations that could conflict should be uninstalled. This will not affect systems which have not had CUDA installed previously, or systems where the installation method has been preserved (RPM/Deb vs. Runfile).

Use the following command to uninstall a Toolkit runfile installation:

~~~bash
sudo /usr/local/cuda-X.Y/bin/uninstall_cuda_X.Y.pl
~~~

Use the following command to uninstall a Driver runfile installation:

~~~bash
sudo /usr/bin/nvidia-uninstall
~~~

Use the following commands to uninstall a RPM/Deb installation:

~~~bash
sudo apt-get --purge remove <package_name>
~~~

### Packager Manager Installation: Install CUDA for Ubuntu with DEB Local

2.5, 2.6 Download NVIDIA CUDA Toolkit

- Install CUDA 10 or later driveres and NVIDIA drivers

1\. Install CUDA 11 with Debian Installer

Install the repository meta-data, update the apt-get cache, and install CUDA:

~~~bash
# Pin file to prioritize CUDA repository
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-ubuntu1804.pin
sudo mv cuda-ubuntu1804.pin /etc/apt/preferences.d/cuda-repository-pin-600
# Download repository meta-data
wget https://developer.download.nvidia.com/compute/cuda/11.0.3/local_installers/cuda-repo-ubuntu1804-11-0-local_11.0.3-450.51.06-1_amd64.deb

# Verify repo meta-data download
# Expected Checksum for cuda-repo-ubuntu1804-11-0-local_11.0.3-450.51.06-1_amd64.deb
# 9ff7717492e88850db704e47c542bbf0

md5sum cuda-repo-ubuntu1804-11-0-local_11.0.3-450.51.06-1_amd64.deb

# Install repository meta-data
sudo dpkg -i cuda-repo-ubuntu1804-11-0-local_11.0.3-450.51.06-1_amd64.deb
# Installing the CUDA public GPG key; When installing using the local repo
sudo apt-key add /var/cuda-repo-ubuntu1804-11-0-local/7fa2af80.pub
# Installs all CUDA Toolkit and Driver packages.
sudo apt-get -y update
sudo apt-get -y install cuda
~~~

2\. Reboot the system to load the NVIDIA drivers

~~~bash
# version of nvidia version 450.51.06
nvidia-smi
~~~

### Post-installation Actions

Some actions must be taken after the installation before the CUDA Toolkit and Driver can be used.

3\. Set up the development environment by modifying the PATH and LD_LIBRARY_PATH variables:

~~~bash
export PATH=/usr/local/cuda-11.0/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-11.0/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
~~~

4\. Verify the driver version

When the driver is loaded, the driver version can be found:

~~~bash
cat /proc/driver/nvidia/version
~~~

5\. Verify CUDA Toolkit version

The nvcc command runs the compiler driver that compiles CUDA programs. It calls the gcc compiler for C code and the NVIDIA PTX compiler for the CUDA code.

~~~bash
nvcc -V
~~~

On systems where SELinux is enabled, you might need to temporarily disable this security feature to run deviceQuery.

~~~bash
cd ~/NVIDIA_CUDA-11.0_Samples
make
./deviceQuery
~~~

The exact appearance and the output lines might be different on your system. The important outcomes are that a device was found (the first highlighted line), that the device matches the one on your system (the second highlighted line), and that the test passed (the final highlighted line).

Running the bandwidthTest program ensures that the system and the CUDA-capable device are able to communicate correctly.

~~~bash
cd ~/NVIDIA_CUDA-11.0_Samples
./bandwidthTest
~~~

Note that the measurements for your CUDA-capable device description will vary from system to system. The important point is that you obtain measurements, and that the second-to-last line (in Figure 2) confirms that all necessary tests passed.

### Optional: Removing CUDA Toolkit and Driver

Remove CUDA Toolkit:

~~~bash
sudo apt-get --purge remove "*cublas*" "*cufft*" "*curand*" \
 "*cusolver*" "*cusparse*" "*npp*" "*nvjpeg*" "cuda*" "nsight*"
~~~

Remove NVIDIA Drivers:

~~~bash
sudo apt-get --purge remove "*nvidia*"
~~~

## Install cuDNN (for TensorFlow)

- required to have installed NVIDIA Graphic Drivers
- required to have installed CUDA Toolkit. Done.

version 7.2.1

~~~bash
# Download cuDNN onto local, then send it to ec2
# cuDNN Runtime Library
# from local to ec2
scp -i $DAI_PEM $HOME/Downloads/libcudnn8_8.0.3.33-1+cuda11.0_amd64.deb ubuntu@$DAI_INSTANCE:/home/ubuntu

# cuDNN Developer Library
# wget https://developer.nvidia.com/compute/machine-learning/cudnn/secure/8.0.3.33/11.0_20200825/Ubuntu18_04-x64/libcudnn8-dev_8.0.3.33-1%2Bcuda11.0_amd64.deb
# from local to ec2
scp -i $DAI_PEM $HOME/Downloads/libcudnn8-dev_8.0.3.33-1+cuda11.0_amd64.deb ubuntu@$DAI_INSTANCE:/home/ubuntu

# Install cuDNN from a debian file
# install the runtime library
sudo dpkg -i libcudnn8_8.0.3.33-1+cuda11.0_amd64.deb

# install the developer library
sudo dpkg -i libcudnn8-dev_8.0.3.33-1+cuda11.0_amd64.deb
~~~

Note: if you download the cuDNN code samples, you can verify the cuDNN install was successful

## Starting NVIDIA Persistence Mode (GPU only)

If you have NVIDIA GPUs, you must run the following NVIDIA command. This command needs to be run every reboot. For more information: http://docs.nvidia.com/deploy/driver-persistence/index.html.

~~~bash
# this command didn't work as ubuntu user, does it work as root user?
sudo nvidia-persistenced --persistence-mode

# if it fails to initialize, check if nvidia-persistenced mode is running
sudo systemctl status nvidia-persistenced

# enable nvidia-persistenced
# sudo systemctl enable nvidia-persistenced

# reboot for execution
# sudo reboot

# alternative command to run as root user. check nvidia-smi --help for -pm arg
# sudo -i
# nvidia-smi -pm 1
~~~

**Reference**:

- Nvidia Dev Forum: [Setting up nvidia-persistenced](https://forums.developer.nvidia.com/t/setting-up-nvidia-persistenced/47986/10)

## Install OpenCL

OpenCL is required in order to run LightGBM on GPUs. Run the following for Ubuntu-based ystems.

~~~bash
sudo apt-get -y install opencl-headers clinfo ocl-icd-opencl-dev

mkdir -p /etc/OpenCL/vendors && \
    echo "libnvidia-opencl.so.1" > /etc/OpenCL/vendors/nvidia.icd
~~~

## Installing the Driverless AI Linux DEB

~~~bash
# Download Driverless AI Latest Stable 1.9
wget https://s3.amazonaws.com/artifacts.h2o.ai/releases/ai/h2o/dai/rel-1.9.0-11/x86_64-centos7/dai_1.9.0.2_amd64.deb
# Install Driverless AI.
sudo dpkg -i dai_1.9.0.2_amd64.deb
~~~

## Starting Driverless AI

Use systemd to start Driverless AI

~~~bash
# Start Driverless AI.
sudo systemctl start dai
~~~

## Debugging Driverless AI looking at log files

Use systemd to look at Driverless AI log files

~~~bash
sudo systemctl status dai-dai
sudo systemctl status dai-h2o
sudo systemctl status dai-procsy
sudo systemctl status dai-vis-server
sudo journalctl -u dai-dai
sudo journalctl -u dai-h2o
sudo journalctl -u dai-procsy
sudo journalctl -u dai-vis-server
~~~

## Open Driverless AI in the browser

https://ec2-13-57-56-230.us-west-1.compute.amazonaws.com:12345

## Stopping Driverless AI

~~~bash
# Stop Driverless AI.
sudo systemctl stop dai
~~~