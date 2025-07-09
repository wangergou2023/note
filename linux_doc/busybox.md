
```shell
内核启动
└── 解析启动参数 rdinit=/linuxrc
    └── 挂载根文件系统
        └── 执行 /linuxrc（软链接 → busybox）
            └── BusyBox 启动
                └── argv[0] == "linuxrc" → 执行 busybox 的 init
                    └── 读取 /etc/inittab
                        │
                        ├── 执行 ::sysinit:/bin/mount -a
                        │   └── 读取 /etc/fstab
                        │       └── 自动挂载配置的文件系统（如 /proc、/sys、/tmp、/dev/pts）
                        │
                        ├── 执行 ::sysinit:/etc/init.d/rcS
                        │   └── export 环境变量
                        │       └── 执行 /etc/init.d/S* 脚本（如 S10mdev、S20network）
                        │           └── 启动挂载、网络、服务等
                        │
                        └── 启动终端 console::askfirst:/bin/sh
                            └── 启动 /bin/sh
                                └── 提供串口登录界面
                                    └── 用户登录，系统进入运行态

```

cat /etc/inittab
```shell
::sysinit:/bin/mount -a
::sysinit:/etc/init.d/rcS
console::askfirst:-/bin/sh
```

cat /etc/fstab
```shell
proc      /proc       proc    defaults        0 0
sysfs     /sys        sysfs   defaults        0 0
tmpfs     /tmp        tmpfs   defaults        0 0
devpts    /dev/pts    devpts  gid=5,mode=620  0 0
```

cat /etc/init.d/rcS
```shell
#!/bin/sh
# Set Global Environment
export PATH=/system/bin:/bin:/sbin:/usr/bin:/usr/sbin
export LD_LIBRARY_PATH=/system/lib:/usr/lib

# Start all init scripts in /etc/init.d
# executing them in numerical order.
#
for i in /etc/init.d/S??* ;do

     # Ignore dangling symlinks (if any).
     [ ! -f "$i" ] && continue

     case "$i" in
	*.sh)
	    # Source shell script for speed.
	    (
		trap - INT QUIT TSTP
		set start
		. $i
	    )
	    ;;
	*)
	    # No sh extension, so fork subprocess.
	    $i start
	    ;;
    esac
done

```