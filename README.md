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

使得按下开关按钮，灯灭；松开开关按钮，灯亮.
