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

## 功能编写

### LED小灯的控制
![image](https://user-images.githubusercontent.com/78637328/158760229-9b36bd77-cf6a-47f6-b6a1-844168ac040b.png)

根据原理图可以知道LED小灯由PE10引脚控制，在STM32CubeMx中将PE10引脚设置为输出

![image](https://user-images.githubusercontent.com/78637328/158760417-bef647de-0922-4d53-aae3-26f9a625636c.png)

按照电路图,LED的一端已经连接了高电平, 那么PE10这一端我们需要给它低电平,才能让灯亮起来

`HAL_GPIO_WritePin(GPIOA,GPIO_PIN_1,GPIO_PIN_RESET);`

如果想让灯灭的话, 我们需要给PE10这一端设定为高电平

`HAL_GPIO_WritePin(GPIOA,GPIO_PIN_1,GPIO_PIN_SET);`

如果想让LED间隔一定时间闪烁的话,我们可以采用如下的方式:

```c
// 闪烁led灯 
HAL_GPIO_WritePin(GPIOA,GPIO_PIN_1,GPIO_PIN_SET); 
HAL_Delay(500); 
HAL_GPIO_WritePin(GPIOA,GPIO_PIN_1,GPIO_PIN_RESET); 
HAL_Delay(500);
```
这段程序可以在main函数中查看到
### 蜂鸣器的控制

![image](https://user-images.githubusercontent.com/78637328/158761139-415a5910-2935-43b3-8eef-82c723824b0b.png)

蜂鸣器的控制和LED一致
```c
        /****************************响_蜂鸣器**********************************/
        HAL_GPIO_WritePin(GPIOB,GPIO_PIN_10,GPIO_PIN_SET);
        //延迟两秒钟
        HAL_Delay(1000);
        /*停_蜂鸣器*/
        HAL_GPIO_WritePin(GPIOB,GPIO_PIN_10,GPIO_PIN_RESET);
        //延迟两秒钟
        HAL_Delay(1000);
```
上面的小灯和蜂鸣器的实验都是拿引脚作为输出来使用，接下来来实现一下引脚作为输入的功能：
### 开关的使用
我们使用GPIO来读取开关状态，其实就是要读取IO口的电平变化

1.查看原理图：

![image](https://user-images.githubusercontent.com/78637328/158765025-cc903ce2-955d-4158-8ceb-8cfea45ebcfb.png)

2.配置STM32的IO口：

我们可以看到，我们将要使用8号引脚作为输入

![image](https://user-images.githubusercontent.com/78637328/158765146-d504ad69-6f58-430e-8bdc-2ef7f0002fbe.png)

3.在main中编写逻辑：

```c
        /***************************开关按钮*********************************************/
        //读取开关按钮状态
        GPIO_PinState state = HAL_GPIO_ReadPin(GPIOD, GPIO_PIN_8);

        if (state==GPIO_PIN_RESET){
            // 如果低电平，亮LED；
            HAL_GPIO_WritePin(GPIOE, GPIO_PIN_10, GPIO_PIN_RESET);
        } else if (state==GPIO_PIN_SET){
            // 如果高电平，灭LED；
            HAL_GPIO_WritePin(GPIOE, GPIO_PIN_10, GPIO_PIN_SET);
```

使得按下开关按钮，==灯灭==；松开开关按钮，==灯亮==

### 知识补充：通讯的基本概念

在计算机设备与设备之间或集成电路之间常常需要进行数据传输

#### 串行通讯与并行通讯

按数据传送的方式，通讯可以分为：

- 串行通讯：串行通讯是指设备之间通过少量数据信号线（一般8根以下），地线以及控制信号线，按数据位形式一位一位地传输数据的通讯方式。

- 并行通讯：而并行通讯一般是指使用 8、16、32及 64 根或更多的数据线进行传输的通讯方式。

  ![1647517382061](C:\Users\29976\AppData\Roaming\Typora\typora-user-images\1647517382061.png)
  
  
  
  串行通讯和并行通讯的特性对比

|   特性    |   串行通讯  |  并行通讯  |
| :-------: | :---: | :---------: |
| 通讯距离 | 较远 |较近 |
| 抗干扰能力 | 较强  |  较弱   |
| 传输速度 | 较慢 | 较快 |
| 成本 | 较低 | 较高 |

#### 全双工、半双工及单工通讯

根据数据通讯的方向，通讯又分为全双工、半双工及单工通讯，它们主要 以信道的方向来区分
|   通讯方式    |   说明  |
| :-------: | :---: |
| 全双工 | 同一时刻，两个设备之间可以同时收发数据 |
| 半双工 | 两个设备之间可以收发数据，但是不能在同一时刻进行  |
| 单工 | 在任何时刻都只能进行一个方向的通讯，即以恶搞固定为发送设备，另一个固定为接收设备（遥控器这种） |

![1647518331694](C:\Users\29976\AppData\Roaming\Typora\typora-user-images\1647518331694.png)

#### 同步通信和异步通信

根据通讯的数据同步方式，又分为同步和异步两种，可以根据通讯过程中是否有使用到时钟信号进行简单的区分。 

简单的说就是主机在相互通信时发送数据的频率是否一样。



![1647518402595](C:\Users\29976\AppData\Roaming\Typora\typora-user-images\1647518402595.png)

![1647518420558](C:\Users\29976\AppData\Roaming\Typora\typora-user-images\1647518420558.png)

#### 通讯速率

衡量通讯性能的一个非常重要的参数就是通讯速率，通常以比特率(Bitrate) 来表示，==即每秒钟传输的二进制位数==， 单位为比特每秒(bit/s)。 

举个例子：

例如比特率为115200，也就是一秒钟可以传递115200bit。

1byte = 8 bit

$115200bit = \frac{115200}{8}byte=14400byte$

1kb = 1024byte

$115200bit=115200/8/1024kb=14.0625kb$ 

容易与比特率混淆的概念是“波特率”(Baudrate)，它表示每秒钟传输了多少个码元。

啥是码元？

信息单位：

- 是1、否0 可以用1bit表示（==如果码元只有两种情况的时候，也就是1个字节的时候，他和比特率是一样的概念==）
- 比如：4，5，6，7 需要用00，01，10，11表示

而码元是通讯信号调制的概念，通讯中常用时间间隔相同的符号来表示一个二进制数字，这样的信号称为码元。如常见的通讯传输中，用 0V 表示数字 0，5V 表示数字 1，那么一个码元可以表示两种状态 0 和 1，所以一个码元等于一个二进制比特位，此时波特率的大小与比特率一致；如果在通讯传输中，有 0V、2V、4V 以及 6V分别表示二进制数 00、01、10、11，那么每个码元可以表示四种状态，即两个二进制比特位，所以码元数是二进制比特位数的一半，这个时候的波特率为比特率的一半。因为很多常见的通讯中一个码元都是表示两种状态，人们常常直接以波特率来表示比特率。 

常见的波特率为4800、9600、115200 等。

### USART串口通信

#### 串口通信

串口通讯(Serial Communication)是一种设备间非常常用的串行通讯方式，因为它简单便捷，因此大部分电子设备 都支持该通讯方式，电子工程师在调试设备时也经常使用该通讯方式输出调试信息。对于通讯协议，可以分为物理层和协议层。 

- **物理层**规定通讯系统中具有机械、电子功能部分的特性，确保原始数据在物理媒体的传输。就是硬件部分 
  * 物理层规定我们用嘴巴还是用肢体来交流

- **协议层**主要规定通讯逻辑，统一收发双方的数据打包、解包标准。就是软件部分 
  * 协议层则规定我们用中文还是英文来交流。

**物理层** 

物理层常见的标准RS232（全双工）、RS485（半双工）、USB转串口(TTL)以及原生的串口到串口(TTL-TTL) 

**协议层** 

**数据包组成**

![1647519874305](C:\Users\29976\AppData\Roaming\Typora\typora-user-images\1647519874305.png)

举个例子（随便写的，只是示意），比如传两个数据：

hsdia shidoaidh这两个数据没有标识位我们不知道一个数据结束没有，下一个数据开始没有。

wwhsdiarr: 看见ww是开始，看见rr结束。:cowboy_hat_face:

















