



## 基础

### cp

~~~sh
cp -rd
~~~

### grep

~~~
grep -nr "test"
grep -nr --include="*.h" "test"
~~~



### ls

~~~shell
ls -l --block-size=K
ls -la
ls -lh
~~~

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

## 压缩

7z

* 解压
  * 7z x 文件

tar

* 解压
  * tar -xzvf 文件.tar.gz
* 压缩
  * tar -czvf 文件.tar.gz 文件夹

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
​    ### udhcpc 自动获取 IP 地址
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

### Vimdiff

如果你倾向于使用命令行而不是图形界面，并且希望实现左右对比的效果，`vimdiff` 是一个非常好的选择。`vimdiff` 可以在命令行中提供一个强大的视觉对比体验，它会将差异以左右分栏的方式展示，这正是你所要求的。

### 配置 Git 使用 Vimdiff

首先，确保你已经安装了 Vim。然后，配置 Git 以使用 `vimdiff` 作为差异工具：

```bash
git config --global diff.tool vimdiff
git config --global difftool.prompt false
```

这样配置后，`git difftool` 命令将使用 `vimdiff` 来显示差异。

### 使用 Vimdiff 查看差异

当你想要查看两个提交之间的差异时，可以使用以下命令：

```bash
git difftool <commit1> <commit2>
```

如果你想要查看工作目录中当前更改与最近一次提交之间的差异，可以不加提交 ID：

```bash
git difftool
```

### difftool 操作基础

在 `vimdiff` 中，你可以使用 Vim 的标准导航键来移动光标。此外，这里有一些基础的 `vimdiff` 操作命令：

- `:qa`：退出所有对比窗口。
- `:qa!`：强制退出所有对比窗口，忽略任何更改。
- `:wqa`：保存所有更改并退出。
- `]c`：跳转到下一个差异。
- `[c`：跳转到上一个差异。
- `zo`：来展开被折叠的相同文本行
- `zc`：来折叠的相同文本行
- `do`：将当前差异从另一侧复制到当前侧（"diff obtain"）。
- `dp`：将当前差异从当前侧复制到另一侧（"diff put"）。
- `Ctrl-ww`：光标在两个窗口间彼此切换

使用 `vimdiff` 查看差异时，文件会被加载到 Vim 的不同窗口中，差异以高亮显示。这种方式提供了一个非常直观和强大的方式来查看和编辑文件的差异。

请注意，虽然 `vimdiff` 提供了一个强大的视觉对比工具，但它也需要一定的 Vim 使用经验。如果你不熟悉 Vim，可能需要先花时间学习 Vim 的基本操作。

diff

要查看文件是被修改还是新增（或删除），你可以使用 `git diff` 命令配合 `--name-status` 参数，这样不仅会列出文件名，还会在文件名前显示文件状态（如修改（M）、新增（A）、删除（D）等）。

```bash
git difftool --name-status 31d83862ce8b6e42bb4adedcb1880ae9d2be8dc2 5daf40ea89075f4e6a11112e5370ab66ff14b94f
```

使用这个命令，你将能够看到每个文件被修改的类型：

- **A**: 文件被添加（新增）
- **M**: 文件被修改
- **D**: 文件被删除

这样，你就可以直观地看到在两个提交之间，哪些文件是新增的，哪些是修改过的，哪些是被删除的。

~~~shell
wang@localhost:~//uboot$ git difftool 31d83862ce8b6e42bb4adedcb1880ae9d2be8dc2 5daf40ea89075f4e6a11112e5370ab66ff14b94f --name-status 
M       common/Makefile
A       common/Makefile.orig
M       common/autoboot.c
A       common/autoboot.c.orig
A       common/soft_crc32.c
A       common/soft_crc32.h
A       common/xiaomi_dfu.c
M       include/configs/infinity3.h
A       include/configs/infinity3.h.orig

~~~

要查看其中一个文件的具体修改内容，例如 `common/Makefile`，你可以使用 `git diff` 命令并指定文件路径：

```bash
git difftool 31d83862ce8b6e42bb4adedcb1880ae9d2be8dc2 5daf40ea89075f4e6a11112e5370ab66ff14b94f -- common/Makefile
```

这个命令会显示出 `31d83862ce8b6e42bb4adedcb1880ae9d2be8dc2` 和 `5daf40ea89075f4e6a11112e5370ab66ff14b94f` 这两个提交之间 `common/Makefile` 文件的具体差异。差异内容会以一种易于理解的方式展示出来，其中添加的行通常以绿色显示，删除的行则以红色显示（这取决于你的终端或 Git 客户端设置）。

~~~shell
git difftool 31d83862ce8b6e42bb4adedcb1880ae9d2be8dc2~1 -- common/Makefile
~~~

这个命令会显示出 `31d83862ce8b6e42bb4adedcb1880ae9d2be8dc2` 和 上一个提交之间 `common/Makefile` 文件的具体差异

如果你觉得每次输入两个对比 ID 很麻烦，有几种方法可以简化这个过程：

Git 支持使用 `HEAD` 来表示当前分支的最新提交。如果你想比较最新提交和之前的某个提交，可以使用相对引用，比如 `HEAD~1` 表示当前提交的前一个提交。例如：

```bash
git difftool HEAD~1 -- common/Makefile
```

这会比较当前最新提交和它的前一个提交之间的 `common/Makefile` 文件差异。

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
  git checkout -b aaaaaaa bbbbb/aaaaaaa
  ~~~

* cherry-pick

  如果你只想把最新的单笔提交合并到 `imi` 分支而不希望合并整个分支的更改，你可以使用 `git cherry-pick` 命令。这个命令允许你选择性地合并一个或几个特定的提交到你当前的分支。下面是如何操作的：

  ### 1. 确定要合并的提交哈希
  首先，你需要找到你想要合并的那个提交的哈希值。在你的原始分支上，你可以使用 `git log` 来查看提交历史并获取相应的提交哈希：
  ```bash
  git log
  ```
  从输出中复制你想要合并的提交的哈希值。

  ### 2. 切换到 `imi` 分支
  然后，切换到你的目标分支，即 `imi` 分支：
  ```bash
  git checkout imi
  ```

  ### 3. 使用 `cherry-pick` 合并特定的提交
  使用 `git cherry-pick` 命令来合并那个特定的提交：
  ```bash
  git cherry-pick <commit-hash>
  ```
  将 `<commit-hash>` 替换为你在步骤 1 中找到的提交哈希值。

  ### 注意事项
  - **冲突处理**：`cherry-pick` 操作可能会引起冲突，特别是当选定的提交与目标分支上的现有更改不兼容时。如果发生冲突，Git 将暂停操作，并要求你解决这些冲突。解决完冲突后，你需要使用 `git cherry-pick --continue` 来完成 `cherry-pick` 操作，或者使用 `git cherry-pick --abort` 来取消操作。
  - **提交历史**：使用 `cherry-pick` 会在目标分支创建一个新的提交，该提交是原提交的一个副本。这意味着它会有一个新的哈希值，但提交内容保持不变。

  通过以上步骤，你可以将指定的单笔提交应用到 `imi` 分支上，而不影响其他提交。这样做的好处是能精确控制合并到分支上的更改。

## 性能分析

### htop

* 查看进程的线程

  打开 `htop` 后，按 `F3`然后选择Display options再按空格选择Show custom thread names，最后按 `F10`退出

* 根据进程名字搜索

  `htop` 是 `top` 的一个增强版，提供了更为友好的用户界面。在 `htop` 中，你可以直接输入线程名来搜索。打开 `htop` 后，按 `F3` 或 `/` 输入搜索词（线程名）。

## 传输文件

`rz` 和 `sz` 是用于 ZMODEM 协议的接收和发送工具，它们可以在两个 Linux 设备之间通过串口进行文件传输。这种方法通常比 XMODEM 更可靠和快速。以下是如何使用 `rz` 和 `sz` 进行文件传输的步骤：

### 使用 ZMODEM 进行文件传输

#### 发送文件

1. **在接收方设备上**：

   准备接收文件，运行 `rz` 命令：

   ```bash
   rz < /dev/ttyGS0 > /dev/ttyGS0
   ```

   这会使接收方进入等待状态，准备接收文件。

2. **在发送方设备上**：

   使用 `sz` 命令发送文件：

   ```bash
   sz ota_t23.img < /dev/ttyACM0 > /dev/ttyACM0
   ```

   这将开始通过 ZMODEM 协议发送 `ota_t23.img` 文件。

### 验证传输成功

传输完成后，检查接收方设备中是否成功接收到 `ota_t23.img` 文件，并验证文件的完整性。例如，可以使用 `md5sum` 或 `sha256sum` 检查文件的一致性：

```bash
md5sum ota_t23.img
```

通过这些步骤，您应该能够使用 `rz` 和 `sz` 成功进行文件传输。如果问题仍然存在，请随时提供更多详细信息，以便进一步帮助您解决问题。



在 USB 串口通信中，波特率的设置通常并不直接影响数据传输的速度或通信的正确性。这是因为 USB 是一种基于包的通信协议，与传统的 UART 不同，它不依赖于波特率来确定传输速率。

### USB-Serial 转换器的波特率

当使用 USB-Serial 转换器（例如 USB 转 UART）时，通常会提供一个虚拟串口（如 `/dev/ttyUSB0`）。在这种情况下：

1. **波特率设置的影响**：
   - 波特率设置主要影响虚拟串口与实际串口设备之间的数据传输（如果存在实际串口设备）。
   - 对于大多数 USB-Serial 转换器，即使波特率设置不正确，USB 传输本身并不会出错。传输的实际速率由 USB 规范和驱动程序管理。
   - 您仍然需要在应用程序层设置波特率，因为许多应用程序会检查波特率设置以确保与传统串口设备的一致性。

2. **默认波特率**：
   - 通常，系统会为 USB-Serial 转换器分配一个默认的波特率（如 9600 或 115200），但这只是一个惯例，并不影响 USB 本身的传输能力。
   - 需要根据所连接的设备需求配置正确的波特率。

3. **与其他设备通信**：
   - 当连接到其他需要特定波特率的设备时（例如通过 USB-Serial 转换器连接到传统串口设备），波特率设置必须与这些设备的要求匹配。
   - 在应用程序中，设置波特率时，您可以使用命令行工具如 `stty` 或通过编程接口（例如在 C/C++ 中使用 `termios` 结构）进行设置。

### 结论

在 USB 通信中，波特率的设置更多是为了兼容传统串口接口和应用程序的需求，而不是对 USB 数据传输本身的性能有实际影响。如果不设置波特率，系统会使用默认设置，但这通常不会影响 USB 通信的正常运行。CRC 错误通常需要从硬件、驱动和数据完整性等角度进行排查。

## 走近科学

### shadowsocks

#### 服务端vps配置

~~~
sudo apt install shadowsocks-libev
~~~

vi /etc/shadowsocks-libev/config.json 

~~~
{
    "server":["::", "0.0.0.0"],
    "mode":"tcp_and_udp",
    "server_port":8388,
    "local_port":1080,
    "password":"your_password",
    "timeout":60,
    "method":"chacha20-ietf-poly1305"
}
~~~

~~~
systemctl restart shadowsocks-libev
~~~

#### 客户端下载

https://github.com/shadowsocks/shadowsocks-windows/releases

#### shadowsocks配置

~~~yaml
1、服务器地址: IP或者域名
2、服务器端口: 默认8388
3、加密方式: chacha20-ietf-poly1305
4、密码: 根据服务器配置文件里面的密码修改
~~~

配置好后可以生成二维码或者链接分享给朋友

#### clash配置

使用[Subscription Converter (yqhss.xyz)](http://sub.yqhss.xyz/)这个网站将shadowsocks分享的链接转化成clash

#### Shadowsocks-manager

使用docker最简单

### ShadowsocksR

#### 服务端vps配置

~~~sh
wget --no-check-certificate https://raw.githubusercontent.com/ShadowsocksR-Live/shadowsocksr-native/master/install/ssrn-install.sh
bash ssrn-install.sh
sudo systemctl restart ssr-native
~~~

参考他人配置

~~~sh
服务器地址：jm1.nodelist.top
服务器端口：20448
加密方式：aes-256-cfb
密码：yMd3UZ
协议：auth_aes128_sha1
协议参数：43259:fLqV25
混淆：tls1.2_ticket_auth
混淆参数：e10f443259.microsoft.com
~~~

~~~json
root@llx:~/temp# cat /etc/ssr-native/config.json 
{
    "password": "yMd3UZ",
    "method": "aes-256-cfb",
    "protocol": "auth_aes128_sha1",
    "protocol_param": "43259:fLqV25",
    "obfs": "tls1.2_ticket_auth",
    "obfs_param": "e10f443259.microsoft.com",

    "udp": false,
    "idle_timeout": 300,
    "connect_timeout": 6,
    "udp_timeout": 6,

    "server_settings": {
        "listen_address": "0.0.0.0",
        "listen_port": 20448
    },

    "client_settings": {
        "server": "120.119.110.520",
        "server_port": 20448,
        "listen_address": "0.0.0.0",
        "listen_port": 1080
    },

    "over_tls_settings": {
        "enable": false,
        "server_domain": "",
        "path": "//",
        "root_cert_file": ""
    }
}

~~~

### ssl证书

~~~sh
# Ubuntu 安装 certbot：
sudo apt install snapd
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
# 生成证书 & 修改 Nginx 配置
sudo certbot --nginx
# 根据指示进行操作
# 重启 Nginx
sudo service nginx restart
~~~



