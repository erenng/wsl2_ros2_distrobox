# wsl2_ros2_distrobox
wsl2 podman distrobox jazzy ros2 setup

Creating development environment
-----------------------
The following stages are followed for creating the development environment.
Create WSL2 host (ubuntu:24.04.1)
Before creating wsl2, crate %USERPROFILE%/.wslconfig.txt with the following content
```ini
[wsl2]
guiApplications=true
packagedGpuLibs=false
```
WSL install reference: https://learn.microsoft.com/en-us/windows/wsl/install
```ini 
# Windows PowerShell, installing Windows Terminal from Store is recommended for seamless integration of both linux and windows terminals
wsl --set-default-version 2 # Set to default version to WSL2
wsl --update # Update to the more recent kernel, else graphics integration might fail
wsl --install -d Ubuntu-24.04 # install desired distro
```
After installing, the following installations made
```ini 
sudo rm -rf /var/cache/snapd/
sudo apt autoremove --purge snapd
rm -rf ~/snap
sudo apt-mark hold snapd
sudo apt update; sudo apt install -y htop terminator tmux curl wget git gedit vim net-tools evince nautilus ncdu ccache ethtool qtcreator qtbase5-dev qt5-qmake cmake mesa-utils inxi pcl-tools podman
mkdir -p ~/WSs/3rdParty/; cd ~/WSs/3rdParty/; git clone https://github.com/89luca89/distrobox.git; cd distrobox/; sudo ./install
mkdir -p ~/WSs/dbx
```
Install nvidia toolkit and generate cdi using the following command flow. ref: container-toolkit/install-guide
```ini 
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg   && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list |     sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' |     sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt update
sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
nvidia-ctk cdi list
```
Further customizations to host:
Host .bashrc:
```ini
# CUSTOM Additions
export DBX_CONTAINER_HOME_PREFIX=$HOME/WSs/dbx
export TMUX_TMPDIR=/var/tmp
export GALLIUM_DRIVER=d3d12
export MESA_D3D12_DEFAULT_ADAPTER_NAME=NVIDIA #INTEL
export DISPLAY=:0
```
Create an integrated distrobox container
Create an integrated distrobox container, tightly coupled with the host. Port can be customized for running multiple containers that is accessible from VSCode via ssh. Additionaly, <host_username_here> and <container_user_password_here> needs to be replaced with a suitable user password for ssh security (later, ssh key pair flow can also be used and ssh via password can be disabled).
```ini
distrobox create --name jazzy --image docker.io/osrf/ros:jazzy-desktop-full --unshare-process  --additional-flags "--device nvidia.com/gpu=all --group-add keep-groups -e DISPLAY=$DISPLAY -v /tmp/.X11-unix:/tmp/.X11-unix -v /usr/lib/wsl:/usr/lib/wsl --security-opt label=type:container_runtime_t"  --additional-packages "vim git htop tmux wget curl net-tools ssh ncdu ccache inxi plocate mesa-utils libpcl-dev libxext-dev libx11-dev libglvnd-dev libglx-dev libgl1-mesa-dev libglx-mesa0 libgl1-mesa-dri libegl1-mesa-dev libgles2-mesa-dev freeglut3-dev mesa-utils-extra  ros-jazzy-ros-gz ros-jazzy-gz-ros2-*" --init --init-hooks "echo 'Port 2233' >> /etc/ssh/sshd_config; echo "<host_username_here>:<container_user_password_here>" | chpasswd"
```
Manual additions to the development environment
Bashrc changes:
```ini
export TMUX_TMPDIR=/var/tmp
export GALLIUM_DRIVER=d3d12
export MESA_D3D12_DEFAULT_ADAPTER_NAME=NVIDIA #INTEL
```
Create .inputrc file in home for arrow history:
```ini
# Respect default shortcuts.
$include /etc/inputrc

## arrow up
"\e[A":history-search-backward
## arrow down
"\e[B":history-search-forward
```
