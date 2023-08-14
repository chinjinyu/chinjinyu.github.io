# AVR汇编（六）：分支指令

分支指令用于改变程序的执行流，分为无条件分支和条件分支两类。

## 无条件分支指令

### `JMP`

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230813191745.png)

`JMP` 指令用于无条件跳转，类似于C中的 `goto` 关键字， `JMP` 指令的跳转范围为\[0, 4M-1\]字。

`RJMP` 指令用于相对跳转，跳转范围为当前位置\[-2K, 2K-1\]字。

`IJMP` 指令用于间接跳转，跳转的目的地址存放在 `Z` 寄存器中（记住单位是字）。

例如：

```asm
    JMP f2             ; 跳转到f2
f1:
    RJMP f3            ; 跳转到f3
f2:
    LDI ZL, lo8(f1)
    LDI ZH, hi8(f1)    ; Z = f1
    CLC
    ROR ZH
    ROR ZL             ; Z = Z >> 1
    IJMP               ; 跳转到f1
f3:
    RJMP f2            ; 跳转到f2
```

注意：实测在GNU汇编下， `IJMP` 指令中不能直接把标签赋值给 `Z` 寄存器，因为标签表示的地址的单位是字节，而 `Z` 寄存器中存放的应该是字地址，所以要将标签右移一位传给 `Z` 寄存器。而 `JMP` 指令和 `RJMP` 指令则可以直接传标签。

### `CALL` / `RET`

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230813200102.png)

`CALL` 指令用于子程序调用，和 `JMP` 指令一样，也可以实现程序跳转，但是 `CALL` 指令在跳转之前会将下一条指令的地址（返回地址）压入栈中。 `CALL` 指令的跳转范围为\[0, 64K-1\]字。

`RCALL` 指令用于相对子程序调用，跳转范围为当前位置\[-2K, 2K-1\]字。

`ICALL` 指令用于间接子程序调用，子程序的地址存放在 `Z` 寄存器中（记住单位是字）。

`RET` 指令用于子程序返回，先将返回地址从栈中弹出，然后进行跳转。

`RETI` 指令用于中断子程序返回，和 `RET` 指令不同的是，它还会设置全局中断使能位 `I` 。

例如：

```asm
    CALL f1            ; 调用f1子程序
    RCALL f1           ; 调用f1子程序
    LDI ZL, lo8(f1)
    LDI ZH, hi8(f1)    ; Z = f1
    CLC
    ROR ZH
    ROR ZL             ; Z = Z >> 1
    ICALL              ; 调用f1子程序
f1:
    ...
    RET                ; 子程序返回

XXX_IRQHandler:
    ...
    RETI               ; 中断子程序返回
```

## 条件分支指令

### `CP`

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230814101058.png)

`CP` 指令用于比较，实际上就是只影响标志位而不保存结果的减法操作。后缀带 `C` 表示带进位比较，后缀带 `I` 表示与立即数比较，后缀带 `SE` 表示如果相等，则跳过下一条指令。

例如：

```asm
    LDI R16, 0x01
    LDI R17, 0x02
    CP R16, R17
    BRLT f1          ; 1 < 2，跳转到f1
    RJMP f3          ; 不会执行
f1:
    CPI R16, 0x01
    BREQ f2          ; 1 == 1，跳转到f2
    RJMP f3          ; 不会执行
f2:
    LDI R17, 0x01
    CPSE R16, R17    ; 1 == 1，跳过下一条指令
    RJMP f3          ; 不会执行
    SEC
    CPC R16, R17
    BRLT f3          ; 1 < 1 + C(1)，跳转到f3
    RJMP f1          ; 不会执行
f3:
    RJMP f3
```

### `SBxx`

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230814103112.png)

形如 `SBxx` 的指令根据寄存器中的某一位来选择跳过执行下一条指令， `SBRC` / `SBRS` 指令根据的是通用寄存器中的位的清除/设置状态，而 `SBIC` / `SBIS` 指令根据的是I/O寄存器中的。

例如：

```asm
f1:
    LDI R16, 0xAA
    SBRS R16, 1      ; R16.1 == 1，跳过下一条指令
    RJMP f1          ; 不会执行
    SBIC PINB, 2     ; 如果PB2输入为低电平，跳过下一条指令
    RJMP f1
```

### `BRxx`

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230814103542.png)

形如 `BRxx` 的指令用于根据条件改变程序执行流，支持的条件具体见下表：

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230814103652.png)

`BRxx` 类的指令一般和 `CP` 或 `SUB` 指令配合使用。

例如：

```asm
f1:
    LDI R16, 0X01
    LDI R17, 0X02
    CP R16, R17
    BRLO f2          ; 0x01 < 0x02，跳转到f2
    RJMP f1          ; 不会执行
f2:
    CPI R16, 0x01
    BRSH f3          ; 0x01 == 0x01，跳转到f3
    RJMP f1          ; 不会执行
f3:
    RJMP f3
```

## 参考资料

1. [ATmega328P Datasheet](https://ww1.microchip.com/downloads/en/DeviceDoc/Atmel-7810-Automotive-Microcontrollers-ATmega328P_Datasheet.pdf)
2. [AVR Instruction Set Manual](https://ww1.microchip.com/downloads/en/DeviceDoc/AVR-InstructionSet-Manual-DS40002198.pdf)