MISC 驱动也叫做杂项驱动，也就是当我们板子上的某些外设无法进行分类的时候就可以使用 MISC 驱动。 MISC 驱动**其实就是最简单的字符设备驱动**，**通常嵌套在 platform 总线驱动**中，实现复杂的驱动。

# MISC驱动结构

- 所有的 MISC 设备驱动的主设备号都为10，不同的设备使用不同的从设备号
- MISC 设备会自动创建 cdev，不需要像我们以前那样手动创建，因此采用 MISC 设备驱动可以简化字符设备驱动的编写
- 只需要填充`miscdevice`结构体
- 嵌套miscdevice到platform驱动中，将注册注销设备操作放在platform_driver的`probe`和`remove`函数中

## miscdevice结构体
定义一个 MISC 设备(miscdevice 类型)以后我们需要设置 minor、 name 和 fops 这三个成员变量，其他不用管：
```c
struct miscdevice  {
    int minor; //从设备号,可以使用预定义宏
    const char *name;//设备名称
    const struct file_operations *fops;//设备操作函数
    struct list_head list;
    struct device *parent;
    struct device *this_device;
    const struct attribute_group **groups;
    const char *nodename;
    umode_t mode;
};
```

## MISC注册注销函数
如下两个函数可以自动完成所有操作，不再需要手动写驱动注册注销的众多逻辑
```c
//自动完成：
// alloc_chrdev_region(); /* 申请设备号 */
// cdev_init(); /* 初始化 cdev */
// cdev_add(); /* 添加 cdev */
// class_create(); /* 创建类 */
// device_create(); /* 创建设备 */
int misc_register(struct miscdevice *misc);

//自动完成：
// cdev_del(); /* 删除 cdev */
// unregister_chrdev_region(); /* 注销设备号 */
// device_destroy(); /* 删除设备 */
// class_destroy(); /* 删除类 */
void misc_deregister(struct miscdevice *misc);
```

# 后记
其他子系统框架，先暂时不更新了，可以直接参考一个嵌入式的文档。讲解的很详细，如果只是复制搬运到这里没什么意思，还是等以后有这方面实际的开发经验之后再记录更深刻。附文档：**《IMX6U嵌入式Linux驱动开发指南V1.5.1》56-73章**​
