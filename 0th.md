# 第零章
## 大小端序
（1）小端：因为低位在低字节，强制转换数据型时不需要再调整字节了。
（2）大端：有符号数，其字节最高位不仅表示数值本身，还起到了符号的作用。符号位固定为第一字节，也就是最高位占据最低地址，符号直接可以取出来，容易判断正负。

小端优势：

    因为在做强制数据类型转换时，如果转换是由低精度转向高精度，这数值本身没什么变化，如 short 是 2 字节，将其转换为 4 字节的 int 类型，无非是由 0x1234 变成了 0x00001234，数值上是不变的，只是存储形式上变了。如果转换是高精度转向低精度，也就是多个字节的数值要减少一些存储字节，这必然是要丢弃一部分数值。编译器的转换原则是强制转换到低精度类型，丢弃数值的高字节位，只保留数值的低字节，

大端优势：

    对于大端的优势，就硬件而言，就是符号位的判断变得方便了。最高位在最低地址，也就是直接就可以取到了，不用再跨越几个字节，减少了时钟周期。另外，对于人类来说，还是大端看上去顺眼，毕竟咱们存储 0x12345678 到内存时，它在内存中的存储顺序也是 0x12345678，而不是 0x78563412，这样看上去才直观。

常见 CPU 的字节序如下。

（1）大端字节序：IBM、Sun、PowerPC。

（2）小端字节序：x86、DEC。

ARM 体系的 CPU 则大小端字节序通吃，具体用哪类字节序由硬件选择。

字节序不仅是在 CPU 访问内存中的概念，而且也包括在文件存储和网络传输中。

bmp 格式的图片就属于小端字节序，而 jpeg 格式的图片则为大端字节序，这没什么可说的，采用什么序列完全是开发者设计产品时的需要。

网络字节序就是大端字节序，所以在 x86 架构上的程序在发送网络数据时，要转换字节顺序。

## BIOS 中断、DOS 中断、Linux 中断的区别
BIOS 和 DOS 都是存在于实模式下的程序，由它们建立的中断调用都是建立在中断向量表（Interrupt 
Vector Table，IVT）中的。它们都是通过软中断指令 int 中断号来调用的。而 Linux 内核是在进入保护模式后才建立中断例程的，不过在保护模式下，中断向量表已经不存在了，
取而代之的是中断描述符表（Interrupt Descriptor Table，IDT）。该表与中断向量表的区别会在讲解中断时详
细介绍。所以在 Linux 下执行的中断调用，访问的中断例程是在中断描述符表中，已不在中断向量表里了。

Linux 的系统调用和 DOS 中断调用类似，不过 Linux 是通过 int 0x80 指令进入一个中断程序后再根据
eax 寄存器的值来调用不同的子功能函数的。再补充一句：如果在实模式下执行 int 指令，会自动去访问
中断向量表。如果在保护模式下执行 int 指令，则会自动访问中断描述符表。

## Section 和 Segment 的区别
`section`:section 称为节，是指在汇编源码中经由关键字 section 或 segment 修饰、逻辑划分的指令或数据区域，
汇编器会将这两个关键字修饰的区域在目标文件中编译成节，也就是说“节”最初诞生于目标文件中。

`segment`:segment 称为段，是链接器根据目标文件中属性相同的多个 section 合并后的 section 集合，这个集合
称为 segment，也就是段，链接器把目标文件链接成可执行文件，因此段最终诞生于可执行文件中。我们
平时所说的可执行程序内存空间中的代码段和数据段就是指segment。
很多时候，我们对segment和section并不作太多区分，但是他俩还是有区别的。
``` shell
    readelf #查看elf文件信息
```
在汇编代码中，若以标准节名定义 section，如我们定义的.bss 便是标准节名。编译器会按照以上说明
中的要求使用 section 内的数据。
不管定义了多少节名，最终要把属性相同的 section，或者编译认为可以放到一块的，合并到一个大的
segment 中，也就是 elf 中说的 program header 中的项。由此可见，某个节（section）属于某个段（segment），
段是由节组成的。

## 魔数
linux程序的基本文件格式为elf，对于elf的header，存在magic number，这里的magic numer主要就是为了方便elf解析器校验文件类型是否是elf的。后面的文件系统也存在类似的东西。

## 操作系统是如何识别文件系统的
对于linux的文件系统诸如ext2 ext4等，其文件系统的大致结构大差不差，识别文件系统主要是通过超级块的魔数，通过魔数表示是什么文件系统。
对于上层的用户程序来说，这些文件系统最终会被统合为一个vfs层。

## 如何控制 CPU 的下一条指令
程序计数器 PC 负责处理器的执行方向，它只是获取下一条指令的方法形式，在不同体系结构的 CPU 中有不同的实现方法。
例如对于x86架构下：

    程序计数器 PC 并不是单一的某种寄存器，它是一种寄存器组合，
    指的段寄存器 CS 和指令指令寄存器 IP。
    
    CS 和 IP 是 CPU 待执行的下一条指令的段基址和段内偏移地址，不能直接用 mov 指令去改变它们，

对于ARM架构下：

    可以用 mov 指令来修改程序流


## 指令集、体系结构、微架构、编程语言

指令集是具体的一套指令编码，微架构是指令集的物理实现方式。

CISC（Complex Instruction Set Computer）
RISC（Reduced Instruction Set Computer）
我们常用的 CPU 是 Intel 和 AMD 公司的产品，它们用的指令集便是基于 CISC 思想的 x86。AMD 的 x86指令架构是 Intel 授权给他们的，为区别于此，Intel 在官方手册上称自己的指令集为 IA32。

在 Intel 的 CPU 上运行的软件也能够在 AMD 的 CPU 上运行，原因就是它们共用了同用一套指令集，也就是对二进制编码达成了共识。

目前市面上常见的指令集有五种，除 x86 是 CISC 指令体系外，ARM、MIPS、Power、C6000 都是RISC 指令体系的指令集。

## MBR、EBR、DBR 和 OBR 各是什么
MBR main boot record 主引导记录。
对于计算机而言，最开始cpu是指向bios的，通过bios启动完成一些简单的初始化或者检测工作，此时是还没有进入操作系统的。

### MBR内容和作用

1. 446字节的引导程序和参数 
2. 64字节的分区表
3. 2字节结束标记0x55和0xaa

其实就是为了从bios手中结果系统的控制权，处理器的使用权。
BIOS指导MBR在0盘0道1扇区，这是约定俗成的，所以他会将MBR引导程序加载到物理地址0x7c00，然后执行。

对于MBR的64字节的分区表而言，每个分区占16个字节，所以一共有4各分区，这四个分区就是次引导程序的候选人群。MBR引导程序遍历这四个分区，然后找到合适的人选将系统控制权交给他。

通常情况下这个“次引导程序”就是操作系统提供的加载器，因此 MBR 引导程序的任务就是把控制权交给操作系统加载器，由该加载器完成操作系统的自举，最终使控制权交付给操作系统内核。

为了让 MBR 知道哪里有操作系统，我们在分区时，如果想在某个分区中安装操作系统，就用分区工具将该分区设置为活动分区，设置活动分区的本质就是把分区表中该分区对应的分区表项中的活动标记为
0x80。MBR 知道`活动分区`意味着该分区中存在操作系统，这也是约定好的。活动分区标记位于分区表项中最开始的 1 字节，其值要么为 0x80，要么为 0，其他值都是非法的。

MBR 在分析分区表时通过辨识“活动分区”的标记 0x80 开始找活动分区，如果找到了，就将 CPU 使用权交给此分区上的引导程序，此引导程序通常是`内核加载器`，下面就直接以它为例。

为了 MBR 方便找到活动分区上的内核加载器，内核加载器的入口地址也必须在固定的位置，这个位置就是各分区最开始的扇区，这也是约定好的。这个`各分区起始的扇区`中存放的是操作系统引导程序—内核加载器，因此该扇区称为操作系统引导扇区，其中的引导程序（内核加载器）称为操作系统引导记录 `OBR`，即 `OS Boot Record`，此扇区也称为 OBR 引导扇区。在 OBR 扇区的前 3 个字节存放了跳转指令，这同样是约定，因此 MBR 找到活动分区后，就大胆主动跳到活动分区 OBR 引导扇区的起始处，该起始处的跳转指令马上将处理器带入操作系统引导程序，从此 MBR 完成了交接工作。

OBR 中开头的跳转指令跳往的目标地址并不固定，这是由所创建的文件系统决定的，对于 FAT32文件系统来说，此跳转指令会跳转到本扇区偏移 0x5A 字节的操作系统引导程序处。不管跳转目标地址是多少，总之那里通常是操作系统的内核加载器。

OBR 是从 DBR 遗留下来的，要想了解 OBR，还是先从了解 DBR 开始。`DBR` 是 `DOS Boot Record`，也就是`DOS 操作系统的引导记录（程序）`，DBR 中的内容大概是：
（1）跳转指令，使 MBR 跳转到引导代码；
（2）厂商信息、DOS 版本信息；
（3）BIOS 参数块 BPB，即 BIOS Parameter Block；
（4）操作系统引导程序；
（5）结束标记 0x55 和 0xaa。

当初为了解决分区数量限制的问题才有了扩展分区，`EBR`是扩展分区中为了兼容 MBR 才提出的概念，主要是兼容 MBR 中的分区表。分区是用分区表来描述的，MBR 中有分区表，扩展分区中的是一个个的逻辑分区，因此扩展分区中也要有分区表，为扩展分区存储分区表的扇区称为 EBR，即 `Expand Boot Record`，从名字上看就知道它是为了“兼容”而“扩展”出来的结构，兼容的内容是分区表，因此它与 MBR 结构相同，只是位置不同，EBR 位于各子扩展分区中最开始的扇区（注意，各主分区和各逻辑分区中最开始的扇区是操作系统引导扇区），理论上 MBR 只有 1 个，EBR 有无数个。


EBR 与 MBR 结构相同，但位置和数量都不同，整个硬盘只有 1 个 MBR，其位于整个硬盘最开始的扇区—0 道 0 道 1 扇区。而 EBR 可有无数个，具体位置取决于扩展分区的分配情况，总之是位于各子扩
展分区最开始的扇区，如果此处不明白子扩展分区是什么，到了以后跟踪分区的章节中大伙儿就会明白。OBR其实就是 DBR，指的都是操作系统引导程序，位于各分区（主分区或逻辑分区）最开始的扇区，访扇区称为操作系统引导扇区，即 OBR 引导扇区。OBR 的数量与分区数有关，等于主分区数加逻辑分区数之和，友情提示：一个子扩展分区中只包含 1 个逻辑分区。

MBR 和 EBR 是分区工具创建维护的，不属于操作系统管理的范围，因此操作系统不可以往里面写东西，注意这里所说的是“不可以”，其实操作系统是有能力读写任何地址的，只是如果这样做的话会破坏“系统控制权接力赛”所使用的数据，下次开机后就无法启动了。OBR 是各分区（主分区或逻辑分区）最开始的扇区，因此属于操作系统管理。

DBR、OBR、MBR、EBR 都包含引导程序，因此它们都称为引导扇区，只要该扇区中存在可执行的程序，该扇区就是可引导扇区。若该扇区位于整个硬盘最开始的扇区，并且以 0x55 和 0xaa 结束，BIOS就认为该扇区中存在 MBR，该扇区就是 MBR 引导扇区。若该扇区位于各分区最开始的扇区，并且以 `0x55`和 `0xaa` 结束，MBR 就认为该扇区中有操作系统引导程序 OBR，该扇区就是 OBR 引导扇区。

DBR、OBR、MBR、EBR 结构中都有引导代码和结束标记 0x55 和 0xaa，因此很多人都容易把它们搞混。不过它们最大的区别是分区表只在 MBR 和 EBR 中存在，DBR 或 OBR 中绝对没有分区表。

大致逻辑：  
1. MBR在0盘0道1扇区，这是约定好的。BIOS就会把MBR引导程序加载到物理地址0x7c00，然后BIOS就会把处理器使用权交给MBR了。  
2. MBR为主引导分区，然后需要找到正确的次引导分区并且家畜系统控制权，次引导分区则存在MBR的分区表内，一个分区大概占16字节，一共有四个分区。这里的次引导分区一般指的是操作系统提供的加载器，用于完成操作系统的自举，加载。这里MBR其实也不知道操作系统在哪。  
3. 而在分区的时候，会存在一个活动分区，这里的活动分区表示该分区内存在操作系统。活动分区标记一般位于分区表项的最开始一个字节，为x80表示有，0表示没有，其他值为非法的。找到了就将cpu的使用权交给此分区上的引导程序。  
4. 该引导程序通常为内核加载器，找到了对应的分区后，内核加载器的入口地址是固定在某个位置的，也就是各分区的最开始的扇区，内核加载器就被称为操作系统引导记录OBR，前三个字节存放了跳转指令，可以跳转到对应的操作系统引导程序，然后就会开始加载内核，然后内核就会正式接管处理器了。

`MBR`:main boot record,bios后面会将使用权交给mbr，通过mbr完成后序内核加载的步骤。  
`DBR`:dos boot record,字如其意， 就是dos系统的引导程序，但是现在dos系统已经退出了历史舞台，现在 DBR 也称为 OBR。  
`OBR`:os boot record，主要是用于操作系统引导加载的，一般来讲，通过OBR后，内核便会正式接管处理器的使用权。  
`EBR`:expand boot record，主要是为了解决分区数量限制的问题，才出现的扩展分区

