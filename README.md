# Working configurations for OMNISCI + Jupyter Free (CPU / GPU)

### OMNISCI's documentation is pretty good but there are a few errors.

If going CPU route, you probably want minimum of 8 cores and 32gigs memory.
For root disk you want at least 50gb to keep docker happy (or you can separate that partition out)
GPU depends on what you have of course, but double precision and memory is important (single GPU, up to 32gb). Something like a T4 (16gb) works well.

These instructions assume Ubuntu 20.04 LTS. GPU steps can be ignored for CPU.


### Install needed packages
```
sudo apt update
sudo apt upgrade
sudo apt -y install apt-transport-https ca-certificates curl gnupg-agent software-properties-common gcc
```

### Install CUDA (GPU only)
```
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin
sudo mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/11.4.2/local_installers/cuda-repo-ubuntu2004-11-4-local_11.4.2-470.57.02-1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu2004-11-4-local_11.4.2-470.57.02-1_amd64.deb
sudo apt-key add /var/cuda-repo-ubuntu2004-11-4-local/7fa2af80.pub
sudo apt-get update
sudo apt-get -y install cuda
sudo apt install nvidia-utils-470
sudo apt update
```

### Install Docker
```
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt update
sudo apt-get install docker-ce docker-ce-cli containerd.io
sudo usermod -aG docker $USER
```


### Install docker compose (for jupyterhub version)
```
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```


### Install nvidia runtime (GPU)
```
curl -s -L https://nvidia.github.io/nvidia-container-runtime/gpgkey | \
  sudo apt-key add -  

distribution=$(. /etc/os-release;echo $ID$VERSION_ID)

curl -s -L https://nvidia.github.io/nvidia-container-runtime/$distribution/nvidia-container-runtime.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-runtime.list

sudo apt-get update
sudo apt-get install -y nvidia-container-runtime
```

### Run this as root (GPU runtime)
```
cat << EOF >> /etc/docker/daemon.json
{
 "default-runtime": "nvidia",
  "runtimes": {
     "nvidia": {
         "path": "/usr/bin/nvidia-container-runtime",
         "runtimeArgs": []
     }
 }
}
EOF

sudo pkill -SIGHUP dockerd
sudo docker run --runtime=nvidia --rm nvidia/cuda:11.0-runtime-ubuntu20.04 nvidia-smi
```



### Configure your /var/lib/omnisci as wherever you want the storage to run
I run it on a separate XFS volume than my OS, size depends on how much data you're planning on but I suggest at least 100gb


### I use cloudflare for these services, but you can do whatever suits your needs.
https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/tunnel-guide

```
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb
```


### Run regular omnisci (GPU,  no jupyter)
```
sudo docker run --runtime=nvidia \
  -d --runtime=nvidia \
  -v /var/lib/omnisci:/omnisci-storage \
  -p 6273-6280:6273-6280 \
  omnisci/omnisci-ee-cuda:v5.8.0
```


