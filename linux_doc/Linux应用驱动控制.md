## GPIO子系统

![](.\3.png)

以操作GPIO0_B0为例,配置为输入引脚，并将其短接GND和3.3V测试

### 引脚编号转换

Rockchip Pin的ID按照 **控制器(bank)+端口(port)+索引序号(pin)** 组成。

- 控制器和GPIO控制器数量⼀致
- 端口固定 A、B、C和D，每个端口仅有8个索引号,(A=0,B=1,C=2,D=3)
- 索引序号固定 0、1、2、3、4、5、6、7

GPIO0_B0表达的意思为**第0组控制器，端口号为B，索引号为0**。 该引脚号的计算公式为**32 x 0 + 8 x 1 + 0 = 8**

### sys文件系统

~~~sh
root@lubancat:~# echo 8 > /sys/class/gpio/export
root@lubancat:~# echo in > /sys/class/gpio/gpio8/direction
root@lubancat:~# cat /sys/class/gpio/gpio8/value 
1
root@lubancat:~# cat /sys/class/gpio/gpio8/value 
0
root@lubancat:~# echo 8 > /sys/class/gpio/unexport
~~~

### gpiod工具

~~~sh
root@lubancat:~# gpioget 0 8
1
root@lubancat:~# gpioget 0 8
0
~~~

补充：

| 命令       | 作用                     | 使用举例           | 说明                             |
| ---------- | ------------------------ | ------------------ | -------------------------------- |
| gpiodetect | 列出所有的GPIO控制器     | gpiodetect(无参数) | 列出所有的GPIO控制器             |
| gpioinfo   | 列出gpio控制器的引脚情况 | gpioinfo 0         | 列出第0组控制器引脚组情况        |
| gpioset    | 设置gpio                 | gpioset 0 8=0      | 设置第0组控制器编号8引脚为低电平 |
| gpioget    | 获取gpio引脚状态         | gpioget 0 8        | 获取第0组控制器编号8的引脚状态   |
| gpiomon    | 监控gpio的状态           | gpiomon 0 8        | 监控第0组控制器编号8的引脚状态   |

### libgpiod库（python版本）

~~~python
import gpiod
import time

# Define the GPIO chip (usually "gpiochip0" for the first controller, but it might be different on your platform)
CHIP_NAME = "gpiochip0"

# Define the pin number
PIN_NUMBER = 8

# Create a chip object
chip = gpiod.Chip(CHIP_NAME)

# Get a line object for the pin
line = chip.get_line(PIN_NUMBER)

# Set the line direction to input
line.request(consumer="gpio_reader", type=gpiod.LINE_REQ_DIR_IN)

try:
    while True:
        value = line.get_value()
        print(f"GPIO Pin {PIN_NUMBER} value: {value}")
        time.sleep(1)  # Wait for a second before the next read

except KeyboardInterrupt:
    print("Exiting...")

finally:
    line.release()

~~~

~~~sh
root@lubancat:~# python3 gpio.py 
GPIO Pin 8 value: 1
GPIO Pin 8 value: 1
GPIO Pin 8 value: 0
GPIO Pin 8 value: 0
GPIO Pin 8 value: 1
GPIO Pin 8 value: 1
~~~

### libgpiod库（c版本）省略



## INPUT子系统

### evtest工具

~~~sh
root@lubancat:~# evtest 
No device specified, trying to scan all of /dev/input/event*
Available devices:
/dev/input/event0:      fdd70030.pwm
/dev/input/event1:      rk805 pwrkey
/dev/input/event2:      adc-keys
/dev/input/event3:      rk-headset
/dev/input/event4:      USB OPTICAL MOUSE 
Select the device event number [0-4]: 4
Input driver version is 1.0.1
Input device ID: bus 0x3 vendor 0x30fa product 0x300 version 0x111
Input device name: "USB OPTICAL MOUSE "
Supported events:
  Event type 0 (EV_SYN)
  Event type 1 (EV_KEY)
    Event code 272 (BTN_LEFT)
    Event code 273 (BTN_RIGHT)
    Event code 274 (BTN_MIDDLE)
    Event code 275 (BTN_SIDE)
    Event code 276 (BTN_EXTRA)
  Event type 2 (EV_REL)
    Event code 0 (REL_X)
    Event code 1 (REL_Y)
    Event code 8 (REL_WHEEL)
  Event type 4 (EV_MSC)
    Event code 4 (MSC_SCAN)
Properties:
Testing ... (interrupt to exit)
Event: time 1679939811.444900, type 2 (EV_REL), code 0 (REL_X), value 1
Event: time 1679939811.444900, type 2 (EV_REL), code 1 (REL_Y), value -1
Event: time 1679939811.444900, -------------- SYN_REPORT ------------
Event: time 1679939811.452787, type 2 (EV_REL), code 0 (REL_X), value 8
Event: time 1679939811.452787, type 2 (EV_REL), code 1 (REL_Y), value 2
Event: time 1679939811.452787, -------------- SYN_REPORT ------------
~~~

### 命令行

~~~
root@lubancat:~# ls /dev/input/by-path/ -l
total 0
lrwxrwxrwx 1 root root 9 Mar 28 01:54 platform-adc-keys-event -> ../event2
lrwxrwxrwx 1 root root 9 Mar 28 01:55 platform-fd840000.usb-usb-0:1:1.0-event-mouse -> ../event4
lrwxrwxrwx 1 root root 9 Mar 28 01:54 platform-fdd40000.i2c-platform-rk805-pwrkey-event -> ../event1
lrwxrwxrwx 1 root root 9 Mar 28 01:54 platform-fdd70030.pwm-event -> ../event0
lrwxrwxrwx 1 root root 9 Mar 28 01:54 platform-rk-headset-event -> ../event3
root@lubancat:~# 
root@lubancat:~# ls /dev/input/by-id/ -l
total 0
lrwxrwxrwx 1 root root 9 Mar 28 01:55 usb-30fa_USB_OPTICAL_MOUSE-event-mouse -> ../event4
root@lubancat:~# 
~~~



"H: Handlers"这一行，它列出了与此设备相关的事件处理程序

~~~sh
root@lubancat:~# cat /proc/bus/input/devices
I: Bus=0019 Vendor=524b Product=0006 Version=0100
N: Name="fdd70030.pwm"
P: Phys=gpio-keys/remotectl
S: Sysfs=/devices/platform/fdd70030.pwm/input/input0
U: Uniq=
H: Handlers=kbd event0 cpufreq dmcfreq 
B: PROP=0
B: EV=3
B: KEY=70010 20000000000000 0 100010002000000 78000004000c800 1e16c000000000 4ffc

I: Bus=0019 Vendor=0000 Product=0000 Version=0000
N: Name="rk805 pwrkey"
P: Phys=rk805_pwrkey/input0
S: Sysfs=/devices/platform/fdd40000.i2c/i2c-0/0-0020/rk805-pwrkey/input/input1
U: Uniq=
H: Handlers=kbd event1 cpufreq dmcfreq 
B: PROP=0
B: EV=3
B: KEY=10000000000000 0

I: Bus=0019 Vendor=0001 Product=0001 Version=0100
N: Name="adc-keys"
P: Phys=adc-keys/input0
S: Sysfs=/devices/platform/adc-keys/input/input2
U: Uniq=
H: Handlers=event2 cpufreq dmcfreq 
B: PROP=0
B: EV=3
B: KEY=1000000 0 0 0 0 0 0

I: Bus=0000 Vendor=0001 Product=0001 Version=0100
N: Name="rk-headset"
P: Phys=
S: Sysfs=/devices/platform/rk-headset/input/input3
U: Uniq=
H: Handlers=event3 
B: PROP=0
B: EV=3
B: KEY=400000000 0 c000000000000 0

I: Bus=0003 Vendor=30fa Product=0300 Version=0111
N: Name="USB OPTICAL MOUSE "
P: Phys=usb-fd840000.usb-1/input0
S: Sysfs=/devices/platform/fd840000.usb/usb3/3-1/3-1:1.0/0003:30FA:0300.0001/input/input4
U: Uniq=
H: Handlers=event4 cpufreq dmcfreq 
B: PROP=0
B: EV=17
B: KEY=1f0000 0 0 0 0
B: REL=103
B: MSC=10

root@lubancat:~# 
~~~



## 串口通讯

### stty工具

~~~sh
root@lubancat:~# stty -F /dev/ttyS3
[  107.367537] of_dma_request_slave_channel: dma-names property of node '/serial@fe670000' missing or empty
speed 9600 baud; line = 0;
-brkint -imaxbel
~~~

### microcom工具

~~~sh
microcom -s [baudrate] /dev/ttySx
~~~

### cfsetospeed和cfsetispeed函数

~~~c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <termios.h>
#include <string.h>
#include <sys/ioctl.h>

//第一部分代码/
//根据具体的设备修改
const char default_path[] = "/dev/ttyS3";

int main(int argc, char *argv[])
{
    int fd;
    int res;
    char *path;
    char buf[1024] = "Embedfire tty send test.\n";

    //第二部分代码/
    //若无输入参数则使用默认终端设备
    if (argc > 1)
        path = argv[1];
    else
        path = (char *)default_path;

    //获取串口设备描述符
    printf("This is tty/usart demo.\n");
    fd = open(path, O_RDWR);
    if (fd < 0) {
        printf("Fail to Open %s device\n", path);
        return 0;
    }


    //第三部分代码/
    struct termios opt;
    //清空串口接收缓冲区
    tcflush(fd, TCIOFLUSH);
    // 获取串口参数opt
    tcgetattr(fd, &opt);
    //设置串口输出波特率
    cfsetspeed(&opt, B9600);
    //设置数据位数
    opt.c_cflag &= ~CSIZE;
    opt.c_cflag |= CS8;
    //校验位
    opt.c_cflag &= ~PARENB;
    opt.c_iflag &= ~INPCK;
    //设置停止位
    opt.c_cflag &= ~CSTOPB;
    //更新配置
    tcsetattr(fd, TCSANOW, &opt);
    printf("Device %s is set to 9600bps,8N1\n",path);

    //第四部分代码/
    do {
        //发送字符串
        write(fd, buf, strlen(buf));
        //接收字符串
        res = read(fd, buf, 1024);
        if (res >0 )
        //给接收到的字符串加结束符
        buf[res] = '\0';
        printf("Receive res = %d bytes data: %s\n",res, buf);
    } while (res >= 0);

    printf("read error,res = %d",res);
    close(fd);
    return 0;
}
~~~



## IIC通讯

### 命令行

~~~sh
root@lubancat:~# ls /sys/class/i2c-dev/
i2c-0  i2c-6
root@lubancat:~# ls /dev/i2c-*
i2c-0  i2c-6
~~~



### i2c-tools

* i2cdetect – 用来列举 I2C bus 和上面所有的设备
* i2cdump – 显示 i2c 设备所有 register 的值
* i2cget – 读取 i2c 设备某个 register 的值
* i2cset – 写入 i2c 设备某个 register 的值

查看I2c Bus0 总线的设备

~~~
~ # i2cdetect -y 0
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- 18 -- -- -- -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- 32 -- -- -- -- -- -- -- -- -- -- -- -- --
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50: -- 51 -- -- -- -- -- -- -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
~~~

查看I2c Bus0 总线的地址0x32的设备

~~~
~ # i2cdump -y 0 0x32
No size specified (using byte-data access)
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f    0123456789abcdef
00: XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX    XXXXXXXXXXXXXXXX
10: 40 33 13 06 31 05 23 d8 00 00 00 00 00 04 20 00    @3??1?#?.....? .
20: 10 11 12 13 14 15 16 17 18 19 1a 1b 1c 1d 1e 1f    ????????????????
30: 20 01 22 00 00 00 00 00 00 00 00 00 00 00 00 00     ?".............
40: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
50: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
60: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
70: XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX    XXXXXXXXXXXXXXXX
80: XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX    XXXXXXXXXXXXXXXX
90: 40 33 13 06 31 05 23 d8 00 00 00 00 00 04 20 00    @3??1?#?.....? .
a0: 10 11 12 13 14 15 16 17 18 19 1a 1b 1c 1d 1e 1f    ????????????????
b0: 20 01 22 00 00 00 00 00 00 00 00 00 00 00 00 00     ?".............
c0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
d0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
e0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00    ................
f0: XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX XX    XXXXXXXXXXXXXXXX
~ #

~~~

### ioctl函数

写应用程序时需要使用ioctl函数设置i2c相关配置，其函数原型如下

~~~c
 #include <sys/ioctl.h>

 int ioctl(int fd, unsigned long request, ...);
~~~

其中对于终端request的值常用的有以下几种

| I2C_RETRIES     | 设置收不到ACK时的重试次数，默认为1 |
| --------------- | ---------------------------------- |
| I2C_TIMEOUT     | 设置超时时限的jiffies              |
| I2C_SLAVE       | 设置从机地址                       |
| I2C_SLAVE_FORCE | 强制设置从机地址                   |
| I2C_TENBIT      | 选择地址长度0为7位地址，非0为10位  |

~~~c
#include <stdio.h>
#include <linux/i2c-dev.h>
#include <sys/ioctl.h>
#include <fcntl.h>

int main() {
    int file;
    char *filename = "/dev/i2c-0";
    int addr = 0x40; // I2C设备地址

    // 打开I2C设备文件
    if ((file = open(filename, O_RDWR)) < 0) {
        perror("Failed to open the i2c bus");
        return 1;
    }

    // 指定我们要通过I2C通信的设备地址
    if (ioctl(file, I2C_SLAVE, addr) < 0) {
        perror("Failed to acquire bus access and/or talk to slave");
        return 1;
    }

    // 此时可以使用read()/write()来与I2C设备进行通信

    // 关闭文件
    close(file);
    return 0;
}

static int i2c_write(int fd, uint8_t addr,uint8_t reg,uint8_t val)
{
    int retries;
    uint8_t data[2];

    data[0] = reg;
    data[1] = val;

    //设置地址长度：0为7位地址
    ioctl(fd,I2C_TENBIT,0);

    //设置从机地址
    if (ioctl(fd,I2C_SLAVE,addr) < 0){
        printf("fail to set i2c device slave address!\n");
        close(fd);
        return -1;
    }

    //设置收不到ACK时的重试次数
    ioctl(fd,I2C_RETRIES,5);

    if (write(fd, data, 2) == 2){
        return 0;
    }
    else{
        return -1;
    }

}

static int i2c_read(int fd, uint8_t addr,uint8_t reg,uint8_t * val)
{
    int retries;

    //设置地址长度：0为7位地址
    ioctl(fd,I2C_TENBIT,0);

    //设置从机地址
    if (ioctl(fd,I2C_SLAVE,addr) < 0){
        printf("fail to set i2c device slave address!\n");
        close(fd);
        return -1;
    }

    //设置收不到ACK时的重试次数
    ioctl(fd,I2C_RETRIES,5);

    if (write(fd, &reg, 1) == 1){
        if (read(fd, val, 1) == 1){
                return 0;
        }
    }
    else{
        return -1;
    }
}

~~~



## SPI通信

### 命令行

~~~
root@lubancat:~# ls /dev/spi*
/dev/spidev3.0
root@lubancat:~# ls /sys/class/spidev/
spidev3.0
~~~

### ioctl函数

编写应用程序时还需要使用ioctl函数设置spi相关配置，其函数原型如下

~~~
 #include <sys/ioctl.h>

 int ioctl(int fd, unsigned long request, ...);
~~~

其中对于终端request的值常用的有以下几种

| SPI_IOC_RD_MODE32        | 设置读取SPI模式(对应上文的SPI的四种模式的表格,SPI_MODE_x) |
| ------------------------ | --------------------------------------------------------- |
| SPI_IOC_WR_MODE32        | 设置写入SPI模式(对应上文的SPI的四种模式的表格,SPI_MODE_x) |
| SPI_IOC_RD_LSB_FIRST     | 设置SPI读取数据模式(LSB先行返回1)                         |
| SPI_IOC_WR_LSB_FIRST     | 设置SPI写入数据模式。(0:MSB，非0：LSB)                    |
| SPI_IOC_RD_BITS_PER_WORD | 设置SPI读取设备的字长                                     |
| SPI_IOC_WR_BITS_PER_WORD | 设置SPI写入设备的字长                                     |
| SPI_IOC_RD_MAX_SPEED_HZ  | 设置读取SPI设备的最大通信频率。                           |
| SPI_IOC_WR_MAX_SPEED_HZ  | 设置写入SPI设备的最大通信速率                             |
| SPI_IOC_MESSAGE(N)       | 一次进行双向/多次读写操作                                 |

~~~c
#include <sys/ioctl.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <linux/spi/spidev.h>

#define SPI_DEV_PATH "/dev/spidev3.0"

/*SPI 接收 、发送 缓冲区*/
unsigned char tx_buffer[100] = "hello the world !";
unsigned char rx_buffer[100];


int fd;                  					// SPI 控制引脚的设备文件描述符
static unsigned  mode = SPI_MODE_2;         //用于保存 SPI 工作模式
static uint8_t bits = 8;        			// 接收、发送数据位数
static uint32_t speed = 10000000; 			// 发送速度
static uint16_t delay;          			//保存延时时间

void transfer(int fd, uint8_t const *tx, uint8_t const *rx, size_t len)
{
    int ret;

    struct spi_ioc_transfer tr = {
        .tx_buf = (unsigned long)tx,
        .rx_buf = (unsigned long)rx,
        .len = len,
        .delay_usecs = delay,
        .speed_hz = speed,
        .bits_per_word = bits,
        .tx_nbits = 1,
        .rx_nbits = 1
    };

    ret = ioctl(fd, SPI_IOC_MESSAGE(1), &tr);
    
    if (ret < 1)
        printf("can't send spi message\n");
}

void spi_init(void)
{
    int ret = 0;
    //打开 SPI 设备
    fd = open(SPI_DEV_PATH, O_RDWR);
    if (fd < 0)
        printf("can't open %s\n",SPI_DEV_PATH);

    //spi mode 设置SPI 工作模式
    ret = ioctl(fd, SPI_IOC_WR_MODE32, &mode);
    if (ret == -1)
        printf("can't set spi mode\n");

    //bits per word  设置一个字节的位数
    ret = ioctl(fd, SPI_IOC_WR_BITS_PER_WORD, &bits);
    if (ret == -1)
        printf("can't set bits per word\n");

    //max speed hz  设置SPI 最高工作频率
    ret = ioctl(fd, SPI_IOC_WR_MAX_SPEED_HZ, &speed);
    if (ret == -1)
        printf("can't set max speed hz\n");

    //打印
    printf("spi mode: 0x%x\n", mode);
    printf("bits per word: %d\n", bits);
    printf("max speed: %d Hz (%d KHz)\n", speed, speed / 1000);
}

int main(int argc, char *argv[])
{
    /*初始化SPI */
    spi_init();

    /*执行发送*/
    transfer(fd, tx_buffer, rx_buffer, sizeof(tx_buffer));

    /*打印 tx_buffer 和 rx_buffer*/
    printf("tx_buffer: \n %s\n ", tx_buffer);
    printf("rx_buffer: \n %s\n ", rx_buffer);
    
    close(fd);
    return 0;
}
~~~

## PWM控制

~~~
root@lubancat:~# ls /sys/class/pwm/
pwmchip0  pwmchip1
~~~

### sys文件系统

~~~
#将pwm3导出到用户空间
echo 0 > /sys/class/pwm/pwmchip1/export

#设置pwm周期 单位为ns
echo 1000000 > /sys/class/pwm/pwmchip1/pwm0/period

#设置占空比
echo 500000 > /sys/class/pwm/pwmchip1/pwm0/duty_cycle

#设置pwm极性
echo "normal" > /sys/class/pwm/pwmchip1/pwm0/polarity

#使能pwm
echo 1 > /sys/class/pwm/pwmchip1/pwm0/enable

#取消将pwm3导出到用户空间
echo 0 > /sys/class/pwm/pwmchip1/unexport
~~~

## 音频

### ALSA框架

## 相机

### V4L2框架

### OpenCV视觉算法库

## 显示

### DRM框架

### Wayland协议
