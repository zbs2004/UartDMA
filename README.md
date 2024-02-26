UART 空闲中断 + 不定长DMA收发
编译平台：Clion 烧录：OpenOCD
MCU：STM32F405RBT6TR
#准备工作
STM32CubeMx，Clion，OpenOCD
#准备库，生成初始文件
使用STM32Cube，在Connectivity中点击串口USART1，选择Asynchronous非同步模式，下方DMA Setting中点击ADD添加接收USART1_RX和发送USART1_TX，NVIC Setting中勾选USART1 global interrupt串口中断
若使用Clion编译项目则在Project Manager中的ToolChain/IDM选择STM32CubeIDE
左侧Code Generator中勾选Generator peripheral initialization as a pair of “.c/.h" files per peripheral
最后点击GENERATE CODE生成初始代码
#UART空闲中断
在USART1的初始化函数MX_USART1_UART_Init中加入
    __HAL_UART_ENABLE_IT(&huart1, UART_IT_IDLE);
以打开空闲中断
#DMA接收

在stm32f4xx_it.c中的USART1_IRQHandler中断处理函数下添加
   if((__HAL_UART_GET_FLAG(&huart1, UART_FLAG_IDLE) != RESET))//判断是否触发空闲
    {
        /* 1.清除标志 */
        __HAL_UART_CLEAR_IDLEFLAG(&huart1); //清除空闲标志
        /* 2.读取DMA */
        HAL_UART_DMAStop(&huart1); //先停止DMA，暂停接收?
        //这里应注意数据接收不要大 USART_DMA_RX_BUFFER_MAXIMUM
        head = 256 - (__HAL_DMA_GET_COUNTER(&hdma_usart1_rx)); //接收个数等于接收缓冲区大小减剩余计数//
        /* 3.储存数据 */
        memcpy(&received,&rxData,head);
        /* 4.清空缓存 */
        memset(&rxData,0,head);
        head = 0;
        /* 5.再次打开接收 */
        Uflag = 1;
        HAL_UART_Receive_DMA(&huart1,(uint8_t *)&rxData, 256);

    }


