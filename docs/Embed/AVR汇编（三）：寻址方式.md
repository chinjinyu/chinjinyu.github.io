# AVR汇编（三）：寻址方式

AVR具有多种寻址方式，在介绍具体的汇编指令之前，有必要对它们做一定了解。

前面介绍过，AVR将内存空间分为多个部分：寄存器堆、I/O空间、数据空间、程序空间。这些空间支持的指令和寻址方式都各不相同。

## 寄存器堆的寻址方式

AVR中寄存器堆的寻址方式分为3种：立即寻址、单寄存器寻址、双寄存器寻址。

### 立即寻址

所谓立即寻址，就是指操作数直接编码在指令中。需要注意的是，只有 `R16` ~ `R31` 寄存器支持立即寻址。

例如：

```asm
ANDI R16, 0x01    ; R16 <- R16 & 0x01
```

### 单寄存器寻址

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230810115330.png)

在单寄存器寻址方式下，操作数存放在目的寄存器 `Rd` 中。

例如:

```asm
INC R16    ; R16 <- R16 + 1
```

### 双寄存器寻址

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230810115923.png)

在双寄存器寻址方式下，操作数分别存放在源寄存器 `Rr` 和目的寄存器 `Rd` 中。

例如：

```asm
ADD R16, R17    ; R16 <- R16 + R17
```

## I/O空间的寻址方式

AVR中I/O空间具有独立的地址，范围为0x00~0x3F，支持直接寻址和位寻址。

### 直接寻址

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230810120238.png)

在直接寻址方式下，操作数地址 `A` 直接编码在指令中。

例如：

```asm
OUT PORTB, R16    ; PORTB <- R16
```

### 位寻址

I/O空间中地址为0x00~0x1F的区域支持按位访问。

例如：

```asm
SBI PORTB, 2    ; PORTB.2 <- 1
```

## 数据空间的寻址方式

AVR中数据空间具有5种寻址方式：直接寻址、间接寻址、带前缀自减的间接寻址、带后缀自增的间接寻址、带偏移量的间接寻址。

### 直接寻址

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230810121541.png)

在直接寻址方式下，操作数地址直接编码在指令中， `Rd` / `Rr` 指定目的寄存器/源寄存器。

例如：

```asm
LDS R16, 0X0100    ; R16 <- (0X0100)
```

### 间接寻址

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230810121935.png)

在间接寻址方式下，操作数地址存放在 `X` / `Y` / `Z` 寄存器中。

例如：

```asm
LD R16, X    ; R16 <- (X)
```

### 带前缀自减的间接寻址

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230810122759.png)

在带前缀自减的间接寻址方式下，操作数地址存放在 `X` / `Y` / `Z` 寄存器中，并且在操作之前 `X` / `Y` / `Z` 先自减一。

例如：

```asm
LD R16, -X    ; X <- X - 1, R16 <- (X)
```

### 带后缀自增的间接寻址

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230810123124.png)

在带后缀自增的间接寻址方式下，操作数地址存放在 `X` / `Y` / `Z` 寄存器中，并且在操作之后 `X` / `Y` / `Z` 再自增一。

例如：

```asm
LD R16, X+    ; R16 <- (X), X <- X + 1
```

### 带偏移量的间接寻址

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230810123446.png)

在带偏移量的间接寻址方式下，操作数地址由 `Y` / `Z` 寄存器和编码在指令中的偏移量 `q` 相加得出。

例如：

```asm
LDD R16, Y+2    ; R16 <- (Y + 2)
```

## 程序空间的寻址方式

AVR中程序空间具有5种寻址方式：常量寻址、带后缀自增的常量寻址、直接寻址、间接寻址、相对寻址，前两种寻址方式用于访问位于程序空间的常量，后三种用于程序跳转。

### 常量寻址

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230810124306.png)

在常量寻址方式下，操作数地址（字节地址）存放在 `Z` 寄存器中， `Z` 寄存器中高15位指定字地址，最低位为0表示访问的是字的低位字节，1表示访问的是字的高位字节。

例如：

```asm
LPM R16, Z    ; R16 <- (Z)
```

### 带后缀自增的常量寻址

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230810125032.png)

在带后缀自增的常量寻址方式下，操作数地址（字节地址）存放在 `Z` 寄存器中，并且在操作之后 `Z` 寄存器自增一。

例如：

```asm
LPM R16, Z+    ; R16 <- (Z), Z <- Z + 1
```

### 直接寻址

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230810125828.png)

在直接寻址方式下，程序接下来的执行地址（字地址） `k` 直接编码在指令中。

例如：

```asm
	JMP FUNC    ; PC <- FUNC
	...
FUNC:
	...
```

### 间接寻址

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230810130415.png)

在间接寻址方式下，程序接下来的执行地址（字地址）存放在 `Z` 寄存器中。

例如：

```asm
IJMP    ; PC <- Z
```

### 相对寻址

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230810130944.png)

在相对寻址方式下，程序接下来的执行地址（字地址）为 `PC` + `k` + 1， `k` 的范围为-2048~2047。

例如：

```asm
	RJMP FUNC    ; PC <- FUNC 汇编器会自动计算k的值
	...
FUNC:            ; FUNC与RJMP指令的距离不能超过-2048~2047
	...
```

## 参考资料

1. [AVR Instruction Set Manual](https://ww1.microchip.com/downloads/en/DeviceDoc/AVR-InstructionSet-Manual-DS40002198.pdf)
