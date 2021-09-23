# 1 Linux文件系统
在设备驱动程序的设计中， 一般而言， 会关心**file和inode这两个结构体**。

## 1.1 file结构体
file结构体代表一个打开的文件， 系统中每个打开的文件在内核空间都有一个关联的struct file。它由内核在打开文件时创建， 并传递给在文件上进行操作的任何函数。 在文件的所有实例都关闭后， 内核释放这个数据结构。
文件读/写模式**mode**、 标志**f_flags**都是设备驱动关心的内容 ，私有数据指针**private_data**在设备驱动中被广泛应用， 大多被指向设备驱动自定义以用于描述设备的结构体。
file结构体的定义如下：
```c
//include/linux/fs.h文件中
struct file {
    union {
        struct llist_node   fu_llist;
        struct rcu_head     fu_rcuhead;
    } f_u;
    struct path     f_path;
    struct inode        *f_inode;   /* cached value */
    const struct file_operations    *f_op; //和文件关联的操作

    /*
     * Protects f_ep_links, f_flags.
     * Must not be taken from IRQ context.
     */
    spinlock_t      f_lock;
    enum rw_hint        f_write_hint;
    atomic_long_t       f_count;
    unsigned int        f_flags;   //文件标志，如O_RDONLY,O_NONBLOCK,O_SYNC
    fmode_t         f_mode;        //文件读写权限，FMODE_READ和FMODE_WRITE
    struct mutex        f_pos_lock;
    loff_t          f_pos;          //当前读写位置
    struct fown_struct  f_owner;
    const struct cred   *f_cred;
    struct file_ra_state    f_ra;

    u64         f_version;
#ifdef CONFIG_SECURITY
    void            *f_security;
#endif
    /* needed for tty driver, and maybe others */
    void            *private_data;    //文件私有数据

#ifdef CONFIG_EPOLL
    /* Used by fs/eventpoll.c to link all the hooks to this file */
    struct list_head    f_ep_links;
    struct list_head    f_tfile_llink;
#endif /* #ifdef CONFIG_EPOLL */
    struct address_space    *f_mapping;
    errseq_t        f_wb_err;
} __randomize_layout
  __attribute__((aligned(4)));  /* lest something weird decides that 2 is OK */
```

## 1.2 inode结构体
inode包含文件访问权限、 属主、 组、 大小、 生成时间、 访问时间、 最后修改时间等信息。 它是**Linux管理文件系统的最基本单位**， 也是文件系统连接任何子目录、 文件的桥梁。主要属性如下：

```c
struct inode {
    umode_t         i_mode;       //inode权限
    unsigned short      i_opflags; 
    kuid_t          i_uid;//拥有者id
    kgid_t          i_gid;//拥有者组id
    unsigned int        i_flags;
	......
    dev_t           i_rdev;//若是设备文件， 此字段将记录设备的设备号
    loff_t          i_size;//文件大小
    struct timespec     i_atime;//最后一次存取时间
    struct timespec     i_mtime;//最近一次修改时间
    struct timespec     i_ctime;//创建时间
    ......
    blkcnt_t        i_blocks;//inode使用的block数，一个block为512字节
	......
    union {
        struct pipe_inode_info  *i_pipe;
        struct block_device *i_bdev; //若是块设备， 为其对应的 block_device 结构体指针
        struct cdev     *i_cdev;//若是字符设备， 为其对应的 cdev 结构体指针
        char            *i_link;
        unsigned        i_dir_seq;
    };

   ......
} __randomize_layout;
```

i_rdev字段包含设备编号。 Linux内核设备编号分为主设备编号和次设备编号， 前者为dev_t的高12位， 后者为dev_t的低20位。如下函数可以获取主次设备号：
```c
unsigned int iminor(struct inode *inode);
unsigned int imajor(struct inode *inode);
```


# 2 udev设备管理
kernel 2.4引入的devfs，在Linux 2.6内核中，被认为是过时的方法，并最终被抛弃，由udev代替。
udev完全在用户态工作，利用设备加入或移除时内核所发送的热插拔事件（Hotplug Event，监听**PF_NETLINK **socket，代码示例如下）来工作，可以根据系统中硬件设备状态来创建或者删除设备文件。udev的设备命名策略、权限控制和事件处理都是在用户态下完成的，它利用从内核收到的信息来进行创建设备文件节点等工作。
> **注：**在嵌入式系统中， 也可以用udev的**轻量级版本mdev**， mdev集成于busybox中。 在编译busybox的时候， 选中mdev相关项目即可。另外**Android采用的是vold**，和udev一样的机制，也是监听netlink套接字。

对于冷插拔的设备， Linux内核提供了sysfs下面一个**uevent节点**， 可以往该节点写一个“add”， 导致**内核重新发送netlink**，之后udev就可以收到冷插拔的netlink消息了。
```c
//netlink套接字监听热插拔事件
#include <linux/netlink.h>
#include <sys/poll.h> 
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>

static void die(char* s)
{
	write(2, s, strlen(s));
	exit(1);
}

int main()
{
	struct sockaddr_nl nls;
	struct pollfd pfd;
	char buf[512];
	memset(&nls, 0, sizeof(struct sockaddr_nl));
	
	pfd.events = POLLIN; //设置监听插入事件
	pfd.fd = socket(PF_NETLINK, SOCK_DGRAM, NETLINK_KOBJECT_UEVENT);
	if (pfd.fd == -1)
		die("not root\n");
	
	nls.nl_family = AF_NETLINK;//设置netlink监听，不是普通socket的ip类型
	nls.nl_pid = getpid();
	nls.nl_groups = -1;
	if (bind(pfd.fd, (void*)&nls, sizeof(struct sockaddr_nl)))
		die("bind socket failed\n");
	
	while(poll(&pfd, 1, -1))
	{
		int i, len = recv(pfd.fd, buf, sizeof(buf), MSG_DONTWAIT);
		if (len == -1)
			die("recv nothing\n");
		
		i = 0;
		while(i < len)
		{
			printf("%s\n", buf+i);
			i+= strlen(buf+i) + 1;
		}
	}
	die("poll\n");
	return 0;
}
```

## sysfs文件系统
Linux 2.6以后的内核引入了sysfs文件系统， sysfs被看成是与proc、 devfs和devpty同类别的文件系统， 该文件系统是一个**虚拟的文件系统**， 它可以产生一个包括所有系统硬件的层级视图， 与提供进程和状态信息的proc文件系统十分类似。挂载位置为`/sys`。sysfs把连接在系统上的设备和总线组织成为一个分级的文件， 它们可以由用户空间存取， 向用户空间导出内核数据结构以及它们的属性。sysfs中的目录来源于bus_type、device_driver device， 而目录中的文件则来源于对应的attribute结构体。
在Linux内核中， 分别使用**bus_type、 device_driver和device**来描述总线、 驱动和设备， 这3个结构体定义于`include/linux`目录中。驱动和设备在内核中时分开注册的，最后通过bus_type中的**match()**函数绑定在一起。

## udev工作过程

- 当内核检测到系统中出现了新设备后， 内核会通过**netlink套接字**发送uevent。
- udev获取内核发送的信息， 进行**规则的匹配**。 匹配的事物包括SUBSYSTEM、ACTION、atttribute、内核提供的名称（通过KERNEL=） 以及其他的环境变量（_我们可以根据这些信息， 创建一个规则， 以便每次插入设备的时候，添加一个设备文件_）。

### udev规则文件
udev的规则文件以行为单位， 以“#”开头的行代表注释行。 其余的每一行代表一个规则。 每个规则分成一个或多个匹配部分和赋值部分。

- 匹配部分关键字： ACTION(行为)、KERNEL(匹配内核设备名)、BUS(匹配总线类型)、SUBSYSTEM(匹配子系统名)、ATTR(属性)
- 赋值部分关键字： NAME(创建的设备文件名)、SYMLINK(符号创建链接名)、OWNER(设置设备的所有者)、GROUP(设置设备的组)、 IMPORT(调用外部程序)、 MODE(节点访问权限)


规则文件示例：
```basic
# 当系统中出现的新硬件属于net子系统范畴， 系统对该硬件采取的动作是“add”这个硬件， 且这个硬件的“address”属性信息等于“08:00:27:35:be:ff”， “dev_id”属性等于“0x0”、 “type”属性为1
SUBSYSTEM=="net", ACTION=="add", DRIVERS==" *", ATTR{address}=="08:00:27:35:be:ff", ATTR{dev_id}=="0x0", ATTR{type}=="1", KERNEL=="eth*", NAME="eth1"
```

# 3 udev点灯示例代码
下面是一个简单的开发板上led点灯代码，主要用到了如下两个知识点：

- 定义有关寄存器物理地址，然后使用`ioremap`函数进行**内存映射，得到对应的虚拟地址**，最后操作寄存器对应的虚拟地址完成对 GPIO 的初始化
- udev机制在加载模块时自动创建设备文件
```c
#include <linux/types.h>
#include <linux/kernel.h>
#include <linux/delay.h>
#include <linux/ide.h>
#include <linux/init.h>
#include <linux/module.h>
#include <linux/errno.h>
#include <linux/gpio.h>
#include <linux/cdev.h>
#include <linux/device.h>

#include <asm/mach/map.h>
#include <asm/uaccess.h>
#include <asm/io.h>

#define NEWCHRLED_CNT			1		  	/* 设备号个数 */
#define NEWCHRLED_NAME			"newchrled"	/* 名字 */
#define LEDOFF 					0			/* 关灯 */
#define LEDON 					1			/* 开灯 */
 
/* 寄存器物理地址 */
#define CCM_CCGR1_BASE				(0X020C406C)	
#define SW_MUX_GPIO1_IO03_BASE		(0X020E0068)
#define SW_PAD_GPIO1_IO03_BASE		(0X020E02F4)
#define GPIO1_DR_BASE				(0X0209C000)
#define GPIO1_GDIR_BASE				(0X0209C004)

/* 映射后的寄存器虚拟地址指针 */
static void __iomem *IMX6U_CCM_CCGR1;
static void __iomem *SW_MUX_GPIO1_IO03;
static void __iomem *SW_PAD_GPIO1_IO03;
static void __iomem *GPIO1_DR;
static void __iomem *GPIO1_GDIR;

/* newchrled设备结构体 */
struct newchrled_dev{
	dev_t devid;			/* 设备号 	 */
	struct cdev cdev;		/* cdev 	*/
	struct class *class;		/* 类 		*/
	struct device *device;	/* 设备 	 */
	int major;				/* 主设备号	  */
	int minor;				/* 次设备号   */
};

struct newchrled_dev newchrled;	/* led设备 */

/*
 * @description		: LED打开/关闭
 * @param - sta 	: LEDON(0) 打开LED，LEDOFF(1) 关闭LED
 * @return 			: 无
 */
void led_switch(u8 sta)
{
	u32 val = 0;
	if(sta == LEDON) {
		val = readl(GPIO1_DR);
		val &= ~(1 << 3);	
		writel(val, GPIO1_DR);
	}else if(sta == LEDOFF) {
		val = readl(GPIO1_DR);
		val|= (1 << 3);	
		writel(val, GPIO1_DR);
	}	
}

/*
 * @description		: 打开设备
 * @param - inode 	: 传递给驱动的inode
 * @param - filp 	: 设备文件，file结构体有个叫做private_data的成员变量
 * 					  一般在open的时候将private_data指向设备结构体。
 * @return 			: 0 成功;其他 失败
 */
static int led_open(struct inode *inode, struct file *filp)
{
	filp->private_data = &newchrled; /* 设置私有数据 */
	return 0;
}

/*
 * @description		: 从设备读取数据 
 * @param - filp 	: 要打开的设备文件(文件描述符)
 * @param - buf 	: 返回给用户空间的数据缓冲区
 * @param - cnt 	: 要读取的数据长度
 * @param - offt 	: 相对于文件首地址的偏移
 * @return 			: 读取的字节数，如果为负值，表示读取失败
 */
static ssize_t led_read(struct file *filp, char __user *buf, size_t cnt, loff_t *offt)
{
	return 0;
}

/*
 * @description		: 向设备写数据 
 * @param - filp 	: 设备文件，表示打开的文件描述符
 * @param - buf 	: 要写给设备写入的数据
 * @param - cnt 	: 要写入的数据长度
 * @param - offt 	: 相对于文件首地址的偏移
 * @return 			: 写入的字节数，如果为负值，表示写入失败
 */
static ssize_t led_write(struct file *filp, const char __user *buf, size_t cnt, loff_t *offt)
{
	int retvalue;
	unsigned char databuf[1];
	unsigned char ledstat;

	retvalue = copy_from_user(databuf, buf, cnt);
	if(retvalue < 0) {
		printk("kernel write failed!\r\n");
		return -EFAULT;
	}

	ledstat = databuf[0];		/* 获取状态值 */

	if(ledstat == LEDON) {	
		led_switch(LEDON);		/* 打开LED灯 */
	} else if(ledstat == LEDOFF) {
		led_switch(LEDOFF);	/* 关闭LED灯 */
	}
	return 0;
}

/*
 * @description		: 关闭/释放设备
 * @param - filp 	: 要关闭的设备文件(文件描述符)
 * @return 			: 0 成功;其他 失败
 */
static int led_release(struct inode *inode, struct file *filp)
{
	return 0;
}

/* 设备操作函数 */
static struct file_operations newchrled_fops = {
	.owner = THIS_MODULE,
	.open = led_open,
	.read = led_read,
	.write = led_write,
	.release = 	led_release,
};

/*
 * @description	: 驱动出口函数
 * @param 		: 无
 * @return 		: 无
 */
static int __init led_init(void)
{
	u32 val = 0;

	/* 初始化LED */
	/* 1、寄存器地址映射 */
  	IMX6U_CCM_CCGR1 = ioremap(CCM_CCGR1_BASE, 4);
	SW_MUX_GPIO1_IO03 = ioremap(SW_MUX_GPIO1_IO03_BASE, 4);
  	SW_PAD_GPIO1_IO03 = ioremap(SW_PAD_GPIO1_IO03_BASE, 4);
	GPIO1_DR = ioremap(GPIO1_DR_BASE, 4);
	GPIO1_GDIR = ioremap(GPIO1_GDIR_BASE, 4);

	/* 2、使能GPIO1时钟 */
	val = readl(IMX6U_CCM_CCGR1);
	val &= ~(3 << 26);	/* 清楚以前的设置 */
	val |= (3 << 26);	/* 设置新值 */
	writel(val, IMX6U_CCM_CCGR1);

	/* 3、设置GPIO1_IO03的复用功能，将其复用为
	 *    GPIO1_IO03，最后设置IO属性。
	 */
	writel(5, SW_MUX_GPIO1_IO03);
	
	/*寄存器SW_PAD_GPIO1_IO03设置IO属性
	 *bit 16:0 HYS关闭
	 *bit [15:14]: 00 默认下拉
     *bit [13]: 0 kepper功能
     *bit [12]: 1 pull/keeper使能
     *bit [11]: 0 关闭开路输出
     *bit [7:6]: 10 速度100Mhz
     *bit [5:3]: 110 R0/6驱动能力
     *bit [0]: 0 低转换率
	 */
	writel(0x10B0, SW_PAD_GPIO1_IO03);

	/* 4、设置GPIO1_IO03为输出功能 */
	val = readl(GPIO1_GDIR);
	val &= ~(1 << 3);	/* 清除以前的设置 */
	val |= (1 << 3);	/* 设置为输出 */
	writel(val, GPIO1_GDIR);

	/* 5、默认关闭LED */
	val = readl(GPIO1_DR);
	val |= (1 << 3);	
	writel(val, GPIO1_DR);

	/* 注册字符设备驱动 */
	/* 1、创建设备号 */
	if (newchrled.major) {		/*  定义了设备号 */
		newchrled.devid = MKDEV(newchrled.major, 0);
		register_chrdev_region(newchrled.devid, NEWCHRLED_CNT, NEWCHRLED_NAME);
	} else {						/* 没有定义设备号 */
		alloc_chrdev_region(&newchrled.devid, 0, NEWCHRLED_CNT, NEWCHRLED_NAME);	/* 申请设备号 */
		newchrled.major = MAJOR(newchrled.devid);	/* 获取分配号的主设备号 */
		newchrled.minor = MINOR(newchrled.devid);	/* 获取分配号的次设备号 */
	}
	printk("newcheled major=%d,minor=%d\r\n",newchrled.major, newchrled.minor);	
	
	/* 2、初始化cdev */
	newchrled.cdev.owner = THIS_MODULE;
	cdev_init(&newchrled.cdev, &newchrled_fops);
	
	/* 3、添加一个cdev */
	cdev_add(&newchrled.cdev, newchrled.devid, NEWCHRLED_CNT);

	/* 4、创建类 */
	newchrled.class = class_create(THIS_MODULE, NEWCHRLED_NAME);
	if (IS_ERR(newchrled.class)) {
		return PTR_ERR(newchrled.class);
	}

	/* 5、创建设备 */
	newchrled.device = device_create(newchrled.class, NULL, newchrled.devid, NULL, NEWCHRLED_NAME);
	if (IS_ERR(newchrled.device)) {
		return PTR_ERR(newchrled.device);
	}
	
	return 0;
}

/*
 * @description	: 驱动出口函数
 * @param 		: 无
 * @return 		: 无
 */
static void __exit led_exit(void)
{
	/* 取消映射 */
	iounmap(IMX6U_CCM_CCGR1);
	iounmap(SW_MUX_GPIO1_IO03);
	iounmap(SW_PAD_GPIO1_IO03);
	iounmap(GPIO1_DR);
	iounmap(GPIO1_GDIR);

	/* 注销字符设备驱动 */
	cdev_del(&newchrled.cdev);/*  删除cdev */
	unregister_chrdev_region(newchrled.devid, NEWCHRLED_CNT); /* 注销设备号 */

	device_destroy(newchrled.class, newchrled.devid);
	class_destroy(newchrled.class);
}

module_init(led_init);
module_exit(led_exit);
MODULE_LICENSE("GPL");
MODULE_AUTHOR("barretren");
```
