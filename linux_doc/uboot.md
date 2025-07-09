## 启动流程

```
BOOTROM => SPL/TPL => U-BOOT => KERNEL
```

```c
start.s
// 汇编环境
=> IRQ/FIQ/lowlevel/vbar/errata/cp15/gic // ARM架构相关的lowlevel初始化
=> _main
    => stack // 准备好C环境需要的栈
    // 【第⼀阶段】C环境初始化，发起⼀系列的函数调⽤
    => board_init_f: init_sequence_f[]
        initf_malloc
        arch_cpu_init // 【SoC的lowlevel初始化】
        serial_init // 串口初始化
        dram_init // 【获取ddr容量信息】
        reserve_mmu // 从ddr末尾开始往低地址reserve内存
        reserve_video
        reserve_uboot
        reserve_malloc
        reserve_global_data
        reserve_fdt
        reserve_stacks
        dram_init_banksize
        sysmem_init
        setup_reloc // 确定U-Boot⾃⾝要reloc的地址
    // 汇编环境
    => relocate_code // 汇编实现U-Boot代码的relocation
    // 【第⼆阶段】C环境初始化，发起⼀系列的函数调⽤
    => board_init_r: init_sequence_r[]
        initr_caches // 使能MMU和I/Dcache
        initr_malloc
        bidram_initr
        sysmem_initr
        initr_of_live // 初始化of_live
        initr_dm // 初始化dm框架
        board_init // 【平台初始化，最核⼼部分】
            board_debug_uart_init // 串口iomux、clk配置
            init_kernel_dtb // 【切到kernel dtb】！
            clks_probe // 初始化系统频率
            regulators_enable_boot_on // 初始化系统电源
            io_domain_init // io-domain初始化
            set_armclk_rate // __weak，ARM提频(平台有需求才实现)
            dvfs_init // 宽温芯⽚的调频调压
            rk_board_init // __weak，由各个具体平台进⾏实现
        console_init_r
        board_late_init // 【平台late初始化】
            rockchip_set_ethaddr // 设置mac地址
            rockchip_set_serialno // 设置serialno
            setup_boot_mode // 解析"reboot xxx"命令、
                            // 识别按键和loader烧写模式、recovery
            charge_display // U-Boot充电
            rockchip_show_logo // 显⽰开机logo
            soc_clk_dump // 打印clk tree
            rk_board_late_init // __weak，由各个具体平台进⾏实现
        run_main_loop // 【进⼊命令⾏模式，或执⾏启动命令】
```

## 内核解压

- 64 位平台的机器通常烧写 Image，由U-Boot 加载到⽬标运⾏地址。

- 32 位平台的机器通常烧写zImage，由U-Boot加载到 kernel_addr_r 地址上，再由内核完成⾃解压到运行地址。

总结

- zImage格式通过自解压机制，通常会从一个地方加载并解压到另一个地方进行执行。
- Image格式则因为没有压缩和解压步骤，直接加载到其最终的运行地址并从该地址启动。

补充

- uImage是一种包含U-Boot特定头部的格式，任何内核映像（如zImage或Image）都可以通过mkimage工具转换为uImage。头部信息包括校验和、加载地址和入口点，以便让U-Boot能够识别和处理。

## 内核传参

### ✅ 一览表：booti vs bootm vs bootz
| 项目              | `bootm`                            | `bootz`                                  | `booti`                                  |
| --------------- | ---------------------------------- | ---------------------------------------- | ---------------------------------------- |
| 💡 含义           | boot **m** = uImage | boot **z** = zImage                      | boot **i** = Image（ARM64）                |
| 🎯 适用平台         | ARM 32 位                           | ARM 32 位                                 | ARM 64 位（AArch64）                        |
| 🧱 内核格式         | `uImage`（带 U-Boot 头）               | `zImage`（压缩内核）                           | `Image`（未压缩裸内核）                          |
| 🧩 是否需要 mkimage | ✅ 需要（uImage 由 `mkimage` 生成）        | ❌ 不需要                                    | ❌ 不需要                                    |
| 📦 参数传递方式       | ATAG（旧）或 FDT（可选）                   | FDT（必须）                                  | FDT（必须）                                  |
| 📁 是否需要 dtb     | 可选（ATAG 或 FDT）                     | ✅ 必须                                     | ✅ 必须                                     |
| 🧰 是否支持 ramdisk | ✅ 支持                               | ✅ 支持                                     | ✅ 支持                                     |
| 📆 是否推荐         | ❌（老旧，逐渐淘汰）                         | ✅（ARM32 推荐方式）                            | ✅✅（ARM64 必须）                             |
| 🛠️ 示例          | `bootm 0x80008000 - 0x82000000`    | `bootz 0x80008000 - 0x82000000` | `booti 0x80008000 - 0x82000000` |
| 🕶️命令行参数格式 | bootm/bootz/booti <kernel_addr> - <fdt_addr> |
