# spi数据结构

[RK3399平台开发系列讲解（SPI子系统）4.13、SPI子系统之数据结构的解析_rk3399 有没有 qspi-CSDN博客](https://xuesong.blog.csdn.net/article/details/113824436)![](.\pic\2024-10-23-spi-driver\1718244342296-8be7f17e-ed9c-451c-b863-b4e17cee7f22.png)

## spi_bus_type SPI 总线
## spi_driver 外设驱动枚举
## spi_device 外设设备信息描述
```c
#define spi_master			spi_controller
struct spi_device {
	struct device		dev;
	struct spi_controller	*controller;	//struct spi_master	master	父节点，控制器设备节点
	u32			max_speed_hz;		//最大速度
	u8			chip_select;	//片选信号
	u8			bits_per_word;		//每帧的字节数
	u16			mode;		//spi模式
#define	SPI_CPHA	0x01			/* clock phase */
#define	SPI_CPOL	0x02			/* clock polarity */
#define	SPI_MODE_0	(0|0)			/* (original MicroWire) */
#define	SPI_MODE_1	(0|SPI_CPHA)
#define	SPI_MODE_2	(SPI_CPOL|0)
#define	SPI_MODE_3	(SPI_CPOL|SPI_CPHA)
#define	SPI_CS_HIGH	0x04			/* chipselect active high? */
#define	SPI_LSB_FIRST	0x08			/* per-word bits-on-wire */
#define	SPI_3WIRE	0x10			/* SI/SO signals shared */
#define	SPI_LOOP	0x20			/* loopback mode */
#define	SPI_NO_CS	0x40			/* 1 dev/bus, no chipselect */
#define	SPI_READY	0x80			/* slave pulls low to pause */
#define	SPI_TX_DUAL	0x100			/* transmit with 2 wires */
#define	SPI_TX_QUAD	0x200			/* transmit with 4 wires */
#define	SPI_RX_DUAL	0x400			/* receive with 2 wires */
#define	SPI_RX_QUAD	0x800			/* receive with 4 wires */
	int			irq;
	void			*controller_state;
	void			*controller_data;
	char			modalias[SPI_NAME_SIZE];
	int			cs_gpio;	/* chip select gpio */

	/* the statistics */
	struct spi_statistics	statistics;

	/*
	 * likely need more hooks for more protocol options affecting how
	 * the controller talks to each chip, like:
	 *  - memory packing (12 bit samples into low bits, others zeroed)
	 *  - priority
	 *  - drop chipselect after each word
	 *  - chipselect delays
	 *  - ...
	 */
};
```

# 子系统框架

[【驱动】SPI驱动分析(四)-关键API解析 - 学习，积累，成长 - 博客园](https://www.cnblogs.com/dongxb/p/17868580.html)

## SPI 匹配框架：
![](.\pic\2024-10-23-spi-driver\1652603351165-37cfe16e-4a7a-4c63-a85a-2c3ed6c5a4a3.png)

![画板](.\pic\2024-10-23-spi-driver\1718268362778-ab26567a-2559-47e4-a27f-bd8e9608de7a.jpeg)

### APP 调用流程
![](C:\Users\xiaodq\Desktop\新建文件夹\pic\2024-10-23-spi-driver\1729661536506-e2d07017-d0d8-45b3-ae7b-5b33f2408ed6.png)

![](C:\Users\xiaodq\Desktop\新建文件夹\pic\2024-10-23-spi-driver\1718022963517-5cb42138-921b-432b-afc1-5f38616e3c4b.png)

**参考：**

[ESP8089-SPI](https://www.yuque.com/yunxiaoyuji-i3zpe/mogigi/gq7siac52zofhi7g#Am4aB)

## 1.设备驱动
`**<font style="color:rgb(51, 51, 51);">spi_driver</font>**`<font style="color:rgb(51, 51, 51);">，里面有</font>`<font style="color:rgb(51, 51, 51);">id_table</font>`<font style="color:rgb(51, 51, 51);">表示能支持哪些SPI设备，有</font>`<font style="color:rgb(51, 51, 51);">probe</font>`<font style="color:rgb(51, 51, 51);">函数</font>

`**<font style="color:rgb(51, 51, 51);">spi_device</font>**`<font style="color:rgb(51, 51, 51);">，用来</font>**<font style="color:rgb(51, 51, 51);">描述SPI设备</font>**<font style="color:rgb(51, 51, 51);">，比如它的片选引脚、频率</font>

+ <font style="color:rgb(51, 51, 51);">可以由S</font>**<font style="color:rgb(51, 51, 51);">PI控制器驱动</font>**<font style="color:rgb(51, 51, 51);">程序解析设备树后创建、注册</font>`**<font style="color:rgb(51, 51, 51);">spi_device</font>**`
+ <font style="color:rgb(51, 51, 51);">可以使用</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(243, 244, 244);">spi_register_board_info</font>`<font style="color:rgb(51, 51, 51);">创建、注册</font>`<font style="color:rgb(51, 51, 51);">spi_device</font>`

### 1.1`spi_driver`设备驱动
<font style="color:rgb(51, 51, 51);">使用spi_driver结构体描述SPI设备驱动：</font>

 实现：`drv_open `、`drv_close `、`drv_read `、`drv_write`

类似于`i2c_driver`:[10.IIC子系统总线驱动](https://www.yuque.com/yunxiaoyuji-i3zpe/ae0qgc/byllfv#TUvOb)

```c
struct spi_driver {  
    const struct spi_device_id *id_table; 	//用于匹配
    int    (*probe)(struct spi_device *spi);  
    int    (*remove)(struct spi_device *spi);  
    struct device_driver    driver;  

}; 
```

`int spi_register_driver(struct spi_driver *sdrv);`    //设备注册

`void spi_unregister_driver(struct spi_driver *sdrv);`   //设备注销

实例：

```c
/* probe 函数 */
static int xxx_probe(struct spi_device *spi)
{
    return 0;
}
/* remove 函数 */
static int xxx_remove(struct spi_device *spi)
{
    return 0;
}
/* 传统匹配方式 ID 列表 */
static const struct spi_device_id xxx_id[] = {
    {"xxx", 0},
    {}
};
/* 设备树匹配列表 */
static const struct of_device_id xxx_of_match[] = {
    { .compatible = "xxx" },
    { /* Sentinel */ }
};
/* SPI 驱动结构体 */
static struct spi_driver xxx_driver = {
    .probe = xxx_probe,
    .remove = xxx_remove,
    .driver = {
        .owner = THIS_MODULE,
        .name = "xxx",
        .of_match_table = xxx_of_match,  
    },
    .id_table = xxx_id,
};
```

### <font style="color:rgb(51, 51, 51);">1.2</font>`<font style="color:rgb(51, 51, 51);">spi_device</font>`<font style="color:rgb(51, 51, 51);">设备数据</font>：
**在内核层匹配完成后自动创建**。

<font style="color:rgb(51, 51, 51);">使用</font>`<font style="color:rgb(51, 51, 51);">spi_device</font>`<font style="color:rgb(51, 51, 51);">结构体描述SPI设备，里面记录有设备的片选引脚、频率、挂在哪个SPI控制器下面：</font>

```c
struct spi_device {  
    struct device       dev;  /*代表该spi设备的device结构*/
    struct spi_master   *master;  /*指向该spi设备所使用的控制器*/
    u32         max_speed_hz;  /*该设备的最大工作时钟频率*/
    u8          chip_select;   /*在控制器中的片选引脚编号索引*/
    u8          mode;     /*设备的工作模式，包括时钟格式，片选信号的有效电平等等*/
    u8          bits_per_word;   /*设备每个单位数据所需要的比特数*/
    int         irq;   /*设备使用的irq编号*/
   void       *controller_state;   
   void       *controller_data;   
   char         modalias[32];   /*该设备的名字，用于spi总线和驱动进行配对*/
}; 
```

```cpp
struct spi_device *spi_new_device(struct spi_controller *, struct spi_board_info *);
```

### 实例：
[驱动框架](https://www.yuque.com/yunxiaoyuji-i3zpe/ae0qgc/lx54tg)

### 设备和驱动匹配流程：
```cpp
static int spi_match_device(struct device *dev, struct device_driver *drv)
{
	const struct spi_device	*spi = to_spi_device(dev);
	const struct spi_driver	*sdrv = to_spi_driver(drv);

    /* 1.强制匹配device中driver_override和driver中的name*/
	/* Check override first, and if set, only use the named driver */
	if (spi->driver_override)
		return strcmp(spi->driver_override, drv->name) == 0;

	/* Attempt an OF style match */
	if (of_driver_match_device(dev, drv))
		return 1;

	/* Then try ACPI */
	if (acpi_driver_match_device(dev, drv))
		return 1;

	if (sdrv->id_table)
		return !!spi_match_id(sdrv->id_table, spi->modalias);

	return strcmp(spi->modalias, drv->name) == 0;
}
```

1. <font style="color:rgb(51, 51, 51);">强制匹配device中driver_override和driver中的name</font>
2. <font style="color:rgb(51, 51, 51);">尝试使用设备树风格的匹配。调用</font><font style="color:rgb(192, 52, 29);background-color:rgb(251, 229, 225);">of_driver_match_device</font><font style="color:rgb(51, 51, 51);">函数，检查设备与驱动程序是否匹配。如果匹配成功，则返回1。</font>
3. <font style="color:rgb(51, 51, 51);">如果设备与驱动程序未通过设备树匹配，尝试使用ACPI匹配。调用</font><font style="color:rgb(192, 52, 29);background-color:rgb(251, 229, 225);">acpi_driver_match_device</font><font style="color:rgb(51, 51, 51);">函数，检查设备与驱动程序是否匹配。如果匹配成功，则返回</font>
4. <font style="color:rgb(51, 51, 51);">如果SPI驱动程序具有</font><font style="color:rgb(192, 52, 29);background-color:rgb(251, 229, 225);">id_table</font><font style="color:rgb(51, 51, 51);">字段，则使用</font><font style="color:rgb(192, 52, 29);background-color:rgb(251, 229, 225);">spi_match_id</font><font style="color:rgb(51, 51, 51);">函数尝试通过ID表进行匹配。如果匹配成功，则返回1。</font>
5. <font style="color:rgb(51, 51, 51);">如果以上匹配方法都失败，则通过比较设备的</font><font style="color:rgb(192, 52, 29);background-color:rgb(251, 229, 225);">modalias</font><font style="color:rgb(51, 51, 51);">和驱动程序的</font><font style="color:rgb(192, 52, 29);background-color:rgb(251, 229, 225);">name</font><font style="color:rgb(51, 51, 51);">字段来进行匹配。如果两者相等，则返回1；否则返回0。</font>

---

## 2.spi核心层
 spi驱动  spi控制器  spi设备的相关结构体，及注册注销函数封装

### 注册设备信息：
```c
struct spi_board_info {
    char modalias[KOBJ_NAME_LEN]; //设备名和模块名是对应的，如同platform_bus “modalias”一般是驱动名
    const void *platform_data;	//对应 spi_device.dev.platform_data
    void *controller_data;	//对应 spi_device.controller_data
    int irq;		//中断号
    u32 max_speed_hz;	//最大速度
    u16 bus_num;//bus_num 跟开发板相关，与之后会注册的 spi_master 的 bus_num匹配
    u16 chip_select;	//反映从设备芯片与主设备的连接
    u8 mode;		//对应 spi_device.mode
};

```

声明：`int spi_register_board_info(struct spi_board_info const *info, unsigned n);`

### ##在spi内核代码中添加spi设备信息
```c
struct spi_board_info spi0_mcp2515_info[] __initdata = {
    [0] = {
        .modalias       = "spi_mcp2515",
        .irq            = IRQ_GPIO_START + PAD_GPIO_B + 25,
        .max_speed_hz   = 10 * 1000 * 1000,
        .bus_num        = 0, 
        .chip_select    = 0, 
        .mode           = SPI_MODE_0,                             
        .controller_data = &spi0_info[0],
    }   
};
spi_register_board_info(spi0_mcp2515_info, ARRAY_SIZE(spi0_mcp2515_info));
```

---

## 3.spi控制器驱动层
<font style="color:rgb(51, 51, 51);">基于"platform平台总线设备驱动"模型来</font>实现spi控制器的驱动，

+ <font style="color:rgb(51, 51, 51);">在设备树里描述SPI控制器的硬件信息，在设备树子节点里描述挂在下面的SPI设备的信息</font>
+ <font style="color:rgb(51, 51, 51);">在</font>`<font style="color:rgb(51, 51, 51);">platform_driver</font>`<font style="color:rgb(51, 51, 51);">中提供一个</font>`<font style="color:rgb(51, 51, 51);">probe</font>`<font style="color:rgb(51, 51, 51);">函数</font>
    - <font style="color:rgb(51, 51, 51);">它会注册一个</font>`<font style="color:rgb(51, 51, 51);">spi_master</font>`
    - <font style="color:rgb(51, 51, 51);">还会解析设备树子节点，创建</font>`<font style="color:rgb(51, 51, 51);">spi_device</font>`<font style="color:rgb(51, 51, 51);">结构体</font>

### 3.1spi_master结构体
<font style="color:rgb(51, 51, 51);">使用</font>`<font style="color:rgb(51, 51, 51);">spi_master</font>`<font style="color:rgb(51, 51, 51);">结构体描述SPI控制器，里面最重要的成员就是</font><font style="color:rgb(51, 51, 51);background-color:rgb(243, 244, 244);">transfer</font><font style="color:rgb(51, 51, 51);">函数指针：</font>

```c
struct spi_master {  
    struct device   dev;  
    s16      bus_num;  /*bus_num为该控制器对应的SPI总线号*/
    u16      num_chipselect; /*num_chipselect 控制器支持的片选数量，即能支持多少个spi设备*/
  /*setup函数是设置SPI总线的模式，时钟等的初始化函数， 针对设备设置SPI的工作时钟及数据传输模式等,在spi_add_device函数中调用*/
    int      (*setup)(struct spi_device *spi);  
    /*实现SPI总线读写方法的函数。实现数据的双向传输，可能会睡眠*/
    int      (*transfer)(struct spi_device *spi, struct spi_message *mesg); 
    void     (*cleanup)(struct spi_device *spi);  /*注销时候调用*/
    struct list_head	queue;
};
```

### 3.2函数
声明：`int spi_register_master(struct spi_master *master)`  //控制器端注册

```c
#define spi_register_master(_ctlr)	spi_register_controller(_ctlr)
```

声明：`void spi_unregister_master(struct spi_master *master)`//控制器端注销

## 4.传输数据：
### 4.1消息结构体：
```c
struct spi_transfer {
    const void	*tx_buf;
    void	*rx_buf;
    unsigned	len;
    dma_addr_t	tx_dma;
    dma_addr_t	rx_dma;
    unsigned	cs_change:1;
    u8		bits_per_word;
    u16		delay_usecs;
    u32		speed_hz;
    struct list_head transfer_list;
};
```

```c
struct spi_message {
    struct list_head	transfers;
    struct spi_device	*spi;
    unsigned		is_dma_mapped:1;
    void		 (*complete)(void *context);
    void		*context;
    unsigned		actual_length;
    int		status;
    struct list_head	queue;
    void		*state;
};
```

1.分配一个自带数个`spi_transfer`结构的`spi_message`

`static inline struct spi_message *spi_message_alloc(unsigned ntrans, gfp_t flags)；`

2.初始化`spi_message`结构

`static inline void spi_message_init(struct spi_message *m)；`

3.将`spi_transfer`加入到`spi_message`中

`static inline void   spi_message_add_tail(struct spi_transfer *t,struct spi_message *m)；`

4.移除一个`spi_transfer`

`static inline void  spi_transfer_del(struct spi_transfer *t)；`

5.发起一个`spi_message`的传送操作

异步版本：`int spi_async(struct spi_device *spi, struct spi_message *message);`

同步版本：`int spi_sync(struct spi_device *spi, struct spi_message *message);`

## 4.2读写操作：
spi读操作

`static inline int spi_write(struct spi_device *spi, const void *buf, size_t len)`

spi写操作

`static inline int spi_read(struct spi_device *spi, void *buf, size_t len)`

spi写-读操作

`int spi_write_then_read(struct spi_device *spi,const void *txbuf,unsigned n_tx,void *rxbuf, unsigned n_rx)`



# spi核心层

[Linux驱动修炼之道-SPI驱动框架源码分析(上,中,下) - Red_Point - 博客园](https://www.cnblogs.com/tureno/articles/6019558.html)

## spi 总线spi_bus_type

# SPI设备驱动



## SPI 设备驱动层设备匹配
使用 spi 总线进行匹配，使用结构体`spi_bus_type`

## 注册 spi driver
## SPI 传输接口：
`<font style="color:rgb(51, 51, 51);background-color:rgb(243, 244, 244);">include\linux\spi\spi.h</font>`

### SPI 传输数据管理：
SPI 传输数据可以使用多个`spi_message` 管理多个要传输的数据`spi_transfer`。

![画板](C:\Users\xiaodq\Desktop\新建文件夹\pic\2024-10-23-spi-driver\1727007072760-4a4d5c3e-663d-4e23-bf26-01a5b9e1fabb.jpeg)

```c
int spi_write_then_read(struct spi_device *spi,
                        const void *txbuf, unsigned n_tx,
                        void *rxbuf, unsigned n_rx)
{
    static DEFINE_MUTEX(lock);

    int			status;
    struct spi_message	message;
    struct spi_transfer	x[2];
    u8			*local_buf;

    /* Use preallocated DMA-safe buffer if we can.  We can't avoid
	 * copying here, (as a pure convenience thing), but we can
	 * keep heap costs out of the hot path unless someone else is
	 * using the pre-allocated buffer or the transfer is too large.
	 */
    if ((n_tx + n_rx) > SPI_BUFSIZ || !mutex_trylock(&lock)) {
        local_buf = kmalloc(max((unsigned)SPI_BUFSIZ, n_tx + n_rx),
                            GFP_KERNEL | GFP_DMA);
        if (!local_buf)
            return -ENOMEM;
    } else {
        local_buf = buf;
    }

    spi_message_init(&message);
    memset(x, 0, sizeof(x));
    if (n_tx) {
        x[0].len = n_tx;
        spi_message_add_tail(&x[0], &message);
    }
    if (n_rx) {
        x[1].len = n_rx;
        spi_message_add_tail(&x[1], &message);
    }

    memcpy(local_buf, txbuf, n_tx);
    x[0].tx_buf = local_buf;
    x[1].rx_buf = local_buf + n_tx;

    /* do the i/o */
    status = spi_sync(spi, &message);
    if (status == 0)
        memcpy(rxbuf, x[1].rx_buf, n_rx);

    if (x[0].tx_buf == buf)
        mutex_unlock(&lock);
    else
        kfree(local_buf);

    return status;
}
EXPORT_SYMBOL_GPL(spi_write_then_read);
```

### 核心层源码

SPI 同步传输数据最终调用`__spi_sync`发送。

```c
static int __spi_sync(struct spi_device *spi, struct spi_message *message)
{
	 
	if (ctlr->transfer == spi_queued_transfer) {

		status = __spi_queued_transfer(spi, message, false);	//添加到传输队列

	}
	if (status == 0) {

		if (ctlr->transfer == spi_queued_transfer) {

			__spi_pump_messages(ctlr, false);	//开始传输传输队列的数据
		}

		wait_for_completion(&done);	//等待结果

	}
	message->context = NULL;
	return status;
}
```

遍历 message 发送所用 transfer。

![](https://cdn.nlark.com/yuque/0/2024/png/21562848/1729663851536-de722f5b-9c7d-401c-9628-dc0916a029c4.png)

<font style="color:rgb(62, 67, 73);">SPI驱动器有两种类型，这里称为：</font>

+ **<font style="color:rgb(62, 67, 73);">Controller drivers</font>**<font style="color:rgb(62, 67, 73);"> 控制器驱动程序 :控制器可以内置于片上系统处理器中，并且通常同时支持控制器和目标角色。这些驱动程序与硬件寄存器接触，并可能使用 DMA。或者它们可以是 PIO bitbangers，只需要 GPIO 引脚。</font>
+ **<font style="color:rgb(62, 67, 73);">Protocol drivers</font>**<font style="color:rgb(62, 67, 73);"> 协议驱动程序 :它们通过控制器驱动程序传递消息，以便与 SPI 链路另一端的目标或控制器设备进行通信</font>

![画板](C:\Users\xiaodq\Desktop\新建文件夹\pic\2024-10-23-spi-driver\1722511536666-e221be88-27c6-4b4c-b80b-be8261a7894b.jpeg)

# spi控制器驱动

## spi_master 主机描述结构体

`#define spi_master spi_controller`

```c
struct spi_master {  
    struct device   dev;  
    s16      bus_num;  /*bus_num为该控制器对应的SPI总线号*/
    u16      num_chipselect; /*num_chipselect 控制器支持的片选数量，即能支持多少个spi设备*/
  /*setup函数是设置SPI总线的模式，时钟等的初始化函数， 针对设备设置SPI的工作时钟及数据传输模式等,在spi_add_device函数中调用*/
    int      (*setup)(struct spi_device *spi);  
    /*实现SPI总线读写方法的函数。实现数据的双向传输，可能会睡眠*/
    int      (*transfer)(struct spi_device *spi, struct spi_message *mesg); 
    void     (*cleanup)(struct spi_device *spi);  /*注销时候调用*/
    struct list_head	queue;
};
```

```c
spi0: spi@1c05000 {
    compatible = "allwinner,suniv-spi",
    "allwinner,sun8i-h3-spi";
    reg = <0x01c05000 0x1000>;
    interrupts = <10>;
    clocks = <&ccu CLK_BUS_SPI0>, <&ccu CLK_BUS_SPI0>;
    clock-names = "ahb", "mod";
    resets = <&ccu RST_BUS_SPI0>;
    status = "disabled";
    #address-cells = <1>;		#定义设备中reg使用几位
    #size-cells = <0>;

};
&pio {
    spi0_cs_pins: spi0_cs_pins {
        pins = "PC3", "PH6";
        function = "gpio_out";
    };
};
&spi0 {
    pinctrl-names = "default";
    pinctrl-0 = <&spi0_pins_a &spi0_cs_pins>;	#使用pinctrl初始化硬件
        cs-gpios = <0> <&pio 2 3 GPIO_ACTIVE_HIGH>, <&pio 7 6 GPIO_ACTIVE_HIGH>;	#cs使用的硬件
            status = "okay";
    spi-max-frequency = <50000000>;
    flash: xt25f128b@0 {
        #address-cells = <1>;
        #size-cells = <1>;
        compatible = "winbond,xt25f128b", "jedec,spi-nor";
        reg = <0>;
        spi-max-frequency = <50000000>;
    };
};
```

`platform driver` 匹配成功时调用 `probe`，将控制器 spi 构造出 `spi_master` 结构体，将设备构造出 `spi_device`。

**gpio 模拟 SPI 控制器：**

![](C:\Users\xiaodq\Desktop\新建文件夹\pic\2024-10-23-spi-driver\1718884108019-cf7aa701-be34-4af6-a7dc-0a9d9898beb7.png)

## SPI 控制器层设备匹配
SPI 设备树节点参数文档：**Linux\Documentation\devicetree\bindings\spi\spi-bus.txt**

**spi 控制器**和 **spi 控制器信息**或**设备树**的匹配方式是 platform 匹配，**probe 主要工作：**

1. 构造 `spi_master`
2. 解析设备树属性，构造`spi_device`
3. 初始化 SPI 硬件

### 构造 `spi_master`
```c
static int sun6i_spi_probe(struct platform_device *pdev)
{

    ...
	ret = devm_spi_register_master(&pdev->dev, master);		//解析设备树，注册控制器和设备
	if (ret) {
		dev_err(&pdev->dev, "cannot register SPI master\n");
		goto err_pm_disable;
	}

	return 0;

err_pm_disable:
	pm_runtime_disable(&pdev->dev);
	sun6i_spi_runtime_suspend(&pdev->dev);
err_free_master:
	spi_master_put(master);
	return ret;
}

```

```c
static int of_spi_register_master(struct spi_controller *ctlr)
{
	int nb, i, *cs;
	struct device_node *np = ctlr->dev.of_node;

	if (!np)
		return 0;

	nb = of_gpio_named_count(np, "cs-gpios");	//解析设备树属性
	ctlr->num_chipselect = max_t(int, nb, ctlr->num_chipselect);

	/* Return error only for an incorrectly formed cs-gpios property */
	if (nb == 0 || nb == -ENOENT)
		return 0;
	else if (nb < 0)
		return nb;

	cs = devm_kzalloc(&ctlr->dev, sizeof(int) * ctlr->num_chipselect,
			  GFP_KERNEL);
	ctlr->cs_gpios = cs;

	if (!ctlr->cs_gpios)
		return -ENOMEM;

	for (i = 0; i < ctlr->num_chipselect; i++)
		cs[i] = -ENOENT;

	for (i = 0; i < nb; i++)
		cs[i] = of_get_named_gpio(np, "cs-gpios", i);

	return 0;
}
```

### 解析设备树属性，构造`spi_device`
```c
/**
 * of_register_spi_devices() - Register child devices onto the SPI bus
 * @ctlr:	Pointer to spi_controller device
 *
 * Registers an spi_device for each child node of controller node which
 * represents a valid SPI slave.
 */
static void of_register_spi_devices(struct spi_controller *ctlr)
{
	struct spi_device *spi;
	struct device_node *nc;

	if (!ctlr->dev.of_node)
		return;

	for_each_available_child_of_node(ctlr->dev.of_node, nc) {
		if (of_node_test_and_set_flag(nc, OF_POPULATED))
			continue;
		spi = of_register_spi_device(ctlr, nc);
		if (IS_ERR(spi)) {
			dev_warn(&ctlr->dev,
				 "Failed to create SPI device for %pOF\n", nc);
			of_node_clear_flag(nc, OF_POPULATED);
		}
	}
}
//解析设备节点设备树
static struct spi_device *
of_register_spi_device(struct spi_controller *ctlr, struct device_node *nc)
{
	struct spi_device *spi;
	int rc;

	/* Alloc an spi_device */
	spi = spi_alloc_device(ctlr);
	if (!spi) {
		dev_err(&ctlr->dev, "spi_device alloc error for %pOF\n", nc);
		rc = -ENOMEM;
		goto err_out;
	}

	/* Select device driver */
	rc = of_modalias_node(nc, spi->modalias,
				sizeof(spi->modalias));
	if (rc < 0) {
		dev_err(&ctlr->dev, "cannot find modalias for %pOF\n", nc);
		goto err_out;
	}

	rc = of_spi_parse_dt(ctlr, spi, nc);	//解析设备树
	if (rc)
		goto err_out;

	/* Store a pointer to the node in the device structure */
	of_node_get(nc);
	spi->dev.of_node = nc;

	/* Register the new device */
	rc = spi_add_device(spi);	//注册新设备
	if (rc) {
		dev_err(&ctlr->dev, "spi_device register error %pOF\n", nc);
		goto err_of_node_put;
	}

	return spi;

err_of_node_put:
	of_node_put(nc);
err_out:
	spi_dev_put(spi);
	return ERR_PTR(rc);
}

```

## 控制器驱动框架：
### 注册 master 流程
参考 [https://docs.kernel.org/spi/spi-summary.html#how-do-i-write-an-spi-controller-driver](https://docs.kernel.org/spi/spi-summary.html#how-do-i-write-an-spi-controller-driver)

1. <font style="color:rgb(62, 67, 73);">使用 </font>`<font style="color:rgb(62, 67, 73);">spi_alloc_host（）</font>`<font style="color:rgb(62, 67, 73);"> 分配主机控制器，使用 </font>`<font style="color:rgb(62, 67, 73);">spi_controller_get_devdata（）</font>`<font style="color:rgb(62, 67, 73);"> 获取为该设备分配的驱动程序专用数据。</font>

```c
struct spi_controller   *ctlr;
struct CONTROLLER       *c;

ctlr = spi_alloc_host(dev, sizeof *c);
if (!ctlr)
        return -ENODEV;

c = spi_controller_get_devdata(ctlr);
```

2. <font style="color:rgb(62, 67, 73);">使用 </font>`spi_register_controller（）<font style="color:rgb(62, 67, 73);"> </font>`<font style="color:rgb(62, 67, 73);">将其发布到系统的其余部分。届时，控制器和任何预声明的 spi 设备的设备节点将可用。</font>
3. <font style="color:rgb(62, 67, 73);">如果需要删除 SPI 控制器驱动程序，</font>`spi_unregister_controller（）`<font style="color:rgb(62, 67, 73);">。</font>

### SPI 硬件控制器初始化
#### SPI 中断初始化
![](C:\Users\xiaodq\Desktop\新建文件夹\pic\2024-10-23-spi-driver\1729324742081-9fb55809-e251-45cb-9431-093d8963f111.png)

![](C:\Users\xiaodq\Desktop\新建文件夹\pic\2024-10-23-spi-driver\1729324674609-60c253e2-8dcc-40d2-9e38-ccde48c5925f.png)

#### 填写必要硬件信息
![](C:\Users\xiaodq\Desktop\新建文件夹\pic\2024-10-23-spi-driver\1729325887224-712aa6f7-34f4-4189-a9d1-5ddad4e2e058.png)

## SPI 传输框架
### 控制器驱动提供接口
<font style="color:rgb(34, 40, 50);">Linux中使用spi_master结构体描述SPI控制器，有两套传输方法：</font>

![](C:\Users\xiaodq\Desktop\新建文件夹\pic\2024-10-23-spi-driver\1729661401350-9cf711a8-d12a-4e1a-a530-172a42650b5a.png)

####  `__spi_sync`
![](pic/84_spi_transfer_old.png)![旧控制器驱动架构](C:\Users\xiaodq\Desktop\新建文件夹\pic\2024-10-23-spi-driver\1724246410734-34161012-3a2e-499c-8a4b-10f326a913c4.png)![新控制器驱动架构](C:\Users\xiaodq\Desktop\新建文件夹\pic\2024-10-23-spi-driver\1724246417195-2a1e76c4-f7f0-4ff9-959c-3250d358bbfe.png)

## 控制器驱动解读：
### 设置时钟频率
```c
static void cdns_spi_config_clock_freq(struct spi_device *spi,
                                        struct spi_transfer *transfer)
{
    struct cdns_spi *xspi = spi_master_get_devdata(spi->master);
    u32 ctrl_reg, baud_rate_val;
    unsigned long frequency;

    frequency = clk_get_rate(xspi->ref_clk);	//获取SPI参考时钟频率

    ctrl_reg = cdns_spi_read(xspi, CDNS_SPI_CR);

    /* Set the clock frequency */
    if (xspi->speed_hz != transfer->speed_hz) {
        /* first valid value is 1 */
        baud_rate_val = CDNS_SPI_BAUD_DIV_MIN;
        //匹配合适分频系数
        while ((baud_rate_val < CDNS_SPI_BAUD_DIV_MAX) &&
                       (frequency / (2 << baud_rate_val)) > transfer->speed_hz)
            baud_rate_val++;

        ctrl_reg &= ~CDNS_SPI_CR_BAUD_DIV;
        ctrl_reg |= baud_rate_val << CDNS_SPI_BAUD_DIV_SHIFT;

        xspi->speed_hz = frequency / (2 << baud_rate_val);	//记录当前速度
    }
    cdns_spi_write(xspi, CDNS_SPI_CR, ctrl_reg);
}
```

由此代码可知，SPI 实际速度只可能小于、等于设备树配置中设置的 SPI 速度最大值，并且**只能是分频系数能分频出的几个值。**

## 底层操作传递给上层控制器驱动的方式
参考 imx6ull 控制器驱动：`drivers/spi/spi-imx.c`

在 底层实现了多种设备的驱动代码，这些驱动代码在控制器驱动的实现流程上是一样的，但是对于寄存器的操作却有所不同。

![](C:\Users\xiaodq\Desktop\新建文件夹\pic\2024-10-23-spi-driver\1724598769068-f34d3783-444a-4755-9130-7993160502f7.png)

![](C:\Users\xiaodq\Desktop\新建文件夹\pic\2024-10-23-spi-driver\1724598788883-c969eab1-4335-45df-aed2-cb066806c50e.png)

为了减少使用重复的代码，imx 在实行上将控制器的底软和控制器核心进行了分离：

![画板](C:\Users\xiaodq\Desktop\新建文件夹\pic\2024-10-23-spi-driver\1724599163402-0da3a524-d890-4e4e-b92a-0fb961051a78.jpeg)

控制器核心并不识别使用什么驱动，而是调用相同的接口，这些接口在 platform 中通过私有数据的方式提供到 probe，然后在 probe 中提供给 spi master 控制器。

![画板](C:\Users\xiaodq\Desktop\新建文件夹\pic\2024-10-23-spi-driver\1724599759502-961d0787-3294-4873-a97d-314664d94f9d.jpeg)

## completion
`completion` 是内核提供的 `transfer masage` 管理接口

```plain
sun4i_spi_probe
  init_completion(&sspi->done);
```

## bitbang 管理接口

# spidev

+ **使用文档：**`** Documentation/spi/spidev**`
+ [**https://docs.kernel.org/spi/spidev.html**](https://docs.kernel.org/spi/spidev.html)

## 注册 spidev
```c
spidev0: spidev@0 {
    compatible = “spidev”;	//设备树要存在
    reg = <0>;
    spi-max-frequency = <50000000>;
};
```

## spidev 接口
![](C:\Users\xiaodq\Desktop\新建文件夹\pic\2024-10-23-spi-driver\1724162876469-7370781f-c674-4287-b56b-a3990e8288aa.png)

## 使用 spidev
<font style="color:rgb(51, 51, 51);">内核提供的测试程序：</font>`<font style="color:rgb(51, 51, 51);background-color:rgb(243, 244, 244);">tools\spi\spidev_fdx.c</font>`

## <font style="color:rgb(51, 51, 51);">使用方法</font>
`<font style="color:rgb(51, 51, 51);background-color:rgb(243, 244, 244);">spidev_fdx [-h] [-m N] [-r N] /dev/spidevB.D</font>`

+ <font style="color:rgb(51, 51, 51);">-h: 打印用法</font>
+ <font style="color:rgb(51, 51, 51);">-m N：先写1个字节0xaa，再读N个字节，</font>**<font style="color:rgb(51, 51, 51);">注意：</font>**<font style="color:rgb(51, 51, 51);">不是同时写同时读</font>
+ <font style="color:rgb(51, 51, 51);">-r N：读N个字节</font>

## 源码解析：
![画板](C:\Users\xiaodq\Desktop\新建文件夹\pic\2024-10-23-spi-driver\1723205148065-dd02daa9-0db4-4d97-ab4e-c33b19027c77.jpeg)

### 多个 dev 管理方式：
spidev 每匹配成功一个，便会把 spi 设备信息保存到 spi list 中

![](C:\Users\xiaodq\Desktop\新建文件夹\pic\2024-10-23-spi-driver\1724162777192-1ef5fb07-66d4-4e61-88ca-3292fa09e4ce.png)

### 内核空间和用户空间数据交互
![](C:\Users\xiaodq\Desktop\新建文件夹\pic\2024-10-23-spi-driver\1724163152910-a142d5db-f696-4e58-9dfd-af4176cd9a8e.png)

![](C:\Users\xiaodq\Desktop\新建文件夹\pic\2024-10-23-spi-driver\1724163167661-1f7dcfe3-3a32-4168-b3db-11fea8b466b6.png)