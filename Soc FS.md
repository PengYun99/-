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
