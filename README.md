# STM32 旋转编码器读取与串口输出

基于 **STM32F103C8T6** 和 **STM32 HAL** 的旋转编码器实验。TIM3 通过 A、B 两相输入进行硬件正交计数，主循环读取计数变化，再通过 USART1 将转动信息发送到电脑。

## 功能

- 转动编码器时，串口输出方向、相邻两次读数的差值和当前计数。
- 程序用 `HAL_GetTick()` 记录最后一次计数变化的时间。计数连续约 2 秒不变时，设计为提示一次，重新转动后再次开始计时。
- TIM3 使用 `TI12` 编码器模式，两路输入的边沿都参与计数。每个机械卡点对应的计数变化取决于编码器和定时器配置，顺逆时针字样取决于 A、B 相的接线顺序。

当前程序用 `int8_t` 存储计数读数和差值，串口显示范围为 −128 到 127。显示值的单位是计数。

## 硬件接线

| 外设信号 | STM32 引脚 | 用途 |
| --- | --- | --- |
| 编码器 A 相 | PA6 | TIM3_CH1 |
| 编码器 B 相 | PA7 | TIM3_CH2 |
| 编码器 GND | GND | 信号共地 |
| USB 转串口 RXD | PA9 | 接收 USART1_TX 的日志 |
| USB 转串口 GND | GND | 与 STM32 共地 |
| USB 转串口 TXD（可选） | PA10 | 对应 USART1_RX，查看日志时可留空 |

使用 **3.3 V 逻辑电平**的 USB 转串口。编码器模块的供电与引脚顺序按实物规格或原理图接线。当前 PA6、PA7 配置为无内部上下拉，编码器输出需提供明确的高、低电平。

## 构建与下载

工程由 STM32CubeMX 生成，使用 **CMake 3.22 或以上版本、Ninja 和 ARM GNU 工具链**。在 VS Code 打开仓库根目录，配置好 STM32Cube 工具链后选择 `Debug` 预设，执行 `CMake: Configure` 和 `CMake: Build`。

如果 CMake、Ninja 和 `arm-none-eabi-gcc` 已加入命令行的 PATH，也可以在仓库根目录执行：

```sh
cmake --preset Debug
cmake --build --preset Debug
```

构建产物为 `build/Debug/TIM_Encoder.elf`。可使用配置好的 CMSIS-DAP 和 OpenOCD 通过 SWD 下载。SWDIO 接 PA13，SWCLK 接 PA14，调试器与目标板共地。

## 串口观察

打开 USB 转串口对应的 COM 口，设置 **115200 波特、8 数据位、无校验、1 停止位、无流控**，文本编码选择 **UTF-8**。转动后停住，观察计数变化和静止提示。以下为输出格式示例。

```text
顺时针旋转1，当前位置：4
当前编码器静止超过2s，等待重新转动....
```

## 代码入口

| 文件 | 内容 |
| --- | --- |
| [Core/Src/main.c](Core/Src/main.c) | 轮询计数、计算差值、判断静止、重定向 `printf` |
| [Core/Src/tim.c](Core/Src/tim.c) | TIM3 编码器模式和 GPIO 输入配置 |
| [Core/Src/usart.c](Core/Src/usart.c) | USART1 与串口引脚配置 |
| [TIM_Encoder.ioc](TIM_Encoder.ioc) | STM32CubeMX 工程配置 |
| [CMakePresets.json](CMakePresets.json) | CMake 配置与构建预设 |
