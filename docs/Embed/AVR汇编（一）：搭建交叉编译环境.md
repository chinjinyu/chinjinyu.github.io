# AVR汇编（一）：搭建交叉编译环境

几年间，陆陆续续接触了很多热门的单片机，如STC、STM8S、STM32、ESP32等。但一直都是抱着急功近利的心态去学习他们，基本上都是基于库函数和第三方组件进行开发，很少静下心来去研究这些不同内核单片机的底层工作原理。因此我打算接下来一段时间好好研究一番，先从相对容易的AVR内核开始。

AVR是Atmel推出的一个8位的RISC微控制器内核，哈佛架构，具备1MIPS/MHz的高速运行处理能力。本文将介绍在Linux系统下搭建AVR交叉编译环境，以及仿真AVR程序的方法，还会提到一些常用的GDB调试命令。

## 搭建AVR交叉编译环境

主要安装 `avr-gcc` 、 `make` 、 `simavr` 软件，前两者用于编译，后者用于仿真。

从Microchip官网下载[GCC Compilers for AVR](https://www.microchip.com/en-us/tools-resources/develop/microchip-studio/gcc-compilers)，选择“AVR 8-Bit Toolchain (Linux)”。

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230808182146.png)

下载后得到 `avr8-gnu-toolchain-3.7.0.1796-linux.any.x86_64.tar.gz` 文件，将之解压到合适位置：

```bash
tar -zxvf avr8-gnu-toolchain-3.7.0.1796-linux.any.x86_64.tar.gz -C /path/to/avr-gcc
```
其中， `-C` 指定解压目录。

解压完成后，得到 `avr8-gnu-toolchain-linux_x86_64` 文件夹，`avr-gcc` 所有的编译工具、库、头文件等都存放在它下面，其中 `bin` 文件夹是 `avr-gcc` 等主要可执行文件的位置。

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230809102515.png)

将 `avg-gcc` 所在的目录添加到 `PATH` 环境变量，然后重新加载终端：

```bash
echo -e "\nexport PATH=\$PATH:/path/to/avr-gcc/avr8-gnu-toolchain-linux_x86_64/bin" >> ~/.zshrc
source ~/.zshrc
```

检查 `avr-gcc` 是否安装成功，如果成功，则会正常输出版本信息：

```bash
avr-gcc --version
```

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230808200738.png)

安装 `make` 和 `simavr` ：

```bash
sudo apt update
sudo apt install make simavr
```

## 编译及仿真

首先准备一个用于仿真的源代码 `hello.c` ，内容如下：

```c title='hello.c'
#include <avr/io.h>
#include <stdint.h>

static void delay(void) {
    for (volatile uint16_t i = 0; i < 0x8000; i++);
}

int main(void)
{
    uint8_t mask = 1 << 5;
    DDRB |= mask;       // set PB5 to output mode
    PORTB &= ~mask;     // PB5 = 0

    for (;;) {
        PINB = mask;    // toggle PB5
        delay();
    }
}
```

这段代码干的事情很简单，设置 `PB5` 为输出模式，然后不断翻转 `PB5` 的输出电平。

然后编写 `Makefile` 文件：

```makefile title='Makefile'
.PHONY: all clean
all: hello.elf

hello.o: hello.c
        avr-gcc -mmcu=atmega328p -c -g -Wall -Og -std=gnu99 -o $@ $^

hello.elf: hello.o
        avr-gcc -mmcu=atmega328p -o $@ $^

clean:
        rm -rf *.o *.elf
```

文件定义的最终编译目标是 `hello.elf` ，其中不要忘了指定 `avr-gcc` 的 `-mmcu` 选项，这里 `-mmcu=atmega328p` 表示编译ATmega328P单片机的代码。

之后执行 `make` 进行编译，生成 `hello.elf` 文件。

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230808210506.png)

接下来对 `hello.elf` 进行仿真，这主要借助 `simavr` 和 `avr-gdb` 来实现。

```bash
simavr -f 16000000 -m atmega328p --gdb hello.elf
```

这条命令中 `-f` 设置仿真频率， `-m` 指定仿真的单片机型号，可以通过 `simavr --list-cores` 查看所有支持的单片机型号， `--gdb` 开启GDB服务，监听端口为 `1234` 。

再打开另一个终端窗口，用于执行 `avr-gdb` 。

```bash
avr-gdb -ex "target remote localhost:1234" -q --tui hello.elf
```

执行这条命令后，将进入 `avr-gdb` 调试界面。

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230808212513.png)

## 常用GDB命令

| 命令 | 功能 | 示例 |
| --- | --- | --- |
| `help` | 查看帮助 | `help all` 查看所有命令<br/> `help print` 查看 `print` 命令的帮助信息 |
| `target remote` | 连接到远程GDB服务器 | `target remote localhost:1234` 连接到本地端口号为 `1234` 的GDB服务器 |
| `layout` | 设置窗口布局 | `layout regs` 显示寄存器窗口<br/> `layout src` 显示源码窗口<br/> `layout split` 显示源码和反汇编窗口 |
| `break` | 设置断点 | `break n` 在第 `n` 行设置断点<br/> `break func` 在 `func` 函数入口处设置断点 |
| `print` | 打印表达式的值 | `print/x var` 以十六进制形式打印变量 `var` 的值 |
| `display` | 在程序每次暂停时打印表达式的值 | `display/x $r24` 以十六进制形式在每次程序暂停时打印 `r24` 寄存器的值 |
| `info registers` | 显示寄存器的内容 | `info registers r24` 显示 `r24` 寄存器的内容 |
| `continue` | 继续运行 | |
| `next` | 单步调试（不进入函数） | `next n` 执行 `n` 步 |
| `step` | 单步调试（进入函数） | `step n` 执行 `n` 步 |
| `backtrace` | 显示当前堆栈 | |
| `list` | 查看源码 | `list n` 显示第 `n` 行前后10行代码<br/> `list func` 显示 `func` 函数的源代码|
| `quit` | 退出GDB | |

## 参考资料

1. [GCC Compilers for AVR](https://www.microchip.com/en-us/tools-resources/develop/microchip-studio/gcc-compilers)
2. [buserror/simavr](https://github.com/buserror/simavr)
3. [GDB常用命令](https://zhuanlan.zhihu.com/p/474736535)
4. [GDB User Manual](https://sourceware.org/gdb/current/onlinedocs/gdb)
