## 系统安装

下载官网工具就可以直接傻瓜安装

~~~
https://devices.ubuntu-touch.io/installer/exe
~~~

## SSH

设置手机口令密码

使用下面的用户名连接

~~~shell
ssh phablet@<设备的IP地址>
~~~

## 软件源

因为Ubuntu Touch默认使用了一个只读的文件系统，这是为了提高系统的稳定性和安全性。当你尝试修改`/etc/apt/sources.list`文件（或任何系统文件）的权限时，会因为文件系统是只读的而失败。

要修改只读文件系统中的文件，你需要先重新挂载根文件系统为可写模式。请按照以下步骤操作：

### 1. 重新挂载文件系统为可写模式

在终端中输入以下命令：

```bash
sudo mount -o remount,rw /
```

这个命令会要求你输入密码，这里应该是你之前设置的SSH密码。

### 2. 修改文件权限

一旦文件系统被重新挂载为可写，你就可以尝试再次修改文件权限：

```bash
chmod 644 /etc/apt/sources.list
vi /etc/apt/sources.list
:%s/ports.ubuntu.com/mirrors.tuna.tsinghua.edu.cn/g
```

### 3. 恢复文件系统为只读（可选）

完成必要的修改后，为了保持系统的稳定性和安全性，建议将根文件系统重新挂载为只读模式：

```bash
sudo mount -o remount,ro /
```

## 宝塔面板

~~~
wget -O install.sh https://download.bt.cn/install/install-ubuntu_6.0.sh && sudo bash install.sh ed8484bec
~~~

