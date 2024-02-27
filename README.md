<div id="top"></div>
<!--
*** Thanks for checking out the Best-README-Template. If you have a suggestion
*** that would make this better, please fork the repo and create a pull request
*** or simply open an issue with the tag "enhancement".
*** Don't forget to give the project a star!
*** Thanks again! Now go create something AMAZING! :D
-->



<!-- PROJECT SHIELDS -->
<!--
*** I'm using markdown "reference style" links for readability.
*** Reference links are enclosed in brackets [ ] instead of parentheses ( ).
*** See the bottom of this document for the declaration of the reference variables
*** for contributors-url, forks-url, etc. This is an optional, concise syntax you may use.
*** https://www.markdownguide.org/basic-syntax/#reference-style-links
-->






<!-- PROJECT LOGO -->
<br />
<div align="center">
  <a href="https://github.com/zbs2004/UartDMA">
    <img src="images/icon.png" alt="Logo" width="100" height="100">
  </a>

  <h3 align="center">Uart空闲中断+DMA不定长收发</h3>

  <p align="center">
    高效的串口接收与打印方案!
    <br />
    <a href="https://github.com/zbs2004/UartDMA/"><strong>浏览文档 »</strong></a>
    <br />
    <br />
    <a href="https://github.com/zbs2004/UartDMA/">查看 Demo</a>
    ·
    <a href="https://github.com/othneildrew/Best-README-Template/issues">反馈 Bug</a>
    ·
    <a href="https://github.com/othneildrew/Best-README-Template/issues">请求新功能</a>
  </p>
</div>



<!-- TABLE OF CONTENTS -->
<details>
  <summary>目录</summary>
  <ol>
    <li>
      <a href="#关于本项目">关于本项目</a>
      <ul>
        <li><a href="#构建工具">构建工具</a></li>
      </ul>
    </li>
    <li>
      <a href="#开始">开始</a>
      <ul>
        <li><a href="#编译平台准备">编译平台准备</a></li>
        <li><a href="#安装">安装</a></li>
      </ul>
    </li>
    <li><a href="#代码详情">代码详情</a></li>
      <ul>
        <li><a href="#stm32cubemx配置">STM32CubeMX配置</a></li>
        <li><a href="#clion编译openocd烧录">Clion编译&openocd烧录</a></li>
        <li><a href="#uart空闲中断">UART空闲中断</a></li>
        <li><a href="#dma接收">DMA接收</a></li>
        <li><a href="#dma发送">DMA发送</a></li>
        <li><a href="#测试函数">测试函数</a></li>
      </ul>
    <li><a href="#贡献">贡献</a></li>
    <li><a href="#联系我们">联系我们</a></li>
    <li><a href="#致谢">致谢</a></li>
  </ol>
</details>



<!-- ABOUT THE PROJECT -->
## 关于本项目

[![Product Name Screen Shot][product-screenshot]](https://github.com/zbs2004/UartDMA/)

基于HAL库，使用UART 空闲中断 + DMA实现不定长收发 MCU：STM32F405RBT6TR

<p align="right">(<a href="#top">回到顶部</a>)</p>



### 构建工具

* [STM32CubeMX](https://www.st.com/zh/development-tools/stm32cubemx.html)
* [Clion](https://www.jetbrains.com/clion/)
* [OpenOCD](https://openocd.org/)
* [MinGW](https://www.mingw-w64.org/)

<p align="right">(<a href="#top">回到顶部</a>)</p>



<!-- GETTING STARTED -->
## 开始

### 编译平台准备

配置好STM32CubeMX和Clion,将OpenOCD下载并解压到你方便的任一地址

### 安装

1. 克隆本仓库
   ```sh
   git clone https://github.com/zbs2004/UartDMA.git
   ```

2. Clion自动配置Cmake

3. 编译并烧录

4. 打开串口传输工具，这里使用[JCOM](http://www.jooiee.com/cms/ruanjian/115.html)

5. 连接设备，选择正确的串口，波特率设置为115200

6. 向单片机发送的消息，将被传送回上位机

<p align="right">(<a href="#top">回到顶部</a>)</p>



<!-- USAGE EXAMPLES -->
## 代码详情

### STM32CubeMX配置

1. 打开STM32CubeMX，在Connectivity中点击串口USART1，选择Asynchronous非同步模式

2. 下方DMA Setting中点击ADD添加接收USART1_RX和发送USART1_TX，NVIC Setting中勾选USART1 global interrupt串口中断

3. 若使用Clion编译项目则在Project Manager中的ToolChain/IDM选择STM32CubeIDE

4. 左侧Code Generator中勾选Generator peripheral initialization as a pair of “.c/.h" files per peripheral

5. 最后点击GENERATE CODE生成初始代码

### Clion编译&OpenOCD烧录

- 在Clion中打开项目文件夹

- 在配置好Cmake的设备上Clion会自动弹出配置项目cmakelist.txt，跟着提示操作即可

- 在项目配置中添加OpenOCD编译并运行 配置正确的可执行文件，根据烧录设备选择烧录配置文件
<p align="right">(<a href="#top">回到顶部</a>)</p>

### UART空闲中断

在USART1的初始化函数MX_USART1_UART_Init中加入
```sh
  __HAL_UART_ENABLE_IT(&huart1, UART_IT_IDLE);
``` 
以打开空闲中断

### DMA接收

在stm32f4xx_it.c中的USART1_IRQHandler中断处理函数下添加用户的中断处理： 

1. 检测是否触发空闲中断

2. 若触发则关闭空闲中断防止重复触发

3. 关闭DMA接收 

4. 计算接收数据长度,注意此处数据长度不能超过接收缓存的最大长度（256） 

5. 复制接收缓存至接收数组 

6. 清空接收缓存 

7. 给出接收完成标志&开启下一步接收 注意主函数中需要添加第一次接收以打开接收

### DMA发送

在Usart.c中添加DMA发送函数DMA_printf，以下为发送处理：

1. 检测上一次发送是否完成 

2. 计算发送数据长度 

3. 根据数据长度发送数据

4. 给出发送中的标志位阻挡下一次发送

5. 发送完成后开启下一次发送

此处可利用宏定义将printf定向为DMA发送函数实现串口打印

### 测试函数

在main.c中添加测试函数CMD,测试操作： 将单片机接收的数据发送回上位机

具体代码细节见对应文件


<!-- CONTRIBUTING -->
## 贡献

贡献让开源社区成为了一个非常适合学习、启发和创新的地方。你所做出的任何贡献都是受人尊敬的。

代码详情出给出了完整的设计思路，希望STM32初学者可以复刻本项目，以练习使用HAL库编程，

不要忘记给项目点一个 star！再次感谢！

<p align="right">(<a href="#top">回到顶部</a>)</p>


<!-- CONTACT -->
## 联系我们

赵宝硕 - @我的邮箱 - zbs2004gg@outlook.com

项目链接: [https://github.com/zbs2004/UartDMA/](https://github.com/zbs2004/UartDMA/)

<p align="right">(<a href="#top">回到顶部</a>)</p>



<!-- ACKNOWLEDGMENTS -->
## 致谢

* [STM32---UART使用DMA数据传输](https://blog.csdn.net/li391402/article/details/117559727)
* [STM32 HAL 之 UART：空闲中断结合DMA实现不定长数据收发](https://blog.csdn.net/xuzhexing/article/details/107926788)
* [Best-README-Template-zh](https://github.com/BreakingAwful/Best-README-Template-zh)

<p align="right">(<a href="#top">回到顶部</a>)</p>



<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->

[product-screenshot]: images/screenshots.png
