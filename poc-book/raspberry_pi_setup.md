# Raspberry Pi 5 Setup

```sh
ssh cavani@192.168.72.123


sudo apt update
sudo apt full-upgrade

sudo raspi-config nonint do_change_locale en_US.UTF-8


# RTC Battery

cat /sys/devices/platform/soc/soc:rpi_rtc/rtc/rtc0/charging_voltage

sudo bash -c 'echo "dtparam=rtc_bbat_vchg=3000000" >> /boot/firmware/config.txt'


# Docker

sudo apt install -y \
--no-install-recommends \
ca-certificates \
curl

sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
"deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
$(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update

sudo apt install -y \
--no-install-recommends \
docker-ce \
docker-ce-cli \
containerd.io \
docker-buildx-plugin \
docker-compose-plugin

sudo groupadd docker
sudo usermod -aG docker $USER
newgrp docker

docker run hello-world


# k3s

cat /proc/cgroups
# memory 0 (enabled)

sudo bash -c 'echo " cgroup_memory=1 cgroup_enable=memory" >> /boot/firmware/cmdline.txt'

sudo reboot

ssh cavani@192.168.72.123

cat /proc/cgroups
# memory 1 (enabled)

curl -sfL https://get.k3s.io | sh -

# After running this installation:
#
# - The K3s service will be configured to automatically restart after node reboots or if the process crashes or is killed
# - Additional utilities will be installed, including kubectl, crictl, ctr, k3s-killall.sh, and k3s-uninstall.sh
# - A kubeconfig file will be written to /etc/rancher/k3s/k3s.yaml and the kubectl installed by K3s will automatically use it

# https://docs.k3s.io/cluster-access
mkdir ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $(id -u):$(id -g) ~/.kube/config
echo 'export KUBECONFIG=$HOME/.kube/config' >> ~/.bashrc

. ~/.bashrc
printenv KUBECONFIG
# /home/cavani/.kube/config

kubectl cluster-info


# Rust

curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- --default-toolchain stable -y

. "$HOME/.cargo/env"

sudo apt install -y \
--no-install-recommends \
libssl-dev

cargo install cargo-edit cargo-update


# Git

sudo apt install -y \
--no-install-recommends \
git \
git-lfs

git config --global user.name "Ciro Cavani"
git config --global user.email "ciro.cavani@gmail.com"

ssh-keygen -t rsa -b 2048 -C P247-ssd -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub
# [...] -> https://github.com/settings/keys

mkdir Garage
cd Garage

git clone git@github.com:cirocavani/programmable-matter-blueprint-1.git
git clone git@github.com:cirocavani/programmable-matter-candle.git
git clone git@github.com:huggingface/candle.git
```

## Audio

```sh
arecord -l
# **** List of CAPTURE Hardware Devices ****
# card 0: Device [USB PnP Sound Device], device 0: USB Audio [USB Audio]
#   Subdevices: 1/1
#   Subdevice #0: subdevice #0

aplay -l
# **** List of PLAYBACK Hardware Devices ****
# card 1: UACDemoV10 [UACDemoV1.0], device 0: USB Audio [USB Audio]
#   Subdevices: 1/1
#   Subdevice #0: subdevice #0
# card 2: vc4hdmi0 [vc4-hdmi-0], device 0: MAI PCM i2s-hifi-0 [MAI PCM i2s-hifi-0]
#   Subdevices: 1/1
#   Subdevice #0: subdevice #0
# card 3: vc4hdmi1 [vc4-hdmi-1], device 0: MAI PCM i2s-hifi-0 [MAI PCM i2s-hifi-0]
#   Subdevices: 1/1
#   Subdevice #0: subdevice #0

alsamixer

# Card USB PnP Sound Device
# F4: Capture
# Mic + 100
# F6: Select sound card
# Card UACDemoV1.0
# F3: Playback
# PCM + 84
# Esc
```

Recording / Playing.

```sh
arecord -D sysdefault:CARD=Device --format=S16_LE --duration=5 --rate=16000 --file-type=wav out.wav

aplay -D sysdefault:CARD=UACDemoV10 --format=S16_LE --rate=16000 out.wav
```

## Camera

<https://www.raspberrypi.com/documentation/computers/camera_software.html>

<https://libcamera.org/getting-started.html#using-gstreamer-plugin>

```sh
sudo apt purge rpicam-apps-lite
sudo apt install \
--no-install-recommends \
rpicam-apps \
gstreamer1.0-tools \
gstreamer1.0-plugins-good \
gstreamer1.0-plugins-bad \
gstreamer1.0-plugins-rtp \
gstreamer1.0-libcamera

rpicam-vid --list-cameras

# Available cameras
# -----------------
# 0 : imx708 [4608x2592 10-bit] (/base/axi/pcie@120000/rp1/i2c@88000/imx708@1a)
#     Modes: 'SBGGR10_CSI2P' : 1536x864 [30.00 fps - (65535, 65535)/65535x65535 crop]
#                              2304x1296 [30.00 fps - (65535, 65535)/65535x65535 crop]
#                              4608x2592 [30.00 fps - (65535, 65535)/65535x65535 crop]
#
# 1 : imx708_noir [4608x2592 10-bit] (/base/axi/pcie@120000/rp1/i2c@80000/imx708@1a)
#     Modes: 'SBGGR10_CSI2P' : 1536x864 [30.00 fps - (65535, 65535)/65535x65535 crop]
#                              2304x1296 [30.00 fps - (65535, 65535)/65535x65535 crop]
#                              4608x2592 [30.00 fps - (65535, 65535)/65535x65535 crop]

# Start Camera Server 1

gst-launch-1.0 libcamerasrc camera-name="/base/axi/pcie@120000/rp1/i2c@88000/imx708@1a" ! \
capsfilter caps=video/x-raw,width=1280,height=720,format=NV12 ! queue ! jpegenc ! multipartmux ! \
tcpserversink host=0.0.0.0 port=5000 \
> cam0.log 2>&1 &

# Start Camera Server 2

gst-launch-1.0 libcamerasrc camera-name="/base/axi/pcie@120000/rp1/i2c@80000/imx708@1a" ! \
capsfilter caps=video/x-raw,width=1280,height=720,format=NV12 ! queue ! jpegenc ! multipartmux ! \
tcpserversink host=0.0.0.0 port=5001 \
> cam1.log 2>&1 &

gst-launch-1.0 tcpclientsrc host=192.168.72.123 port=5000 ! \
multipartdemux ! jpegdec ! autovideosink

gst-launch-1.0 tcpclientsrc host=192.168.72.123 port=5001 ! \
multipartdemux ! jpegdec ! autovideosink
```
