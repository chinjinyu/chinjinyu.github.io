# AVR汇编（七）：位操作和MCU控制指令

## 位操作指令

### `SBI` / `CBI`

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230814171104.png)

`SBI` 指令用于设置I/O寄存器中的第 `b` 位， `CBI` 指令用于清除I/O寄存器中的第 `b` 位。

例如：

```asm
SBI DDRB, 5     ; PB5设为输出模式
CBI PORTB, 5    ; PB5输出低电平
```

### 移位

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230814173005.png)

`LSL` 指令用于逻辑左移，低位补0。

`LSR` 指令用于逻辑右移，高位补0。

`ASR` 指令用于算术右移，高位补符号位。

`ROL` 指令用于循环左移，低位补 `C` 标志位，高位进入 `C` 标志位。

`ROR` 指令用于循环右移，高位补 `C` 标志位，低位进入 `C` 标志位。

例如：

```asm
LDI R16, 0x88    ; R16 = 0x88
LSR R16          ; R16 = 0x44
LSL R16          ; R16 = 0x88
ASR R16          ; R16 = 0xC4
SEC              ; C = 1
ROR R16          ; R16 = 0xE2, C = 0
ROL R16          ; R16 = 0xC4, C = 1
```

### `SWAP`

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230814180411.png)

`SWAP` 指令用于交换寄存器的高低4位。

例如：

```asm
LDI R16, 0xA5    ; R16 = 0xA5
SWAP R16         ; R16 = 0x5A
```

### `BSET` / `BCLR`

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230814195813.png)

`BSET` 用于设置 `SREG` 寄存器中的第 `s` 位， `BCLR` 用于清除 `SREG` 寄存器中的第 `s` 位。

例如：

```asm
BSET 0    ; C = 1
BCLR 0    ; C = 0
```

### `BST` / `BLD`

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230814200755.png)

`BST` 用于将寄存器中的第 `b` 位保存到 `T` 标志位， `BLD` 用于将 `T` 标志位加载到寄存器的第 `b` 位。

例如：

```asm
LDI R16, 0xAA
BST R16, 1       ; T = 1
BLD R16, 0       ; R16 = 0xAB
```

### `SEx` / `CLx`

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230814201753.png)

形如 `SEx` 的指令用于设置 `SREG` 寄存器中对应的标志位，形如 `CLx` 的指令用于清除 `SREG` 寄存器中对应的标志位，这两条指令作用和 `BSET` / `BCLR` 指令相同。

例如：

```asm
SEC    ; SREG = 0x01
SEZ    ; SREG = 0x03
SEN    ; SREG = 0x07
SEV    ; SREG = 0x0F
SES    ; SREG = 0x1F
SEH    ; SREG = 0x3F
SET    ; SREG = 0x7F
SEI    ; SREG = 0xFF
CLI    ; SREG = 0x7F
CLT    ; SREG = 0x3F
CLH    ; SREG = 0x1F
CLS    ; SREG = 0x0F
CLV    ; SREG = 0x07
CLN    ; SREG = 0x03
CLZ    ; SREG = 0x01
CLC    ; SREG = 0x00
```

## MCU控制指令

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230814203325.png)

`NOP` 指令占用一个周期而不做任何操作。

`SLEEP` 指令用于进入睡眠模式。

`WDR` 指令用于复位看门狗。

`BREAK` 指令供调试系统使用，应用程序用不到。

## 参考资料

1. [ATmega328P Datasheet](https://ww1.microchip.com/downloads/en/DeviceDoc/Atmel-7810-Automotive-Microcontrollers-ATmega328P_Datasheet.pdf)
2. [AVR Instruction Set Manual](https://ww1.microchip.com/downloads/en/DeviceDoc/AVR-InstructionSet-Manual-DS40002198.pdf)