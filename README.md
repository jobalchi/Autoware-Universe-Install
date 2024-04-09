# Autoware Universe (ROS 2 Galactic Ver.) Guide
**Please** refer to the official autoware document for the original installation guide
[Autoware Documentation](https://autowarefoundation.github.io/autoware-documentation/main)

**It reveals that it's based on Autoware Documentation!!**

## Prerequisites
- OS
&nbsp;&nbsp;Ubuntu 20.04
- ROS
&nbsp;&nbsp;Galactic
- Git

```sh
sudo apt-get -y update
sudo apt-get -y install git
```

## Set up a development environment

### Clone autoware
```sh
git clone https://github.com/autowarefoundation/autoware.git -b galactic
```
### Installing dependencies manually
1. [Install ROS 2 Galactic](https://docs.ros.org/en/galactic/Installation/Ubuntu-Install-Debians.html)
```sh
locale  # check for UTF-8

sudo apt update && sudo apt install locales
sudo locale-gen en_US en_US.UTF-8
sudo update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8
export LANG=en_US.UTF-8

locale  # verify settings

sudo apt install software-properties-common
sudo add-apt-repository universe

sudo apt update && sudo apt install curl
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg

echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

sudo apt update
sudo apt upgrade

sudo apt install ros-galactic-desktop
sudo apt install ros-dev-tools

# Add ~/.bashrc
source /opt/ros/galactic/setup.bash
```
2. [Install  ROS  2 Dev Tools](https://github.com/autowarefoundation/autoware/tree/galactic/ansible/roles/ros2_dev_tools)
```sh
# Taken from https://docs.ros.org/en/galactic/Installation/Ubuntu-Development-Setup.html
sudo apt update && sudo apt install -y \
  build-essential \
  cmake \
  git \
  python3-colcon-common-extensions \
  python3-flake8 \
  python3-pip \
  python3-pytest-cov \
  python3-rosdep \
  python3-setuptools \
  python3-vcstool \
  wget

# Install some pip packages needed for testing
python3 -m pip install -U \
  flake8-blind-except \
  flake8-builtins \
  flake8-class-newline \
  flake8-comprehensions \
  flake8-deprecated \
  flake8-docstrings \
  flake8-import-order \
  flake8-quotes \
  pytest-repeat \
  pytest-rerunfailures \
  pytest \
  setuptools

# Initialize rosdep
sudo rosdep init
rosdep update
```
3. [Install the RMW Implementation](https://github.com/autowarefoundation/autoware/tree/galactic/ansible/roles/rmw_implementation)
```sh
wget -O /tmp/amd64.env https://raw.githubusercontent.com/autowarefoundation/autoware/galactic/amd64.env && source /tmp/amd64.env

# For details: https://docs.ros.org/en/galactic/How-To-Guides/Working-with-multiple-RMW-implementations.html
sudo apt update
rmw_implementation_dashed=$(eval sed -e "s/_/-/g" <<< "${rmw_implementation}")
sudo apt install ros-${rosdistro}-${rmw_implementation_dashed}

# (Optional) You set the default RMW implementation in the ~/.bashrc file.
echo '' >> ~/.bashrc && echo "export RMW_IMPLEMENTATION=${rmw_implementation}" >> ~/.bashrc
```
4. [Install pacmod](https://github.com/autowarefoundation/autoware/tree/galactic/ansible/roles/pacmod)
```sh
# Taken from https://github.com/astuff/pacmod3#installation
sudo apt install apt-transport-https
sudo sh -c 'echo "deb [trusted=yes] https://s3.amazonaws.com/autonomoustuff-repo/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/autonomoustuff-public.list'
sudo apt update
sudo apt install ros-${rosdistro}-pacmod3
```
5. [Install Autoware Core dependencies](https://github.com/autowarefoundation/autoware/tree/galactic/ansible/roles/autoware_core)
```sh
# Install gdown to download files from CMakeLists.txt
pip3 install gdown
```
6. [Install Autoware Universe dependencies](https://github.com/autowarefoundation/autoware/tree/galactic/ansible/roles/autoware_universe)
```sh
sudo apt install geographiclib-tools

# Add EGM2008 geoid grid to geographiclib
sudo geographiclib-get-geoids egm2008-1
```
7. [Install pre-commit dependencies](https://github.com/autowarefoundation/autoware/tree/galactic/ansible/roles/pre_commit)
```sh
clang_format_version=14.0.6
pip3 install pre-commit clang-format==${clang_format_version}

# Install Golang (Add Go PPA for shfmt)
sudo add-apt-repository ppa:longsleep/golang-backports
sudo apt install golang
```
8. [Install Nvidia CUDA](https://github.com/autowarefoundation/autoware/tree/galactic/ansible/roles/cuda)
```sh
# Modified from:
# https://developer.nvidia.com/cuda-11-4-4-download-archive?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=20.04&target_type=deb_network
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin
sudo mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600
sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/3bf863cc.pub
sudo add-apt-repository "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/ /"
sudo apt-get update
wget https://developer.download.nvidia.com/compute/cuda/11.6.0/local_installers/cuda_11.6.0_510.39.01_linux.run
sudo sh cuda_11.6.0_510.39.01_linux.run

# Add ~/.bashrc
export PATH=/usr/local/cuda-11.6/bin${PATH:+:${PATH}}
export LD_LIBRARY_PATH=/usr/local/cuda-11.6/lib64:${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
```
9. [Install Nvidia cuDNN and TensorRT](https://github.com/autowarefoundation/autoware/tree/galactic/ansible/roles/tensorrt)
```sh
# Taken from: https://docs.nvidia.com/deeplearning/tensorrt/install-guide/index.html#installing

[# Install cudnn-linux-x86_64-8.4.1.50_cuda11.6-archive.tar.xz](https://developer.nvidia.com/rdp/cudnn-archive)
unxz cudnn-linux-x86_64-8.4.1.50_cuda11.6-archive.tar.xz
tar -xvf cudnn-linux-x86_64-8.4.1.50_cuda11.6-archive.tar

sudo cp cudnn-linux-x86_64-8.4.1.50_cuda11.6-archive/include/* /usr/local/cuda-11.6/include
sudo cp -P cudnn-linux-x86_64-8.4.1.50_cuda11.6-archive/lib/* /usr/local/cuda-11.6/lib64
sudo chmod a+r /usr/local/cuda-11.6/lib64/libcudnn*

sudo apt-get install libcudnn8=${cudnn_version} libcudnn8-dev=${cudnn_version}
sudo apt-mark hold libcudnn8 libcudnn8-dev

sudo apt-get install libnvinfer8=${tensorrt_version} libnvonnxparsers8=${tensorrt_version} libnvparsers8=${tensorrt_version} libnvinfer-plugin8=${tensorrt_version} libnvinfer-dev=${tensorrt_version} libnvonnxparsers-dev=${tensorrt_version} libnvparsers-dev=${tensorrt_version} libnvinfer-plugin-dev=${tensorrt_version}
sudo apt-mark hold libnvinfer8 libnvonnxparsers8 libnvparsers8 libnvinfer-plugin8 libnvinfer-dev libnvonnxparsers-dev libnvparsers-dev libnvinfer-plugin-dev
```

## Set up a workspace
1. Create the src directory and clone repositories into it
```sh
cd autoware
mkdir -p src
vcs import src < autoware.repos
```
[**Install Lanelet2 lib from github!**](https://github.com/fzi-forschungszentrum-informatik/Lanelet2)

2. Install dependent ROS & External packages
```sh
sudo apt install ros-galactic-diagnostic-updater ros-galactic-osrf-testing-tools-cpp ros-galactic-ros-testing ros-galactic-tensorrt-cmake-module ros-galactic-tvm-vendor ros-galactic-rqt-robot-monitor ros-galactic-pcl-ros ros-galactic-radar-msgs ros-galactic-xacro ros-galactic-topic-tools ros-galactic-octomap ros-galactic-behaviortree-cpp-v3 ros-galactic-rosbridge-server ros-galactic-filters ros-galactic-nav2-msgs ros-galactic-geographic-msgs ros-galactic-point-cloud-msg-wrapper ros-galactic-usb-cam ros-galactic-nav2-costmap-2d ros-galactic-octomap-msgs ros-galactic-osqp-vendor ros-galactic-velodyne-pointcloud ros-galactic-ament-clang-format ros-galactic-diagnostic-aggregator ros-galactic-mrt-cmake-modules ros-galactic-cudnn-cmake-module ros-galactic-rqt-runtime-monitor ros-galactic-ublox-gps ros-galactic-osrf-testing-tools-cpp
```
```sh
sudo apt install geographiclib-tools libgeographic-dev librange-v3-dev libpcap-dev libcpprest-dev libfmt-dev libcgal-dev libnl-genl-3-dev libpugixml-dev nlohmann-json3-dev
```
3. Build the workspace
```sh
colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release
```
