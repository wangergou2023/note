



## 基础

### make

* make -n 
  在编译的时候加上了-n的参数
  执行该命令编译并不真正的执行，只是顺序的列出该编译过程都执行哪些编译命令。
* V=1
  Linux 编译的时候也可以通过“V=1”来输出完整的命令，这个和uboot 一样

* make -f ./scripts/Makefile.build obj=
  在Kbuild系统中，Makefile.build文件算是最重要的文件之一了，它控制着整个内核的核心编译部分.
  参考文章：https://zhuanlan.zhihu.com/p/362958145

### tail

~~~sh
tail -f /tmp/messages
~~~

在 Linux 和 Unix 系统中，`tail -f` 命令用于实时查看文件内容的更新。当您运行 `tail -f /tmp/messages` 时，它会实时显示 `/tmp/messages` 文件的末尾内容，并且当文件有新内容添加时，这些内容会立即显示在您的终端上。

### watch

~~~
watch -n 1 pwd
~~~

`watch` 是一个在 Linux 和 Unix 系统中常用的命令，用于定期执行指定的命令并显示其输出。您提供的命令 `watch -n 1 pwd` 的含义是：

- `watch`: 这是调用 `watch` 命令本身。
- `-n 1`: 这个选项告诉 `watch` 每隔 1 秒刷新一次。
- `pwd`: 这是您想要定期执行的命令。`pwd`（print working directory）是一个显示当前工作目录的命令。

### awk

awk '{print $3}'



### 打补丁

* 单个文件

~~~
diff -up diff/kernel/drivers/sstar/crypto/mdrv_aes.c kernel/drivers/sstar/crypto/mdrv_aes.c > 0017-sha256-clone-exception.patch
~~~

* 文件夹

~~~
diff -uprN diff_dir/mbedtls-2.25.0/library/ mbedtls-2.25.0/library/ > library.patch
~~~

应用补丁

~~~
patch -p1 < 补丁
~~~



### 火焰图

#### 内核配置选项

建议厂家提供

#### 替换驱动

使用内核重新编译驱动

#### 环境变量

* pc编译
  * 用ubuntu虚拟机，可以联网

* board运行
  * export LD_LIBRARY_PATH=你的perf动态库路径:$LD_LIBRARY_PATH

#### 工具编译

* 源码
  * kernel源码：厂家提供的
  * libunwind源码:https://github.com/libunwind/libunwind/archive/refs/tags/v1.5.0.tar.gz
  * zlib源码:https://zlib.net/current/zlib-1.3.tar.gz
* 指令

~~~sh
#!/bin/sh

set -e #显示指令详细执行过程

BUILD_DIR=$PWD
INSTALL_DIR=$PWD/install

export ARCH=arm 
export CROSS_COMPILE=arm-sigmastar-linux-uclibcgnueabihf-

export CC=arm-sigmastar-linux-uclibcgnueabihf-gcc
export CXX=arm-sigmastar-linux-uclibcgnueabihf-g++
export LD=arm-sigmastar-linux-uclibcgnueabihf-ld

cd $BUILD_DIR
cd libunwind-1.5.0
autoreconf -i
./configure -host=arm-sigmastar-linux-uclibcgnueabihf --enable-shared --disable-static --prefix=$INSTALL_DIR
make clean
make -j8
make install

# cd $BUILD_DIR
# cd zlib-1.3
# cmake -DUSE_SHARED_MBEDTLS_LIBRARY=ON -DUSE_STATIC_MBEDTLS_LIBRARY=ON -DCMAKE_INSTALL_PREFIX=$INSTALL_DIR
# cmake --build . --target install


cd $BUILD_DIR
cd kernel/tools/perf
make clean
make DEBUG=1 LIBUNWIND_DIR=$INSTALL_DIR

# Makefile:215: Please install asciidoc xmlto to have the man pages installed
# sudo apt-get install asciidoc xmlto
~~~

* 输出

根据下面的输出可看到 **libunwind: [ on  ]**

~~~
Auto-detecting system features:
...                         dwarf: [ OFF ]
...            dwarf_getlocations: [ OFF ]
...                         glibc: [ on  ]
...                        libbfd: [ OFF ]
...                libbfd-buildid: [ OFF ]
...                        libcap: [ OFF ]
...                        libelf: [ on  ]
...                       libnuma: [ OFF ]
...        numa_num_possible_cpus: [ OFF ]
...                       libperl: [ OFF ]
...                     libpython: [ OFF ]
...                     libcrypto: [ OFF ]
...                     libunwind: [ on  ]
...            libdw-dwarf-unwind: [ OFF ]
...                          zlib: [ on  ]
...                          lzma: [ OFF ]
...                     get_cpuid: [ OFF ]
...                           bpf: [ on  ]
...                        libaio: [ OFF ]
...                       libzstd: [ OFF ]
...        disassembler-four-args: [ OFF ]
~~~

* 重要依赖
  * **libunwind**: libunwind 用于支持用户空间和内核空间的栈回溯和跟踪功能。这对于性能分析和调试非常有用，因为它允许你追踪函数调用链并查找性能瓶颈。

#### 工具指令

* 设备执行

  * 建议监控单个进程

    ~~~
    # ./perf record -a -g -p 1105 sleep 15
    Warning:
    PID/TID switch overriding SYSTEM
    Couldn't synthesize bpf events.
    Couldn't synthesize cgroup events.
    [ perf record: Woken up 16 times to write data ]
    [ perf record: Captured and wrote 3.865 MB perf.data (43702 samples) ]
    # ls
    lib            lib.tar.gz     p1             perf           perf.data
    # ./perf script -i perf.data > p1
    # 
    
    ~~~

  * 监控所有进程

  ~~~
  # ./perf record -a -g sleep 60
  [ perf record: Woken up 89 times to write data ]
  [ perf record: Captured and wrote 22.128 MB perf.data (198027 samples) ]
  # ls
  perf       perf.data
  # ls -lh
  total 24M    
  -rwxrwxrwx    1 1003     1003        1.9M Dec  5 10:43 perf
  -rw-------    1 root     root       22.1M Dec  5 20:20 perf.data
  # ./perf script -i perf.data > p1
  
  ~~~

* 主机执行

  * 下载生成火焰图的工具FlameGraph
  * 工具地址[Releases · brendangregg/FlameGraph (github.com)](https://github.com/brendangregg/FlameGraph/releases)
  * 下载成功后将固件端生成的p1文件拷贝到目录中
  * 顺序执行命令生成火焰图文件

  ~~~
  wangyongchao@localhost:~/workspace/perf/FlameGraph-1.0$ perl stackcollapse-perf.pl p1 > perf.folder
  wangyongchao@localhost:~/workspace/perf/FlameGraph-1.0$ perl flamegraph.pl perf.folder > perf.svg
  wangyongchao@localhost:~/workspace/perf/FlameGraph-1.0$ 
  ~~~


## 网络

### iw

* 查询接口状态，是否当前有连接

~~~sh
# ./iw dev wlan0 link
Connected to 06:0d:9e:bc:a9:87 (on wlan0)
        SSID: lab-Office
        freq: 5180
        signal: -30 dBm
        tx bitrate: 72.2 MBit/s
# 
~~~

* 扫描热点

~~~sh
# ./iw dev wlan0 scan |grep SSID:
        SSID: lab-Office
        SSID: lab-guest
        SSID: 1234

~~~

### ip查看连接的设备

~~~sh
# ip neigh show dev wlan1
192.168.16.10 lladdr b4:c2:e0:b1:55:2d used 93269/0/93269 probes 0 PERMANENT
# 
~~~

### iwlist扫描热点

~~~sh
# iwlist wlan0 scanning | grep ESSID
                    ESSID:"1234"
                    ESSID:"ZACH_24"
                    ESSID:"CMCC-i6cM"
~~~

### hostapd开热点

查看热点信息，通过配置文件查看

~~~
# ps | grep hostapd
  869 root     hostapd /tmp/hostapd.conf -B
  983 root     grep hostapd
# cat /tmp/hostapd.conf | grep ssid=
ssid=camera-xxx  
~~~

### tftp

对于BusyBox的TFTP客户端，命令格式略有不同。你可以尝试以下格式来下载文件：

```
tftp [服务器IP] -g -r [远程文件名] -l [本地文件名]
```

- `-g` 表示GET操作，即下载文件。`-p` 表示PUT操作，即上传文件。
- `-r [远程文件名]` 指定要从TFTP服务器下载的文件名。
- `-l [本地文件名]` 是可选的，用于指定本地文件名。如果不指定，文件将被保存为远程文件名。



    ### 启动 wlan0 网卡
​    ifconfig wlan0 up

    ### wpa_supplicant 连接无线网络
​    wpa_supplicant -i wlan0 -c /etc/wpa_supplicant.conf &
    ### udhcpc 自动获取 IP 地址
​    udhcpc -i wlan0

## 音频

~~~shell
# arecord -l
**** List of CAPTURE Hardware Devices ****
card 0: Device [USB PnP Sound Device], device 0: USB Audio [USB Audio]
  Subdevices: 1/1
  Subdevice #0: subdevice #0
# 
~~~

现在`arecord -l`命令显示了一个可用的录音设备，这意味着你的系统已经成功识别了连接的USB声卡。从输出信息来看，这个声卡被识别为`card 0`，设备名称是`USB PnP Sound Device`，并且具有一个可用的录音子设备。

### 开始录音

既然声卡已被识别，你可以开始使用它进行录音了。使用`arecord`命令，你可以指定声卡和子设备号来录音。基于你提供的信息，这里是一个简单的录音命令示例：

```bash
arecord -D plughw:0,0 -f cd test.wav
```

这里的命令参数含义如下：

- `-D plughw:0,0` 指定了使用的硬件设备。`plughw:0,0`表示使用第一个声卡（card 0）的第一个子设备（device 0）。这是`arecord`识别硬件设备的方式。
- `-f cd` 指定录音的格式为CD质量，即44100 Hz采样率，16位采样深度，双声道。
- `test.wav` 是录音保存的文件名。

运行这个命令后，`arecord`将开始使用USB声卡进行录音，录音数据将保存在当前目录下的`test.wav`文件中。要停止录音，可以使用Ctrl+C终止命令。

### 录音文件播放

录音完成后，你可以使用`aplay`命令或任何其他音频播放器软件来播放录音文件：

```bash
aplay test.wav
```

## 代码仓库

### tig

### git

* 打补丁

  ~~~
  git add kernel/scripts/mkimage
  git commit -m "添加或修改 mkimage 脚本"
  git format-patch -1 HEAD
  ~~~

* 重置到某个版本

  ~~~
  git log
  查看版本
  git reset 版本id
  ~~~

* 分支

  * 查看本地分支

  ~~~
  git branch
  查看分支名字
  ~~~

  * 切换本地分支

  ~~~
  git checkout 本地分支名字
  ~~~

  * 删除本地分支

  ~~~
  git branch -d 本地分支名字
  ~~~

  * 同步远端分支

  ~~~
  git checkout -b aaaaaaa origin/aaaaaaa
  ~~~

## 性能分析

### htop

* 查看进程的线程

  打开 `htop` 后，按 `F3`然后选择Display options再按空格选择Show custom thread names，最后按 `F10`退出

* 根据进程名字搜索

  `htop` 是 `top` 的一个增强版，提供了更为友好的用户界面。在 `htop` 中，你可以直接输入线程名来搜索。打开 `htop` 后，按 `F3` 或 `/` 输入搜索词（线程名）。

