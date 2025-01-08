



* PVE版本8.2.2

* 宿主机开启显卡虚拟化

```shell
sed -i 's|^deb http://ftp.debian.org|deb https://mirrors.ustc.edu.cn|g' /etc/apt/sources.list
sed -i 's|^deb http://security.debian.org|deb https://mirrors.ustc.edu.cn/debian-security|g' /etc/apt/sources.list
source /etc/os-release
echo "deb https://mirrors.ustc.edu.cn/proxmox/debian/pve $VERSION_CODENAME pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list 
apt update
apt install build-essential dkms mdevctl pve-headers-$(uname -r)
```

nano /etc/default/grub

```shell
wangergou@llxspace:~/workspace$ cat /etc/default/grub | grep GRUB_CMDLINE_LINUX_DEFAULT
# GRUB_CMDLINE_LINUX_DEFAULT="quiet"
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt"
wangergou@llxspace:~/workspace$ 
```

```shell
echo vfio >> /etc/modules
echo vfio_iommu_type1 >> /etc/modules
echo vfio_pci >> /etc/modules
echo vfio_virqfd >> /etc/modules
echo "blacklist nouveau" >> /etc/modprobe.d/blacklist-nouveau.conf
echo "blacklist i2c_nvidia_gpu" >> /etc/modprobe.d/blacklist-i2c-nvidia-gpu.conf
echo "options nouveau modeset=0" >> /etc/modprobe.d/blacklist-nouveau.conf

update-initramfs -u -k all
reboot 
```

```shell
wget https://alist.homelabproject.cc/d/foxipan/vGPU/17.3/NVIDIA-Linux-x86_64-550.90.05-vgpu-kvm-patched.run
chmod +x NVIDIA-Linux-x86_64-550.90.05-vgpu-kvm-patched.run 
./NVIDIA-Linux-x86_64-550.90.05-vgpu-kvm-patched.run 
reboot 
```

```shell
nvidia-smi
mdevctl types | grep Name
```

* ubuntu22.04虚拟机安装vgpu

~~~shell
sudo apt install git build-essential dkms mdevctl linux-headers-$(uname -r)
echo "blacklist nouveau" >> /etc/modprobe.d/blacklist-nouveau.conf
sudo update-initramfs -u -k all
reboot
chmod +x NVIDIA-Linux-x86_64-550.90.07-grid.run 
./NVIDIA-Linux-x86_64-550.90.07-grid.run 
reboot
nvidia-smi
~~~

* docker搭建授权服务器

~~~shell
# 拉取镜像
docker pull collinwebdesigns/fastapi-dls:latest
# 创建目录
mkdir -p /opt/fastapi-dls/cert
cd /opt/fastapi-dls/cert
# 生成公私钥
openssl genrsa -out /opt/fastapi-dls/cert/instance.private.pem 2048 
openssl rsa -in /opt/fastapi-dls/cert/instance.private.pem -outform PEM -pubout -out /opt/fastapi-dls/cert/instance.public.pem
# 生成SSL证书
openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -keyout  /opt/fastapi-dls/cert/webserver.key -out /opt/fastapi-dls/cert/webserver.crt
# 创建容器, 1825是授权天数, YOUR_IP处填VM / LXC的IP
docker volume create dls-db
docker run -d --restart=always -e LEASE_EXPIRE_DAYS=1825 -e DLS_URL=<YOUR_IP> -e DLS_PORT=443 -p 443:443 -v /opt/fastapi-dls/cert:/app/cert -v dls-db:/app/database collinwebdesigns/fastapi-dls:latest
~~~

* window授权，使用管理员权限的Powershell执行

~~~shell
#192.168.3.200:443替换为你自己的API地址
curl.exe --insecure -L -X GET https://192.168.3.200:443/-/client-token -o "C:\Program Files\NVIDIA Corporation\vGPU Licensing\ClientConfigToken\client_configuration_token_$($(Get-Date).tostring('dd-MM-yy-hh-mm-ss')).tok"
#重启NV相关服务
Restart-Service NVDisplay.ContainerLocalSystem
~~~

* linux授权

~~~shell
#192.168.3.200:443替换为你自己的API地址
curl --insecure -L -X GET https://192.168.3.200:443/-/client-token -o /etc/nvidia/ClientConfigToken/client_configuration_token_$(date '+%d-%m-%Y-%H-%M-%S').tok

sudo systemctl restart nvidia-gridd

nvidia-smi -q | grep License
~~~

* docker使用vgpu

~~~shell
sudo curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://nvidia.github.io/libnvidia-container/stable/deb/amd64 /" | sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

sudo apt-get update

sudo apt-get install -y nvidia-container-toolkit

~~~

cat /etc/docker/daemon.json

~~~sh
{
    "registry-mirrors": [
        "https://docker.1ms.run"
    ],
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    },
    "default-runtime": "nvidia"
}

~~~

~~~shell
sudo systemctl restart docker
docker run --rm --gpus all nvcr.io/nvidia/cuda:12.4.0-base-ubuntu22.04 nvidia-smi
~~~

* whisper模型使用vgpu

~~~sh
docker run -d --gpus all -p 9000:9000 -e ASR_MODEL=small -e ASR_ENGINE=openai_whisper onerahmet/openai-whisper-asr-webservice:latest-gpu
~~~

