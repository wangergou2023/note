



## 基础

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

### ip

* 查询设备

~~~sh
# ip neigh show dev wlan1
192.168.16.10 lladdr b4:c2:e0:b1:55:2d used 93269/0/93269 probes 0 PERMANENT
# 
~~~

### iwlist

~~~sh
# iwlist wlan0 scanning | grep ESSID
                    ESSID:"1234"
                    ESSID:"ZACH_24"
                    ESSID:"CMCC-i6cM"
~~~

### hostapd

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

