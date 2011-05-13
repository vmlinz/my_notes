# notes on arm assembly #

学习一种计算机体系结构的最好方法就是学习它的汇编 -- anonymous

## introduction to assembly ##

Liberty means responsibility.  That is why most men dread it.  ~George Bernard Shaw, Man and Superman, "Maxims: Liberty and Equality," 1905

计算机程序语言从底层往上大致可以分为4个类型：机器语言，汇编语言，编译语言(c/c++),解释语言(java,php,python)。越是高级的语言越接近人类的思维，表达能力越强，程序的移植性越好，运行时代价越高；汇编的好处就是让你编写更适合机器，运行更快的代码。汇编语言和机器语言是等价的，它是机器语言的助记符。通过汇编你可以得到cpu的完全控制能力，你可以享受完全的自由。但是请记住，享受自由意味着你需要完全对自由负责。也可以这么认为，使用汇编语言你可以做任何事情，但是你得做所有事情。

### reasons to learn asm ###

- 理解cpu的工作方式
- 知道一些系统性能的瓶颈
- 通过其他语言编译器生成的汇编，理解各种语言的工作与原理
- 了解c语言和汇编语言的交互
- geeky and cool :)

### general assembly ###

汇编是和机器语言一一对应的宏语言。

- cpu一般都可以实现算术操作和位操作。
- 在汇编中你可以使用寄存器，内存块和堆栈来存储和表示变量。
- 跳转和条件，cpu中有一个用于指向下一条指令的pc寄存器，可以通过改写该寄存器实现控制cpu的执行流。当然通过条件控制的跳转才是最强力的。arm中的cpsr(current processor state register)中的控制段中有一些用于控制跳转的条件码。
- arm中汇编和c语言相互调用时需要遵循AAPCS，这个规定了调用时参数传递，堆栈和寄存器使用的标准。这样你写的代码就可以和编译器生成的代码和谐相处了。

## introduction to arm ##

arm是由arm公司开发的一个32位的精简指令集架构。arm在嵌入式和移动设备中占主导地位。arm公司是通过授权arm核心生产来盈利的，这是一种与传统生产链完全不同的生产方式。从上游设计到下游的生产各个部分都由不同的公司协作完成。这极大的提高了他们的生产效率。

arm得到了大多数硬件厂商的支持，特别是移动设备厂商。arm的主要特性是精简指令集以及低功耗。

## arm instructions ##

arm核支持两个指令集：arm和thumb（很形象，arm是手臂，thumb是手指。这就知道两种指令集的大体区别了）。arm指令一般有涉及3个寄存器，目标寄存器和两个操作寄存器。arm指令是定长指令（都是32位），可以简化指令的编解码。这也解释了arm中的立即数只有一个byte长度。thumb指令则是16位长度，使用thumb指令可以极大的提高指令密集度。thumb指令可以看作arm指令的子集，学会arm指令之后再了解thumb指令中什么不能做就很简单了。

### basic features ###

- arm registers, r0-r15, refer to arm instruction mannual to get detailed information. r12(IP), r13(SP), r14(LR), r15(PC).
- arm 指令分类：数据操作，内存操作，分支和跳转操作
- arm 指令都支持条件跳转
- arm 位移操作，arm 指令的第二个操作数寄存器支持位移操作(lsl, lsr, asr, ror)，位移的开销很小，灵活使用可以提高性能和速度
- 受限的立即数，arm指令中的立即数位数很少一共有12位，8数据位，4个位移位。arm 能表示的立即数必须是8位的偶数位移数，即数中最多有8个bit有效。v=n ror 2*r(v为立即数，n为8位位图数，r为位移位)。要载入更大的数，需要使用伪指令ldr rd, =dw。

### data instructions ###

arm 中用于执行算术和逻辑操作的指令。arm 和其他体系结构不同，它只能在寄存器上处理数据，处理内存数据要先载入到寄存器上。

- 指令结构 `op{cond}{s} Rd, Rn, Op2 cond`和s是指令后缀，rd是目标寄存器，rn是第一操作寄存器，op2是第二操作数，arm 指令中最灵活的地方就是第二操作数。
- 算术操作 add, adc, sub, sbc, rsb, rsc
- 位操作 and, bic, eor, mov, mvn, orr
- 状态操作 cmp, cmn, teq, tst 他们可以设置cpsr中的某些位
- 乘法操作 mla, mul, smlal

### memory instructions ###

- 内存操作分为两种 str(保存寄存器到内存)和ldr(加载内存到寄存器)
- 指令结构 `op{cond}{type} Rd, [Rn, Op2]` type是指定传输的内存单位，比如字，半子，字节等。

#### 寻址模式 ####

- 寄存器简介寻址 前(后)索引寄存器寻址 `[Rn]` `[Rn, Op2]`
- pc 寄存器相对寻址 通过计算内存地址相对当前pc寄存器位置来访问内存 `ldr R0 =0x1110`
- 回写模式 `ldr Rd, [Rn, Op2]!` `ldr Rd, [Rn], Op2`

#### 块传输 ####

- 指令结构 `op{cond}{mode} Rd{!}, {Rlist}`
- ldm stm 保存和加载寄存器
- mode分为IA IB DA DB 控制索引增加减少，索引变化和访问内存的先后顺序
- 堆栈操作ED EI FD FI AAPCS 中使用FD 这种堆栈(满递减)，stmfd和ldmfd相当于使用stmdb和ldmia来操作堆栈
- `ldmia r0, {r4-r7}` `ldmfd r0 {r4-r7}`

### branching ###

arm 主要有3个跳转指令

- b 简单的段内跳转
- bl 函数调用跳转
- bx 切换模式跳转 跳转的同时切换arm 指令模式(arm thumb)
- gcc 中使用`.L`前缀来标示内部标签，函数或其他有具体意义的符号则不用前缀

arm 有4个状态标志，保存在cpsr和spsr中。

- Z 操作结果为0
- N 负数
- C 进位(unsigned overflow)
- V 溢出(signed overflow) 两个大的正数相加溢出成负数

函数调用，实践中你需要遵从AAPCS 来使你写的代码能很好的和遵从该标准的编译器生成的代码正常交互。

- 前4个参数用r0-r3来存放，其他的参数按出现顺序入栈
- 返回值保存到r0
- r0-r3和r12这样的临时寄存器在调用过程中是随时变化的，可以随便使用，也不用保存
- 其他寄存器则需要在函数调用前后保持一致

## thumb instructions ##

thumb 指令更短，只有16位。在通过16位bus读取的时候可以直接执行，32位指令则需要等到获取到指令的第二半之后才能执行。thumb 指令和arm 指令很类似，学会arm 指令之后，你只要知道thumb 不能做什么就可以了。接下来就是thumb 指令和arm 指令的区别，

- 有些指令不存在 比如，rsb
- 新指令 pop，push
- 没有条件码，除了跳转(b)指令
- 受限的寄存器访问 一般情况thumb 只能访问r0-r7
- 设置状态一直开启 执行sub就相当于subs，它会设置状态标志
- 位移指令被单独取出来，不能在直接结合Op2使用了
- 没有立即数和Op2 有例外，请参考指令手册
- 没有寄存器回写模式 内存访问指令没有回写模式了

## gnu assembler ##

arm cpu指令只是arm 汇编功能的一部分，你还需要汇编器指令来控制代码段，对齐以及创建变量等等。汇编器指令通常和特定的汇编器相关，一般不可移植。本节主要介绍一些GNU AS的汇编器指令，要更详细的资料，请参考GNU AS的手册。

### symbols ###

数据和函数被统称为符号，和c 一样，它们有声明和定义。标签很可能是一个符号。最好把局部的符号和全局的符号区别开，`.global lname`将符号声明为全局，这样其他文件就可以访问这个符号了。按照惯例，局部标签由`.L`开始。要使用外部符号，需要使用`.extern lname`声明符号是外部定义的(相当于c 语言的`extern`关键字)。

- `.type, str` 用来说明标签的类型，比如函数(str=%function)
- `.code n` 用来说明这段标签的指令类型是arm 还是thumb
- `.thumb_func` 指定下一个标签的thumb 函数可以和c互相连接
- `.align n` 让下一个指令对齐到2^n地址
- `.extern lname` 声明符号为外部定义的符号
- `.global lname` 声明符号为全局符号

GAS默认未明确标示的符号为外部符号，这点和c 语言默认全局符号一样，破坏了代码的局部性。

### definition of variables ###

数据定义的主要汇编器指令, `.byte` `.word` `.hword` `.float` `.string`

### data sections ###

数据段定义主要用来给连接器提供信息，让它根据链接脚本将数据放在合适的位置， `.text` `.data` `.section secname, "flags", %type` `.bss`

了解这些汇编器指令之后你就可以创建函数，声明定义变量甚至还可以把它们放在合适的位置，可以应付多数情况了。余下的指令你只要参考GAS的手册或者去读一些GCC生成的汇编代码就可以解决了。

## resources ##

- [arm architecture](http://en.wikipedia.org/wiki/ARM_architecture)
- [tour of arm asm](http://www.coranac.com/tonc/text/asm.htm)
- [misc arm assembly](http://www.heyrick.co.uk/assembler/)
- [resources to learn arm asm](http://stackoverflow.com/questions/270078/resources-for-learning-arm-assembly)
- [eabi introduction on debian wiki](http://wiki.debian.org/ArmEabiPort#In_a_nutshell)
- [arm faq](http://www.codesourcery.com/sgpp/lite/arm/portal/target_arch?@action=faq&target_arch=arm)
- [arm docs](http://infocenter.arm.com/help/index.jsp)
