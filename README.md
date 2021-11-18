![](/5.Docs/Images/Holo1.jpg)
# HoloCubic--Multifunctional transparent display desktop station

**Video introduction:** https://www.bilibili.com/video/BV1VA411p7MD/

## 0. About this project

> Zhihui Jun's Note: This is to update the video and ease the embarrassment of delaying the update for two months. An interesting gadget that I rushed out in a weekend :D

As mentioned in the video, the interesting part of this project is the use of a dichroic prism to design the effect of `pseudo holographic display`. In general, this small device has more functions, because it is equipped with WiFi and Bluetooth capabilities to achieve many network applications. In this warehouse, we provide you with a development framework and some basic functions (weather, fan count monitor, etc.), everyone I can continue to expand and implement more applications based on my plan.

The hardware solution of this project is based on `ESP32PICO-D4`, a very practical MCU chip of Espressif. Due to the use of SiP package, the entire board area of ​​PCBA can be the size of a coin; the software is mainly based on `lvgl- GUI` library, I transplanted ST7789 1.3 inch `240x240` resolution screen display driver, and at the same time use `MPU6050` as an input device to simulate the encoder key value through induction to interact.

## 1. Hardware proofing instructions

**For PCB proofing, I haven't found anything that needs special attention for the time being. ** The PCB file can be directly taken to the factory for proofing. The two-layer board is very cheap, and the device BOM is also more commonly used. The entire board cost is less than 50 yuan.

The `Hardware` file currently contains two versions of the PCB circuit:

* **Naive Version**: The version that appears in the video, onboard ESP32, IMU, ambient light sensor, SD card slot, download circuit, and two RGB lights
* **Ironman Version**: Based on the above version slightly modified, the ambient light sensor is deleted, and the PCB shape is modified to fit the new housing

> Because the new shell plan is to use CNC for metal processing, the ambient light is easily blocked, and there are not many scenarios for this function, so it is deleted in the new version.

**Shell processing** According to the version you like, the `3D Model` folder currently contains four versions of shell files:

* **Naive Version**: The version that appears in the video, which is relatively simple (because it is designed for temporary work), and it is best to use light-curing 3D printing processing

  ![](/5.Docs/Images/Holo3.png)

* **Bilibili Version**: The shell structure of the Hundred trophy of Station B that appears in the back of the video, adapts to the PCB of the `Naive Version`, **It is of entertainment nature, and it is not recommended for non-Baba UP**.

* **Metal Version**: After the video is released, the newly revised housing structure design optimizes the layout and controls to be more compact and exquisite. It is suitable for the PCB of `Naive Version`. CNC processing is recommended.

  ![](/5.Docs/Images/Holo2.jpg)

* **Ironman Version**: Newly designed Wild Iron Man style structure, this version is designed in cooperation with a friend, and he may be authorized to mass produce it later. This structure is adapted to the PCB of the `Ironman Version`

  ![](/5.Docs/Images/Holo.jpg)

> The processing of the structural parts of the joint version of Wild Iron Man is more complicated, and it requires post-sandblasting, anodizing and other processes, so the single-piece manufacturing cost is very high (inquired about the next set of 3 parts at least 1,000 yuan +), so you have your own For processing channels, you can use the provided documents to do it yourself.
>
> If you don’t have a channel but you want this version of the hardware, **I authorized that friend to mass produce a small batch**. His shop is called [Xikii](https://shop68240117.taobao.com) and is a guest A geek who is very experienced in customizing the keyboard, if you are interested, you can pay attention to it~

## 2. Firmware compilation instructions

The firmware framework is mainly developed based on Arduino. If you have played with Arduino, there is basically no difficulty in getting started. Just install the library in Firmware/Libraries to the Arduino library directory (if you are using Arduino IDE).

> I use the Visual Micro plug-in on Visual Studio for Arduino development. Because I am familiar with VS, you can choose your favorite IDE.

**Then you need to modify an official library file to use it normally:**

First of all, you must install the ESP32 Arduino support package (Baidu has a large number of tutorials), and then in the `esp32\hardware\esp32\1.0.4\libraries\SPI\src\SPI.cpp` file of the installed support package, **modify The MISO in the following code is 26**:

    if(sck == -1 && miso == -1 && mosi == -1 && ss == -1) {
        _sck = (_spi_num == VSPI)? SCK: 14;
        _miso = (_spi_num == VSPI)? MISO: 12; // Need to be changed to 26
        _mosi = (_spi_num == VSPI)? MOSI: 13;
        _ss = (_spi_num == VSPI)? SS: 15;
This is because two hardware SPIs are used to connect the screen and SD card on the hardware. The default MISO pin of HSPI is 12, and 12 is used in ESP32 to set the flash level when powering on, and power on before powering on. Pulling will cause the chip to fail to start, so we replaced the default pin with 26.

> This problem can also be solved by setting the chip fuse, but that kind of operation is one-time irreversible and it is not recommended to play this way.

**in addition:**

Since I rushed to make the video, the code was written temporarily and very messy with a lot of dirty code, so the template code in the warehouse is all the template code after the driver is adjusted, and you can freely develop it based on this framework.

**APP application code will be updated gradually as I organize it. **

## 3. Visual Studio Simulator & Image Conversion Script

A Visual Studio project is included in the `Software` folder. After opening it with VS (C++ development components are required), the LVGL interface effect can be simulated on the computer. After the modification, the code can be pasted to the Arduino firmware to complete the interface. transplant.

> This saves the need to re-compile the Arduino firmware for every modification to improve development efficiency.

![](/5.Docs/Images/Holo4.jpg)

The `ImageToHolo` folder contains a Python script to convert images into image resources used in the HoloCubic firmware.

> Because image resources generally take up more space, if they are all stored in ESP32 Flash, there are not a few that can be stored. Therefore, I transplanted LVGL's FAT file system support into the framework, and the image resources can be stored in the SD card for reading.
>
> The official image conversion tool is online: [https://lvgl.io/tools/imageconverter](https://lvgl.io/tools/imageconverter), you need to select the `Indexed 4 colors` format.
>
> **But the official tool can only convert one piece at a time and uploading and downloading is very troublesome**, so I wrote a script for batch conversion.

The image resource used by HoloCubic is named `xxx.bin` file, you can use the script I provided to convert it and put it into the SD card, and then you can read it like this:

```
lv_obj_t* imgbtn = lv_imgbtn_create(lv_scr_act(), NULL);
lv_imgbtn_set_src(imgbtn, LV_BTN_STATE_PRESSED, "S:/dir/icon_pressed.bin");
lv_imgbtn_set_src(imgbtn, LV_BTN_STATE_RELEASED, "S:/dir/icon_released.bin");
```

Among them, `S:` refers to the root directory of the SD card (note that **S is capitalized**), and the following is exactly the same as the path in Linux.

> This script refers to the implementation of [W-Mai/lvgl_image_converter](https://github.com/W-Mai/lvgl_image_converter).



**In addition, because the conversion script needs to be in the Python environment, if you don’t want to install the environment, you can also use my pre-compiled exe file to transfer. The method of use is very simple. Drag the `jpg/png/bmp` picture to Just click on the icon of `holoconverter.exe` (you can drag multiple upwards at the same time), and the corresponding `.holo` file will be generated in the current directory. **

> Download link of the converter software:
>
> Link: https://pan.baidu.com/s/11cPOVYnKkxmd88o-Ouwb5g Extraction code: xlju

## 4. About Dichroic Prism

I used a 25.4mm x 25.4mm x 25.4mm prism, which should be available on Taobao. The single price is about 80 yuan.

Fixing the spectroscopic prism is more troublesome. If glue is used, it will easily penetrate into the screen and cause watermarks. Therefore, it is recommended to search for `OCA glue` in TB. This is a solid glue used to bond the screen in the `Full-fit screen process` Not bad and very cheap.

> But OCA is very sticky, you must be careful not to leave bubbles in the operation, otherwise it will be difficult to remove after sticking.

## Other follow-ups will be added, if useful, remember to point the stars~


# HoloCubic--多功能透明显示屏桌面站

**视频介绍：** https://www.bilibili.com/video/BV1VA411p7MD/

## 0. 关于本项目

> 稚晖君注：这是为了更新视频，缓解本人拖更两个月的尴尬，用一个周末时间赶出来的一个有意思的小玩意 :D

如视频所述，本项目有意思的地方在于使用了一个分光棱镜来设计出`伪全息显示`的效果。这个小设备总的来说功能比较多，因为搭载了WiFi和蓝牙能力可以实现很多网络应用，在本仓库中给大家提供了一个开发框架以及一些基础功能（天气、粉丝数监视器等），大家可以基于我的方案继续扩展实现更多应用。

本项目的硬件方案是基于`ESP32PICO-D4`的，乐鑫的一个很实用的MCU芯片，由于采用了SiP封装是的PCBA整板面积能做到一个硬币大小；软件方面主要是基于`lvgl-GUI`库，我移植了ST7789 1.3寸`240x240`分辨率屏幕的显示屏驱动，同时将`MPU6050`作为输入设备通过感应的方式模拟编码器键值来交互。

## 1. 硬件打样说明

**PCB打样的话暂时没发现有啥需要特别注意的。** PCB文件可以直接拿去工厂打样，两层板很便宜，器件BOM的话也都是比较常用的，整板成本在50元以内。

`Hardware`文件内目前包含两个版本的PCB电路：

* **Naive Version** ：即视频中出现的版本，板载ESP32、IMU、环境光传感器、SD卡槽、下载电路、以及两个RGB灯
* **Ironman Version** ：基于上面的版本轻微修改，删去了环境光传感器，修改了PCB形状以适配新的外壳

> 因为新款的外壳计划是使用CNC进行金属加工，因此环境光容易被遮挡，而且该功能使用场景不多所以在新版删去了。

**外壳加工** 根据自己喜欢的版本选择，`3D Model`文件夹目前包含四个版本的外壳文件：

* **Naive Version** ：即视频中出现的版本，比较简约（因为临时赶工设计的），最好使用光固化3D打印加工

  ![](/5.Docs/Images/Holo3.png)

* **Bilibili Version** ：视频中后面出现的B站百大奖杯形式的外壳结构，适配`Naive Version`的PCB， **属于娱乐性质，非百大UP不建议采用**。

* **Metal Version** ：视频发布后全新改版的外壳结构设计，优化了布局控件整体更紧凑精致，适配`Naive Version`的PCB，该建议使用CNC加工制作

  ![](/5.Docs/Images/Holo2.jpg)

* **Ironman Version** ：新设计的野生钢铁侠风格结构件，该版本为和朋友合作设计的，后面可能会授权他联名量产，该结构适配`Ironman Version`的PCB

  ![](/5.Docs/Images/Holo.jpg)

> 野生钢铁侠联名的版本的结构件加工比较复杂，而且需要后期喷砂、阳极氧化等工艺所以单件制造成本很高（打听了下整套3个部件至少要1000元+），因此大家自己有加工渠道的可以用提供的文件自己去做。
>
> 没有渠道但是又想要这个版本硬件的，**我授权了那位朋友量产一小批**，他的店铺名为[Xikii](https://shop68240117.taobao.com)，是做客制化键盘很有经验的一个极客，大家感兴趣的可以去关注一下~

## 2. 固件编译说明

固件框架主要基于Arduino开发完成，玩过Arduino的基本没有上手难度了，把Firmware/Libraries里面的库安装到Arduino库目录（如果你用的是Arduino IDE的话）即可。

> 我使用的是Visual Studio上面的Visual Micro插件进行Arduino开发，因为对VS比较熟悉，大家选择自己喜欢的IDE就好了。

**然后这里需要修改一个官方库文件才能正常使用：**

首先肯定得安装ESP32的Arduino支持包（百度有海量教程），然后在安装的支持包的`esp32\hardware\esp32\1.0.4\libraries\SPI\src\SPI.cpp`文件中，**修改以下代码中的MISO为26**：

    if(sck == -1 && miso == -1 && mosi == -1 && ss == -1) {
        _sck = (_spi_num == VSPI) ? SCK : 14;
        _miso = (_spi_num == VSPI) ? MISO : 12; // 需要改为26
        _mosi = (_spi_num == VSPI) ? MOSI : 13;
        _ss = (_spi_num == VSPI) ? SS : 15;
这是因为，硬件上连接屏幕和SD卡分别是用两个硬件SPI，其中HSPI的默认MISO引脚是12，而12在ESP32中是用于上电时设置flash电平的，上电之前上拉会导致芯片无法启动，因此我们将默认的引脚替换为26。

> 也可以通过设置芯片熔丝的方式解决这个问题，不过那样的操作是一次性不可逆的，不建议这么玩。

**另外：**

由于我赶视频制作，代码都是临时写的非常杂乱有很多dirty code，因此仓库中的是所有驱动调通之后的模板代码，可以自己基于这个框架自由开发。

**APP应用代码我在整理中慢慢也会更新出来。**

## 3. Visual Studio模拟器 & 图片转换脚本

在`Software`文件夹中包含了一个Visual Studio的工程，用VS打开（需要安装C++开发组件）后可以在电脑上模拟LVGL的界面效果，改好之后代码粘贴到Arduino固件那边就可以完成界面移植。

> 这样省的每次修改都要重新交叉编译Arduino的固件，提升开发效率。

![](/5.Docs/Images/Holo4.jpg)

`ImageToHolo`文件夹下包含一个Python脚本，用于将图片转换成HoloCubic固件中用到的图像资源。

> 因为图像资源一般都比较占空间，如果全部存在ESP32的Flash中的话存不了几张，因此我在框架中移植了LVGL的FAT文件系统支持，可以将图片资源存储在SD卡内进行读取。
>
> 官方的图转换工具是在线的：[https://lvgl.io/tools/imageconverter](https://lvgl.io/tools/imageconverter) ，需要选择 `Indexed 4 colors` 格式。
>
> **但是官方工具每次只能转换一张还要上传下载很麻烦**，因此我自己写了个脚本用于批量转换。

HoloCubic用到的图片资源名为`xxx.bin`文件，大家用我提供的脚本转好后放入SD卡，然后可以像这样读取：

```
lv_obj_t* imgbtn = lv_imgbtn_create(lv_scr_act(), NULL);
lv_imgbtn_set_src(imgbtn, LV_BTN_STATE_PRESSED, "S:/dir/icon_pressed.bin");
lv_imgbtn_set_src(imgbtn, LV_BTN_STATE_RELEASED, "S:/dir/icon_released.bin");
```

其中`S:`指代SD卡根目录（注意**S是大写的**），后面就是跟Linux中的路径完全表示一致了。

> 该脚本参考了[W-Mai/lvgl_image_converter](https://github.com/W-Mai/lvgl_image_converter) 的实现。



**另外由于转换脚本的使用需要再Python环境下，如果大家不想安装环境的话，也可以用我预编译好的exe文件来转，使用方法很简单，把`jpg/png/bmp`图片拖到`holo转换器.exe`的图标上就行了（可以同时拖动多个上去），会在当前目录生成对应的`.holo`文件。**

> 转换器软件的下载地址：
>
> 链接：https://pan.baidu.com/s/11cPOVYnKkxmd88o-Ouwb5g  提取码：xlju 

## 4. 关于分光棱镜

我用的时25.4mm x 25.4mm x 25.4mm的棱镜，淘宝应该可以搜到，单个价格80元左右。

分光棱镜的固定比较麻烦，用胶水的话容易渗入屏幕导致水印，因此建议去TB搜一下`OCA胶`，这是一种`全贴合屏幕工艺`中用来粘合屏幕的固态胶，效果很不错也很便宜。

> 但是OCA粘性非常强，大家操作一定要仔细不要留气泡，不然粘上后就很难取下了。

## 其他的后续再补充，有用的话记得点星星~

