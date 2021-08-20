# Install docker and nvidia-docker2 on ubuntu18

## Install docker

ref:
https://github.com/CNOCycle/Linux-baseImage/blob/master/install.sh?fbclid=IwAR3JgVx3mM9KSDq1dfYm7X1V_4CVoi7Gb4ommzU9EaVuCbTRUn9Z5s4YVyE

```
# switch user to `root`
sudo -i

apt install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common && \
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add - && \
apt-key fingerprint 0EBFCD88 && \
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable" && \
apt update && \
apt install -y docker-ce docker-ce-cli containerd.io

# test docker
docker version
docker run hello-world
```

## Install nvidia-docker2
```
# switch user to `root`
sudo -i

# install nvidia-docker
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
  apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  tee /etc/apt/sources.list.d/nvidia-docker.list
apt-get update && \
apt install -y nvidia-docker2 && \
pkill -SIGHUP dockerd

# ref: https://github.com/NVIDIA/nvidia-docker/tree/master#upgrading-with-nvidia-docker2-deprecated
# update nvidia-docker2 and restart docker
# On debian based distributions: Ubuntu / Debian
apt-get update
apt-get --only-upgrade install docker-ce nvidia-docker2
systemctl restart docker

# test nvidia-docker
docker run --gpus all nvidia/cuda:9.0-base nvidia-smi
```

## Add (current) user to docker group
```
# change $USER if you want
sudo gpasswd -a $USER docker
```
then re-login the account