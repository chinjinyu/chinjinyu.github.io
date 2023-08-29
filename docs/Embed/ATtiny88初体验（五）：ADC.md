# ATtiny88初体验（五）：ADC

## ADC模块介绍

ATtiny88单片机包含一个10bit分辨率的ADC模块，拥有8个通道，最大采样率15kSPS，转换时间14us。ATtiny88的ADC参考电压可以来自外部，也可以使用内部1.1V的电压源。支持自由运行模式和单次转换模式，支持多种自动触发源，在睡眠模式下拥有噪声消除器。

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230826181449.png)

注意：为了使用ADC模块， `PRR` 寄存器的 `PRADC` 位必须设为0。

ADC转换结果保存在 `ADCH` 和 `ADCL` 寄存器中，可以通过 `ADLAR` 位来选择左对齐还是右对齐。为了防止在读取ADC结果时，ADC结果发生改变，必须先读取 `ADCL` 寄存器，然后再读取 `ADCH` 寄存器。因为在读取 `ADCL` 寄存器时，ADC转换的结果会被锁定，直到 `ADCH` 寄存器被读取为止。

ADC有两种模式：单次触发模式和自由运行模式。在单次触发模式下，向 `ADSC` 位写1启动转换，转换完成后该位会自动清零；在自由运行模式下， `ADSC` 位会一直维持1。

在睡眠模式下，ADC模块可以启用噪声消除器减少来自CPU内核和其他I/O外设的噪声，方法如下：

1. 确保ADC使能并处于非忙状态，选择单次转换模式，使能ADC转换结束中断。
2. 进入ADC噪声减少模式（或空闲模式），当CPU停止工作时，ADC会开始一次转换。
3. 当转换结束时，ADC转换结束中断会唤醒CPU，除非再次使用睡眠命令，否则CPU会一直处于活跃状态。

注意：进入除空闲模式及ADC噪声减少模式外的其他睡眠模式时，不会自动关闭ADC，建议在进入这些睡眠模式时将 `ADEN` 位清零。

ADC转换结果与电压的关系如下式所示：

$$ADC=\frac{V_{IN} \times 1024}{V_{REF}}$$

ATtiny88内部有一个温度传感器，它连接到ADC8通道，在测量温度时，必须选择内部1.1V参考电压。

ADC的测量电压与温度约为线性关系，灵敏度约为1LSB/℃，典型值如下：

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230828162203.png)

为了获得更高的精度，可以使用如下公式进行软件校正：

$$T = k \times [(ADCH << 8) | ADCL] + T_{OS}$$

其中， $k$ 是斜率，是固定的，通常数值非常接近1， $T_{OS}$ 是传感器偏移量。

## 相关寄存器

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230828163112.png)

- `REFS0` ：参考电压选择。
	![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230828163347.png)
- `ADLAR` ：ADC结果左对齐，设为0右对齐，设为1左对齐。
- `MUX[3:0]` ：模拟通道选择。
	![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230828163516.png)

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230828163131.png)

- `ADEN` ：使能ADC。
- `ADSC` ：启动ADC转换，转换结束后自动清零。
- `ADATE` ：使能ADC自动触发。
- `ADIF` ：ADC中断标志位，中断程序执行结束后清零，或者也可以写1清零。
- `ADPS[2:0]` ：ADC分频选择，分频后的频率不要超过1MHz。
	![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230828164327.png)

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230828163155.png)

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230828163211.png)

- `ADTS[2:0]` ：ADC自动触发源选择。
	![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230828164456.png)

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230828163225.png)

- `ADCnD` ：关闭对应ADC引脚的数字输入缓冲。

## 代码

下面的代码展示了如何使用ATtiny88的ADC模块读取ADC0通道（PC0引脚）的电压值，代码文件的整体组织结构如下：

```
.
├── Makefile
├── inc
│   ├── serial.h
│   └── serial_stdio.h
└── src
    ├── main.c
    ├── serial.c
    └── serial_stdio.c
```

`src/main.c` 源文件的内容如下：

```c title='src/main.c'
#include <stdint.h>
#include <stdio.h>
#include <avr/io.h>
#include <avr/interrupt.h>
#include <serial_stdio.h>

static void delay(void);

int main(void)
{
    cli();
    stdio_setup();              // initialize stdio and redirect it to serial
    ADMUX = _BV(REFS0);         // external reference, align right, select channel ADC0(PC0)
    ADCSRA = _BV(ADEN) | _BV(ADIF) | _BV(ADPS2) | _BV(ADPS1) | _BV(ADPS0);
                                // enable ADC, clear ADC interrupt flag, disable ADC interrupt, division factor = 128
    DIDR0 = _BV(ADC0D);         // disable digital input buffer of ADC0 pin
    sei();

    for (;;) {
        ADCSRA |= _BV(ADSC);            // start conversion
        while (!(ADCSRA & _BV(ADIF)));  // wait for completion
        uint16_t value = ADCL;          // read low byte first
        value |= ADCH << 8;             // then read the high
        uint16_t voltage = (5000UL * value) >> 10;  // convert digital value to voltage
        printf("ADC0 value: 0x%04X, voltage: %dmV.\r\n", value, voltage);
        ADCSRA |= _BV(ADIF);            // clear flag
        delay();
    }
}

static void delay(void)
{
    for (volatile uint32_t i = 0; i < 0x8000; i++);
}
```

## 参考资料

1. [ATtiny88 Datasheet](https://ww1.microchip.com/downloads/en/DeviceDoc/doc8008.pdf)
