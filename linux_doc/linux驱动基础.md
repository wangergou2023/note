



## 同步与互斥

### **Spinlock (自旋锁)**

### **Semaphore (信号量)**

### **Mutex (互斥锁)**

## 内存申请

在Linux内核中，`devm_kcalloc` 和 `devm_kzalloc` 都是设备管理的内存分配函数，它们的特点是分配的内存会与指定的设备关联，当设备被注销或驱动被卸载时，这些内存会被自动释放。这大大简化了内存管理，并减少了内存泄漏的风险。

以下是这两个函数的详细说明：

1. **`devm_kzalloc`**:
   
   - **目的**：为设备分配并初始化内存。
   - **原型**：
     
     ```c
     void *devm_kzalloc(struct device *dev, size_t size, gfp_t gfp);
     ```
   - **参数**：
     - `dev`: 要分配内存的设备。
     - `size`: 请求的内存大小。
     - `gfp`: 分配标志，例如 `GFP_KERNEL`。
   - **返回**：成功时返回指向分配内存的指针，失败时返回 `NULL`。
   - **注意**：分配的内存区域将被初始化为零。
   
2. **`devm_kcalloc`**:
   
   - **目的**：为设备分配并初始化数组内存。
   - **原型**：
     ```c
     void *devm_kcalloc(struct device *dev, size_t n, size_t size, gfp_t gfp);
     ```
   - **参数**：
     - `dev`: 要分配内存的设备。
     - `n`: 数组中的元素数量。
     - `size`: 每个元素的大小。
     - `gfp`: 分配标志，例如 `GFP_KERNEL`。
   - **返回**：成功时返回指向分配内存的指针，失败时返回 `NULL`。
   - **注意**：分配的内存区域将被初始化为零。

这两个函数的主要区别在于它们的用途：`devm_kzalloc` 是用于为设备分配一块特定大小的内存，而 `devm_kcalloc` 是用于为设备分配一个数组。

使用这些函数的好处是它们提供了一个简单、一致的方法来为设备分配内存，而不必担心在设备注销或驱动卸载时释放内存。这样可以简化代码并降低出错的风险。

## 工作队列

~~~c
//初始化一个延迟工作队列项
#define INIT_DELAYED_WORK(_work, _func)					\
	__INIT_DELAYED_WORK(_work, _func, 0)
~~~

## 定时器

~~~c
//初始化一个高分辨率定时器
void hrtimer_init(struct hrtimer *timer, clockid_t which_clock,
			 enum hrtimer_mode mode);
~~~



## 基础知识

### 一切皆文件

include\linux\fs.h

#### struct file

~~~c
struct file {
    struct path		f_path;          // 文件的路径结构
    struct inode	*f_inode;        // 与此文件关联的inode对象
    const struct file_operations	*f_op;   // 文件操作函数指针的集合

    spinlock_t		f_lock;          // 自旋锁，用于保护某些字段
    atomic_long_t		f_count;       // 文件引用计数
    unsigned int 		f_flags;       // 文件标志
    fmode_t			f_mode;        // 文件模式（例如，读、写等）
    struct mutex		f_pos_lock;    // 用于保护 f_pos 的互斥锁
    loff_t			f_pos;         // 文件的当前位置
    struct fown_struct	f_owner;      // 文件的所有者信息
    const struct cred	*f_cred;      // 文件的凭证信息
    struct file_ra_state	f_ra;       // 用于文件预读的状态

    struct address_space	*f_mapping; // 文件数据的内存映射

    void			*private_data; // 指向私有数据的指针，可能用于各种驱动程序
} __attribute__((aligned(4)));	/* 确保至少4字节对齐 */
~~~

#### struct inode

```c
struct inode {
	umode_t i_mode; // 文件模式（例如，读、写、执行等权限）
	kuid_t i_uid; // 文件的用户ID
	kgid_t i_gid; // 文件的组ID
	unsigned int i_flags; // inode标志
	const struct inode_operations *i_op; // inode操作函数指针的集合
	struct super_block *i_sb; // inode所属的超级块

	unsigned long i_ino; // inode号码
	dev_t i_rdev; // 特殊文件的设备号
	union {
		struct block_device *i_bdev;
		struct cdev *i_cdev;
		char *i_link;
	};
	loff_t i_size; // 文件大小
	spinlock_t i_lock; // 自旋锁，用于保护某些字段

	unsigned long i_state; // inode状态
	struct rw_semaphore i_rwsem; // 读写信号量

	atomic64_t i_version; // inode版本
	atomic_t i_count; // inode引用计数

	const struct file_operations *i_fop; // 默认的文件操作

	void *i_private; // 文件系统或设备的私有指针，例如与I2C, SPI, USB, USART, RMII等设备驱动程序相关的数据
} __randomize_layout;
```

#### struct file_operations

```c
struct file_operations {
	struct module *owner; // 所属模块，通常用于模块计数
	loff_t (*llseek)(struct file *, loff_t, int); // 改变文件的当前读/写位置
	ssize_t (*read)(struct file *, char __user *, size_t, loff_t *); // 从文件读取数据
	ssize_t (*write)(struct file *, const char __user *, size_t, loff_t *); // 向文件写入数据
	int (*open)(struct inode *, struct file *); // 打开文件
	int (*release)(struct inode *, struct file *); // 关闭文件
	int (*mmap)(struct file *, struct vm_area_struct *); // 创建内存映射
	long (*unlocked_ioctl)(struct file *, unsigned int, unsigned long); // 无锁IO控制
	int (*flush)(struct file *, fl_owner_t id); // 刷新文件
	int (*fsync)(struct file *, loff_t, loff_t, int datasync); // 同步文件
	__poll_t (*poll)(struct file *, struct poll_table_struct *); // 轮询文件状态
} __randomize_layout;
```

`struct file`、`struct inode` 和 `struct file_operations` 在 Linux 内核中是文件系统和设备驱动的核心结构。它们的关系如下：

1. **`struct inode`**: 
    - 这是文件系统中的一个核心结构，代表一个 inode 对象。在 UNIX 世界中，几乎所有内容都是文件（文件、目录、设备等）。每个文件在文件系统中都有一个与之关联的 inode，其中包含了该文件的元数据（例如，文件的大小、权限、创建时间、修改时间等）。
    - `struct inode` 还包含指向与特定文件系统相关的操作的指针，这些操作由文件系统实现。例如，EXT4 和 NFS 文件系统会为其 inodes 提供不同的操作集。

2. **`struct file`**: 
    - 当一个进程打开一个文件时，它会获得一个与该文件关联的 `struct file` 对象。这个结构描述了一个打开的文件实例。
    - `struct file` 中有一个 `f_op` 字段，该字段是一个指向 `struct file_operations` 的指针，其中包含了一系列与该文件相关的操作。
    - 同样，`struct file` 也有一个 `f_inode` 字段，这是一个指向 `struct inode` 的指针，表示该文件的 inode。

3. **`struct file_operations`**: 
    - 这个结构体定义了一组可以在一个打开的文件上执行的操作的函数指针。
    - 这是驱动程序和文件系统实现与内核接口的主要方式。当你在一个文件上执行一个操作（如读或写）时，内核会调用与该文件关联的 `struct file_operations` 中的相应函数。
    - 例如，当用户空间的程序调用 `read()` 系统调用时，内核会转而调用与打开的文件关联的 `struct file_operations` 中的 `read` 函数。

**关系概述**：

- 每个文件都有一个 inode（`struct inode`），它包含文件的元数据。
- 当一个文件被打开时，会创建一个文件描述符（`struct file`）来表示这个打开的文件实例。
- `struct file` 包含一个指向 `struct inode` 的指针（即文件的元数据）以及一个指向 `struct file_operations` 的指针（即可以在该文件上执行的操作）。

在实际的文件系统或设备驱动开发中，这三个结构体的实例通常会密切地协同工作，以提供期望的功能和性能。

### 设备树

include\linux\of.h

~~~c
//为 Linux 内核提供了一个与设备树或 ACPI 描述符交互的抽象机制，而不需要知道具体是哪种描述符
struct fwnode_handle {
	struct fwnode_handle *secondary;
	const struct fwnode_operations *ops;
	struct device *dev;
	struct list_head suppliers;
	struct list_head consumers;
	u8 flags;
};
~~~

~~~c
struct device_node {
	const char *name;
	const char *type;
	phandle phandle;
	const char *full_name;
	struct fwnode_handle fwnode;

	struct	property *properties;
	struct	property *deadprops;	/* removed properties */
	struct	device_node *parent;
	struct	device_node *child;
	struct	device_node *sibling;
#if defined(CONFIG_OF_KOBJ)
	struct	kobject kobj;
#endif
	unsigned long _flags;
	void	*data;
#if defined(CONFIG_SPARC)
	const char *path_component_name;
	unsigned int unique_id;
	struct of_irq_controller *irq_trans;
#endif
};
~~~

- **name：** 节点中属性为name的值
- **type：** 节点中属性为device_type的值
- **full_name：** 节点的名字，在device_node结构体后面放一个字符串，full_name指向它
- **properties：** 链表，连接该节点的所有属性
- **parent：** 指向父节点
- **child：** 指向子节点
- **sibling：** 指向兄弟节点



~~~c
struct of_device_id {
	char	name[32];
	char	type[32];
	char	compatible[128];
	const void *data;
};	
~~~

- **name：** 节点中属性为name的值
- **type：** 节点中属性为device_type的值
- **compatible：** 节点的名字，在device_node结构体后面放一个字符串，full_name指向它
- **data：** 链表，连接该节点的所有属性



~~~c
const struct of_device_id *of_match_node(const struct of_device_id *matches, const struct device_node *node);
~~~

这个函数接受两个参数：match（匹配函数）和np（设备树节点指针）。首先，它会调用match函数，并将np作为参数传递给它，以便进行节点匹配。如果match函数返回真，则表示节点匹配成功，of_match_node函数将返回匹配项的指针。



Linux内核版本4.19的设备树子系统提供了一系列的API来与设备树进行交互。以下是一些常用的设备树API：

1. **节点查找**:
    - `of_find_node_by_path()`: 根据设备树路径查找节点。
    - `of_find_node_by_name()`: 根据节点名称查找节点。
    - `of_find_node_by_phandle()`: 根据phandle查找节点。
    - `of_find_compatible_node()`: 根据兼容性字符串查找节点。
    - `of_find_matching_node()`: 根据匹配列表查找节点。
    - `of_find_node_with_property()`: 查找具有特定属性的节点。

2. **属性查找与读取**:
    - `of_get_property()`: 获取设备节点的属性值。
    - `of_find_property()`: 查找设备节点的属性。
    - `of_property_read_*()`: 一系列函数（如`of_property_read_u32`、`of_property_read_string`等）来读取特定类型的属性值。

3. **设备状态与兼容性检查**:
    - `of_device_is_available()`: 检查设备节点是否可用。
    - `of_device_is_compatible()`: 检查设备节点是否与给定的兼容字符串匹配。

4. **子节点与父节点迭代**:
    - `of_get_child_by_name()`: 获取具有指定名称的子节点。
    - `of_get_next_child()`: 获取下一个子节点。
    - `of_get_next_available_child()`: 获取下一个可用的子节点。
    - `of_get_parent()`: 获取父节点。

5. **地址和大小转换**:
    - `of_translate_address()`: 将设备树地址转换为物理地址。
    - `of_address_to_resource()`: 将设备树地址转换为资源结构。

6. **中断处理**:
    - `of_irq_get()`: 从设备树节点获取中断号。
    - `of_irq_count()`: 获取设备树节点的中断数量。

7. **内存管理**:
    - `of_reserved_mem_device_init()`: 为设备初始化预留的内存。

8. **设备树节点的参考计数**:
    - `of_node_get()`: 增加节点的引用计数。
    - `of_node_put()`: 减少节点的引用计数。
    
9. 匹配表

    * of_match_node():

    * ```
      
      ```

这只是内核中设备树API的一个子集。有关更详细的信息和其他API，可以参考Linux内核的源代码，特别是`include/linux/of.h`和`drivers/of/`目录。

kernel下查看设备树/proc/device-tree

~~~shell
root@lubancat:~# ls /proc/device-tree/pinctrl/gpio\@fdd60000/
#gpio-cells           gpio-controller       name
#interrupt-cells      gpio-ranges           phandle
clocks                interrupt-controller  reg
compatible            interrupts 
~~~

uboot下查看设备树
~~~
=> fdt print /pinctrl/gpio@fdd60000
gpio@fdd60000 {
        compatible = "rockchip,gpio-bank";
        reg = <0x00000000 0xfdd60000 0x00000000 0x00000100>;
        interrupts = <0x00000000 0x00000021 0x00000004>;
        clocks = <0x00000032 0x0000002e 0x00000032 0x0000000c>;
        gpio-controller;
        #gpio-cells = <0x00000002>;
        gpio-ranges = <0x00000119 0x00000000 0x00000000 0x00000020>;
        interrupt-controller;
        #interrupt-cells = <0x00000002>;
        phandle = <0x00000037>;
};
~~~



### device设备

include\linux\device.h

~~~c
struct device {
	struct device_node *of_node; 		/* 与设备相关联的设备树节点 */
	struct fwnode_handle *fwnode;		/* 设备树子节点 */
	void *platform_data;				/* 平台特定数据 */
	void *driver_data;					/* 驱动私有数据 */
};
~~~

~~~c
//获取设备的私有数据
static inline void *dev_get_platdata(const struct device *dev)
{
	return dev->platform_data;
}

//设备有多少个子节点
unsigned int device_get_child_node_count(const struct device *dev);

//获取子节点
struct fwnode_handle *device_get_next_child_node(const struct device *dev,
						 struct fwnode_handle *child);

//这个宏为给定设备的每个子节点执行循环。在每次循环迭代中，child 都会被设置为当前的子节点。这使得你可以在循环体中处理每个子节点，而不必手动遍历它们。
#define device_for_each_child_node(dev, child)				\
	for (child = device_get_next_child_node(dev, NULL); child;	\
	     child = device_get_next_child_node(dev, child))

//检查设备是否具有给定的属性名称，并返回一个相应的布尔值。
static inline bool device_property_read_bool(const struct device *dev,
					     const char *propname)
{
	return device_property_present(dev, propname);
}

//从设备的属性中读取一个字符串值
int device_property_read_string(const struct device *dev, const char *propname,
				const char **val)


//检查固件节点是否具有给定的属性名称，并返回一个相应的布尔值
static inline bool fwnode_property_read_bool(const struct fwnode_handle *fwnode,
					     const char *propname)
{
	return fwnode_property_present(fwnode, propname);
}

//从给定的固件节点属性中读取一个 u32 (32位无符号整数) 
static inline int fwnode_property_read_u32(const struct fwnode_handle *fwnode,
					   const char *propname, u32 *val)
{
	return fwnode_property_read_u32_array(fwnode, propname, val, 1);
}

//为给定的设备添加一个自定义的清理动作。这个动作会在设备卸载时被自动执行
#define devm_add_action(release, action, data) \
	__devm_add_action(release, action, data, #action)

//这个函数的目的是请求一个中断，该中断可以在任何上下文中（任务上下文或中断上下文）被处理。它是设备管理 (devm) 版本的函数，这意味着请求的中断与设备的生命周期关联：当设备被注销或驱动被卸载时，这个中断会被自动释放。
extern int __must_check
devm_request_any_context_irq(struct device *dev, unsigned int irq,
		 irq_handler_t handler, unsigned long irqflags,
		 const char *devname, void *dev_id);
~~~



### driver驱动

~~~c
struct device_driver {
	const char		*name;
	const struct of_device_id	*of_match_table;
	const struct dev_pm_ops *pm;
};

~~~



### bus总线

### class类

## platform虚拟总线

Platform总线参考代码（内核源码）
drivers\base\platform.c
include\linux\platform_device.h

### Platform驱动

~~~c
struct platform_driver {
		int (*probe)(struct platform_device *);		//平台驱动初始化时会调用该函数
		int (*remove)(struct platform_device *);	//平台驱动卸载时会调用该函数
		void (*shutdown)(struct platform_device *);	//平台驱动关闭时会调用该函数
		int (*suspend)(struct platform_device *, pm_message_t state);
		int (*resume)(struct platform_device *);	//平台驱动释放时会调用该函数
		struct device_driver driver;			//内置device_driver结构体
		const struct platform_device_id *id_table;	//该设备驱动支持的设备列表，通过该指针指向platform_device_id类型的数组
	};
~~~

* 注册函数：platform_driver_register
* 完成注册以后再 /sys/bus/platform/drivers目录下会看到 dev.name的文件

### Platform设备

~~~c
struct platform_device {
	const char	*name;				//platform设备名字
	int		id;				//用于区分设备名相同时将在设备名后面追加该ID
	struct device	dev;				//内置device结构体
	u32		num_resources;			//资源结构体数量
	struct resource	*resource;			//指向资源结构体数组
	const struct platform_device_id	*id_entry;	//用来进行与设备驱动匹配用的id_table表
	struct pdev_archdata	archdata;		//私有数据，添加自己的东西
};
~~~

* 注册函数：platform_add_devices
* 完成注册以后将会在/sys/bus/platform/devices目录下会看到dev.name的文件夹

### Platform匹配机制

~~~c
static int platform_match(struct device *dev, struct device_driver *drv)
{
	struct platform_device *pdev = to_platform_device(dev);
	struct platform_driver *pdrv = to_platform_driver(drv);

	/* When driver_override is set, only bind to the matching driver */
	if (pdev->driver_override)
		return !strcmp(pdev->driver_override, drv->name);

	/* Attempt an OF style match first */
	if (of_driver_match_device(dev, drv))
		return 1;

	/* Then try ACPI style match */
	if (acpi_driver_match_device(dev, drv))
		return 1;

	/* Then try to match against the id table */
	if (pdrv->id_table)
		return platform_match_id(pdrv->id_table, pdev) != NULL;

	/* fall-back to driver name match */
	return (strcmp(pdev->name, drv->name) == 0);
}	

//设置驱动程序的私有数据
static inline void platform_set_drvdata(struct platform_device *pdev,
					void *data)
{
	dev_set_drvdata(&pdev->dev, data);
}
~~~

![1](.\2.png)

## 
