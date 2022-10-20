# 特性列表
## BUS相关功能描述
CXL内存盘端到端保护：Host下发送数据携带DIF信息，对接收数据完成DIF校验；  

前端->后端：写flash：  

前端写DDR时的错误需以4KB粒度发送到后端；  

读flash：后端写DDR时的错误需以4KB粒度发送到前端。  

系统bus地址按照40bit设计

## 安全相关功能描述
FW启动时，只有安全版本号大于等于EFUSE中的安全版本号时才能启动；  

安全.FS.011.002:芯片提供SHA256的数据计算带宽不小于250MBps

## Cortex-M7 相关功能描述
FS.SOC_M7.001:支持M7地址重映射功能：将一段DDR空间划分为多段后，映射给不同M7的同一地址空间。  

## DDR相关功能描述
DDR支持以下几种自刷新进入方式：a：软件配置进入，这种需要通过系统保证其他模块不访问MDDRC；b：自动进入，当较长时间没有业务访问就会进入；c：看门狗复位自动进入。  

空闲进入powerdown的时间：0x4个时钟周期，powerdown协议要求不能关时钟

## BOOTROM相关功能描述
 支持BOOTROM的容量：80KB  
 支持BOOTROM的ECC校验：1bit纠错，2bit检错  
 支持BOOTROM的中断：高电平有效中断  
 
 ## SPIFlash 相关功能描述
 支持对接单双线SPI NORFLASH颗粒  
 不支持ECC 1bit纠错和2bit检错  
 支持SPI NORFLASH接口带宽40Mbit/s  
 支持单片SPI NORFLASH最大容量16MByte

## UART相关功能描述
支持UART接口接收FIFO深度256BByte，发送FIFO深度256Byte

# 功能描述
## SOC_APP_CORE
### 规格描述
Hi1813EV100计算和控制子系统由位于同一个Cluster内部的4个ARM Cortex-A55处理器构成。其中1个Cortex-A55承载OM和FE功能，另外3个Cortex-A55承载BE功能

## SOC_BUS
### 规格描述
整系统总线分为2份：APP_QIC，NFI_QIC。2个总线之间通过QSP接口互联。其中NFI_QIC可以访问APP_QIC内部的外设寄存器，DME及DDR空间；APP_QIC可以访问NFI_QIC内部的所有寄存器及M7 CORE的TCM空间。APP_QIC/NFI_QIC访问各自内部的寄存器及存储空间在各自总线内部完成，不会通过QSP总线转到对端总线后再环回。

#### NFI_QIC总线
NFI_QIC为SOC_NFI_CORE内部的总线，主要完成SOC_NFI_CORE内部的总线互联访问，同时将内部CPU及其他master对DME及模块寄存器的访问通过QSP接口转到APP_QIC;将CPU及其他master对DDR的访问通过QSP接口转到DMI_QIC

#### APP_QIC总线
APP_QIC为系统中的主总线，承担着系统主要的访问通路路由及分发工作


## IP特性
### UART
UART是一个异步串行的通信接口，主要执行串并转换功能　　

支持中断方式和软件查询方式进行数据搬运　　　　

支持8bit宽、深度为256的发送FIFO和10bit宽、深度为256的接收FIFO  

UART内部有2个时钟：pclk和uart_clk  
pclk：APB接口时钟，配置寄存器时钟  
uart_clk：模块内部逻辑工作时钟  
两者为同频同步时钟  

UART内部有2个复位信号：presset_n和uart_preset_n  
preset_n：APB接口复位信号，用于pclk时钟域的复位，低电平有效，异步复位，同步解复位  
uart_preset_n：模块内部逻辑复位信号，用于uart_clk时钟域的复位，低电平有效，异步复位，同步解复位  

### GPIO
GPIO为通用可编程输入输出外设  
支持1组32bit GPIO输入输出信号，支持GPIO个数：21个，支持高电平有效中断  

滤除毛刺：
如果毛刺信号跨越了两个debounce_clk的上升沿，则毛刺信号不会被滤除，如果没有跨越两个debounce_clk的上升沿，则毛刺信号被滤除。  

GPIO有2个内部时钟：  
pclk：APB接口时钟，GPIO的工作时钟　　
gpio_db_clk：对输入信号做Debounce（防抖）功能时，需要CRG提供一个单独的时钟，其频率决定了滤除毛刺的长度，可以与pclk不同步，但频率比pclk低  

GPIO内部有2个复位信号：
preset_n：APB接口复位信号，除Debounce功能时所有逻辑的复位信号，异步复位，同步解复位，低电平有效  
gpio_db_rst_n：debouncce功能复位信号，异步复位，同步解复位，低电平有效  

初始化： 上电复位时，所有管脚默认为输入状态  

### I2C
采用串行时钟线SCL和串行数据线SDA进行数据传输，每次传输都以一个字节为单位。SCL和SDA都为双向总线，通过漏极开路实现线与逻辑  

同步串行全双工

支持I2C接口的接收FIFO/发送FIFO深度：分别为64Byte  

接口的中断：高电平有效中断  

内部涉及2个时钟：pclk和i2c_clk,同步同频200Mhz  

preset_n为pcclk作用寄存器的复位信号，低电平有效，i2x_rst_n为i2c_clk作用寄存器的复位信号，低电平有效。复位均为异步复位，同步解复位。复位撤离时同时撤离


### SPI
SPI总线系统时一种同步串行外设接口，它可以使MCU与各种外围设备以串行方式进行通信以交换信息  

数据按位传输，高位在前，低位在后， 全双工通信  

收/发为分开的32bit宽、深度为16的FIFO　　

内部涉及２个时钟：　　
pclk:APB接口时钟，配置寄存器时钟　　
spi_clk:SPI模块内部逻辑工作时钟  
两者为同步同频时钟

2个复位信号:
preset_n:APB接口复位信号,用于pclk作用寄存器的复位，低电平有效，异步复位，同步解复位  
spi_rst_n：SPI模块内部逻辑复位信号，用于spi_clk作用寄存器的复位，低电平有效。异步复位，同步解复位  
复位撤离时同时撤离  

### Timer
Hi1813EV100芯片支持48个Timer 模块  
每个Timer模块支持64bit计数  
支持Timer中断上报，高电平有效中断  
功能原理：64 bits Timer基于一个64 bits减法计数器。计数器的值在每个计数时钟的上升沿减1  
初始化：设置计数初值；设置计数周期；配置计数模式、计数器长度、预分频系数及中断使能

### FMC
FMC（Flash Memory Controller），实现对片外SPI NOR Flash数据存取控制  
提供两个AHB Slave接口，一个寄存器接口，一个mem读写接口      
每个片选的存储空间最大支持到16MB  
FMC的片选信号均为低电平有效  

### IPC
IPC（Inter-Processor Communication）模块用于APP_CORE和NFI_CORE相互通信传递数据  
每个IPC可以提供最多32个通道进行消息传送  

### JTAG
JTAG是芯片和关键子系统调试的主要控制接口，外部仿真器或者调试工具通过JTAG接口对芯片调试  

### SYSCNT
系统中提供两个64bit位宽的全局counter：SYSCNT0和SYSCNT1  ，频率为固定的25M
SYSCNT0采用晶振时钟，不受看门狗复位，仅在上电复位期间被复位为0  
SYSCNT0支持所有的CPU读取访问（ 4 APP_CORE + 8 NFI_CORE）  
SYSCNT1采用晶振时钟，不受看门狗复位，仅在上电复位期间被复位为0  
SYSCNT1支持所有的CPU读取访问（ 4 APP_CORE + 8 NFI_CORE）  
SYSCNT0支持软件同步配置系统时间，SYSCNT1不支持软件同步配置系统时间  

### WDOG
WDOG（系统看门狗）用于系统异常情况下，一定时间内发出异常指示信号或者复位信号，以复位整个系统  
系统中支持12个WDOG, 其中包含1个全局WDOG（又称硬狗）和11个普通WDOG（又称软狗）  
软狗初始化：步骤 1	配置WDOG使能寄存器WDG_CONTROL[inten]为0，关闭看门狗；步骤 2	配置WDOG初始比较阈值寄存器WDG_LOAD；步骤 3	配置WDOG使能寄存器WDG_CONTROL[inten]为1，打开看门狗  


### DMA
DMA：直接内存访问方式， 是一种完全由硬件执行数据搬移的工作方式，通过一个独立于处理器的总线主控制器执行数据搬移操作  
不需要经过CPU，数据被直接在存储器之间，存储器与外设之间进行搬移  
一般以中断方式向CPU报告传送操作的完成

# DEV
DEV主要是通过OS的timer、retry队列和状态机实现子系统的各个设备的监控和管理  

DEV的模块的软件实现的主要目标：  
将功能实现和业务逻辑功能进行剥离参数化  
硬件互斥访问。由于底层部分硬件共用，需要处理赢家资源互斥访问。  
功能实现灵活易扩展。由于硬件ip的差异性，需要功能实现灵活易扩展  

针对这些目标，主要通过OS Timer、retry Queue、状态机来实现这三个目标  
通过业务数据和业务逻辑剥离和参数化来实现、巡检任务并行化通过OS timer实现  
硬件互斥访问，通用单个OS retry Queue来实现业务功能呢访问的串行化，多OS retry Queue队列来实现业务功能的并行化访问 
功能实现灵活易扩展OS状态机来实现  

## SOC_APP_CORE
Hi1813EV100计算和控制子系统由位于同一个Cluster内部的4个ARM Cortex-A55处理器构成。其中1个Cortex-A55承载OM和FE功能，另外3个Cortex-A55承载BE功能

## SOC_NFI_CORE
支持240个中断  
支持每个NFI_CORE 可配置4个可编程64 bits Timer  
每个Timer支持当计数值减到0的当拍会产生高电平中断  
支持8个NFI_CORE共享1个UART接口  
