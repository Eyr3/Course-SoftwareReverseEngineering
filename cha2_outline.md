# x86&x64体系结构
## x86&x64基本概念
1. x86：intel体系结构的32位实现
- 基于Intel8086处理器的小端序体系结构
- 主要操作模式：实模式（16位指令集）、保护模式（虚拟内存、分页）、系统管理模式（执行嵌入在固件中的特殊代码）

2. x64：x86-64，x86体系结构的扩展，与x8664位CPU兼容的体系结构

3. 字节序： 

小端序效率高

|type&size|变量|大端序|小端序|
|:---:|:----:|:------:|:------:|
|DWORD&4|dw=0x12345678|[12][34][56][78]地址：左低右高|[78][56][34][12]|
|char[ ]&6|str="abcde"|[61][62][63][64][65][00]|同大端序|
4. 保护环

- x86对优先权分隔的支持机制
- ring0~ring3
- ring0：优先权最高-内核；ring3：用户态应用

## IA-32内存模型与内存管理
1. 内存模型：
- 平面内存模型：代码、数据、栈在线性地址空间0~2^31 - 1
- 分段内存模型：代码/数据/栈 段，逻辑地址=段选择器+偏移量
![内存分段.png](https://i.loli.net/2017/10/26/59f152aae7530.png)
    - 实模式？
    - 保护模式？

## IA-32寄存器集合
1. GPR 通用寄存器
- 所有Win32API函数会先将返回值保存在EAX中再返回
- Win32 API函数内部会使用ECX EDX，在编写汇编程序调用Win32 API之前应先将其内容备份至其他寄存器/栈
2. EFLAGS 程序状态与控制寄存器
- 主要用于实现条件分支
3. EIP 指令指针寄存器
- 存放指令指针，即当前代码段中将被执行的下一条指令的线性地址偏移
- 不能直接修改。JMP, Jcc, CALL, RET；中断/异常
4. 段寄存器
- CS SS DS ES/FS/GS

## IA-32指令集
1. MOV数据移动指令
![MOV.png](https://i.loli.net/2017/10/26/59f1948767811.png)
- MOVSB/MOVSW/MOVSD：在2内存地址之间移动1/2/4字节
- 使用EDI保存目标地址；使用ESI保存源地址
- 源/目标地址会自动更新，根据EFLAGS中的DF标识决定是自增（DF=1）还是自减（DF=0）
- 有时可配合REP指令（将指令操作重复ECX次）使用
![SCAS.png](https://i.loli.net/2017/10/26/59f197c0b997c.png)

2. 栈
连续内存区域 包含在一个栈段内，可由SS寄存器所指
- 栈帧：
    - 联系调用函数、被调用函数的机制
    - 栈被分割为栈帧，栈帧组成栈
    - 包括：函数局部变量、向被调用函数传递的参数、函数调用的联系信息（栈帧相关的指针）
        - 栈帧基指针：被调用函数确定的一个参考点
        - 返回指令指针：被调用函数返回后首先执行的调用函数指令的地址，即调用函数中CALL指令的下一条指令的地址
3. CALL
- CALL指令执行：
    1. 将返回指令指针（CALL指令的下一条指令的地址，也是EIP的当前值）压栈
    2. 将CALL的目标指令地址（被调用函数的第一条指令的地址）载入EIP
- CALL执行完后：
    1. 开始执行被调用函数，执行PUSH EBP，将调用函数的栈帧的EBP压栈
    2. ESP -> EBP，确立新栈帧（被调用函数栈帧）的基址针
    3. 执行被调用函数的具体功能
4. RETN
- RETN执行之前：
    1. EBP -> ESP，清空当前被调用函数的栈帧（此时ESP指向的栈顶内容恰好为调用者函数的EBP）(可选)
    2. 执行POP EBP，将EBP恢复为调用者函数的原始EBP
- RETN指令执行：
    1. 将栈顶的内容（返回指令指针）弹出到EIP
    2. 若RETN指令有参数n，则将栈顶指针ESP增加n字节，从而释放栈上的参数
    3. 恢复对调用函数的执行
5. 调用惯例
- cdecl：C语言中使用，参数从右到左压栈，调用者清理栈
- stdcall：常用于Win32 API，被调用者清理栈
- fastcall：类似于stdcall，但使用寄存器ECX、EDX传递函数的前2个参数
6. 控制流指令
    1. CMP：
    比较两个操作数（通过相减）,并设置EFLAGS中的适当条件码
    2. TEST：
    比较两操作数（逻辑AND），设置EFLAGS中的适当条件码
    3. JMP：
    无条件跳转至目标指令
    4. Jcc cc=conditional code
    ![Jcc.png](http://reclass-1252703453.coscd.myqcloud.com/Jcc.png?sign=qVktz8/LQ5N7afo2qSwdr/sXDCNhPTEyNTI3MDM0NTMmaz1BS0lEOFNyVDgxV085dnBzeDJJZHpMakFsT09icTZQN2FZQnQmZT0xNTExNjAwOTI4JnQ9MTUwOTAwODkyOCZyPTEyMDkxODA4NTYmZj0vSmNjLnBuZyZiPXJlY2xhc3M=)
    5. if_else
    ![if_else.png](http://reclass-1252703453.coscd.myqcloud.com/if_else.png?sign=7Ug5lt2qO1MrbN3zE0bkPtQ/xmFhPTEyNTI3MDM0NTMmaz1BS0lEOFNyVDgxV085dnBzeDJJZHpMakFsT09icTZQN2FZQnQmZT0xNTExNjAwOTI4JnQ9MTUwOTAwODkyOCZyPTE2OTQ5ODc0MzQmZj0vaWZfZWxzZS5wbmcmYj1yZWNsYXNz)
    6. switch_case
    ![switch_case.png](http://reclass-1252703453.coscd.myqcloud.com/switch_case.png?sign=vpM0rn7ZlAYUN+SpPP+c2/e1o6BhPTEyNTI3MDM0NTMmaz1BS0lEOFNyVDgxV085dnBzeDJJZHpMakFsT09icTZQN2FZQnQmZT0xNTExNjAwOTI4JnQ9MTUwOTAwODkyOCZyPTk0NjI1NzU1MSZmPS9zd2l0Y2hfY2FzZS5wbmcmYj1yZWNsYXNz)
    7. loop
    ![loop.png](http://reclass-1252703453.coscd.myqcloud.com/loop.png?sign=pGAUL92KIKkwBNvrNRnOs8caNKdhPTEyNTI3MDM0NTMmaz1BS0lEOFNyVDgxV085dnBzeDJJZHpMakFsT09icTZQN2FZQnQmZT0xNTExNjAwOTI4JnQ9MTUwOTAwODkyOCZyPTIyMjUwNDUwMSZmPS9sb29wLnBuZyZiPXJlY2xhc3M=)

2. LEA load effective address，不去读地址上的内容
### OllyDBG 用法说明
![OllyDBG.png](https://i.loli.net/2017/10/26/59f18de638d5f.png)