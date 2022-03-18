# ZHANGSHUO_CAR_V1 此工程用来记录无人小车的驱动部分(STM32开发)

## 所使用单片机和环境说明：
### 所使用的单片机：

本工程所使用的单片机为以STM32F103VET6：
![image](https://user-images.githubusercontent.com/78637328/158755345-769e37a6-3798-4121-8149-6b9bae4f1427.png)  

有的童鞋可能对这个命名方式很感兴趣，在这里可以了解一下：
![image](https://user-images.githubusercontent.com/78637328/158755571-ab8bb1a1-5d50-4a9f-93e2-e5c349a1bbf3.png)

本工程使用的下位机开发板如图所示
![0f086450e21dac6539681fe26112849](https://user-images.githubusercontent.com/78637328/158755885-da75319c-c507-4f0f-bcb4-ea2067fba4da.jpg)
### 开发环境的搭建：
- *安装JDK*:执行 jdk-8u181-windows-x64.exe,一路下一步,没有下一步就点击finish
- *STM32芯片引脚配置工具*：解压en.stm32cubemx_v5.4.0.zip并执行SetupSTM32CubeMX-5.4.0.exe，点击下一步下一步就可以完成安装
- *交叉编译工具*：解压arm的交叉编译工具：解压gcc-arm-none-eabi-9-2019-q4-major-win32.zip 将它的根目录和bin路径配置到系统的Path中
- *烧录工具*：解压FlyMcu.zip 可以直接使用.

上面的安装工作完成之后，我们就可以进入STM32开发的工作中啦！

## 项目创建：

可以使用STM32CubeMX创建工程
### 选择芯片：
前面也说到了，该工程使用的单片机为STM32F103VET6,在搜索栏搜STM32F103VETx双击创建
![image](https://user-images.githubusercontent.com/78637328/158757389-b372b9af-981d-425b-a97d-1700adf5863c.png)  
### 打开串口调试：
![image](https://user-images.githubusercontent.com/78637328/158758058-c99f8cef-57a3-4467-92c8-5d1beab586ea.png)
### 配置时钟：

选择高速时钟

![image](https://user-images.githubusercontent.com/78637328/158758132-e34e1daa-cf39-4f49-82c8-b990cbd3dd87.png)

72Mhz指的是1秒钟执行72 000 000次

![image](https://user-images.githubusercontent.com/78637328/158758233-80e4dfd0-0de0-4954-84c8-b76ff12eaf59.png)
### 工程配置：
<img width="1393" alt="屏幕截图 2022-03-17 152934" src="https://user-images.githubusercontent.com/78637328/158758608-4eb7ea83-640a-4c8b-a59d-487fbfe8cb09.png">

![image](https://user-images.githubusercontent.com/78637328/158758810-6ecc8fcf-648f-438a-bd98-83b43c28b8af.png)

### 生成代码
![image](https://user-images.githubusercontent.com/78637328/158758883-90e2a0d4-9083-47c4-bcd1-7ba2f705da58.png)
到此为止一个最基本的工程创建就完成了！
### 敲项目
用代码编辑工具（vs,clion,...都可）就可以愉快的玩耍了！
### 烧录程序：
<img width="536" alt="image" src="https://user-images.githubusercontent.com/78637328/158759303-c6c2e840-14ac-4219-8119-53c9bfd16314.png">

- port:选择连接STM32的串口，可在电脑设备管理器中查询
- bps:可以调最高，加快烧写速度
- 联机下载的程序文件：stm32工程生成的hex文件
- 最下面选择：DTR的电平复位，RTS高电平进BootLoader

以上就是开发下位机的一个大概流程，后面的各个模块驱动由下面给出：


功能编写见[help](https://github.com/Zhangshuor/ZHANGSHUO_CAR_V1/blob/master/HELP.pdfhttps://github.com/Zhangshuor/ZHANGSHUO_CAR_V1/blob/master/HELP.pdf)文档！！！持续更新














