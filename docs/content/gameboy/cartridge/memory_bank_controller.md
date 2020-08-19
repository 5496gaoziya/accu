# Memory Bank Controller

自从计算机被发明以来, 许多计算机系统都不得不面对一个问题: 计算机上所运行的程序需要越来越多的内存, 以至于超出了当时的硬件能力范围. 传统上, 有两种方式可以来处理这个问题:

- 增加 CPU 可寻址的地址空间. 设计一个新的拥有更多地址总线的 CPU, 使其能够寻址更大的内存量. 这是优选的解决方案, 但是需要时间和金钱来解决问题. 比如近几年已经不太常见的 32 位系统, 因为其使用的 32 位 CPU 只能寻址最多 4G 内存, 无论是播放超清视频还是玩大型游戏都显得捉襟见肘.
- 虚拟内存方案. 这可以是映射到软盘上的 RAM, 当 CPU 需要时交换软盘与内存中的数据; 或是已经提前预写的 ROM 分块数据. 在这两种情况下, 系统硬件只需要一点扩展, 成本低廉, 但系统中的任何软件必须知道该虚拟内存体系的存在, 才能使用它.

由于 Game Boy 是一个定价亲民的掌上游戏机, 使用昂贵的 CPU 将极大提升成本, 因此任天堂通过虚拟内存的方式来支持更大的存储空间. Game Boy 的内存管理单元分配给 Cartridge 的内存范围是 0x0000...0x7fff 和 0xa000...0xbfff 两个区间, 总计 0x8000 + 0x2000 = 32k + 8k = 40k 的存储空间, 其中后 8k 被分配给外部内存. 可以得出一个简单的结论是, 如果直接将卡带的物理存储映射到内存管理单元的地址, 卡带的最大容量不得超过 32k. 不过幸运的是, 开发者可以通过内嵌在 Cartridge 中的 Memory Bank Controller(内存存储体控制器, 简称 MBC)技术来扩展游戏的体积.

CPU 通常只能与系统地址空间直接通信(除 I/O 口外), 因此如果一个外部设备期望与 CPU 通信的话, 可以通过将自己的存储空间映射到系统地址空间上来完成. MBC 技术正是位于这层"映射"过程中.

# 原理详解

Memory Bank Controller 是计算机设计中的一种常见技术, 用于将可用存储容量增加到超出处理器可直接寻址的范围. 它最初起源于小型嵌入式系统, 之后在 8 位机上发扬光大, 到现代, 依然可以在许多微机系统中看到它的身影(不过在现代它通常被称为 Bank Switing, 存储体切换, 实际上指代的是同一件事物). MBC 被认为是通过一些外部寄存器扩展处理器地址总线的一种方式, 它的原理异常简单, 例如, 具有 16 位外部地址总线的处理器只能寻址 2^16 = 65536 个存储器位置. 如果将外部锁存器添加到系统, 则可以使用它来控制两组存储设备中的哪一组, 每个存储设备具有 65536 个地址. 处理器可以通过设置或清除锁存器位来更改当前使用的设置, 如图所示, 当 CPU 试图读取系统地址为 0x00001 的值时, 根据锁存器标志位的设置与否, 将从物理地址中实际读取到值 A 或者值 C.

![img](/img/gameboy/cartridge/memory_bank_controller/mbc.png)

由于外部存储体选择锁存器(或寄存器)未直接与处理器的程序计数器连接, 因此当程序计数器溢出时, 它不会自动更改状态. 由于程序计数器是处理器的内部寄存器, 因此外部锁存器无法检测到此错误, 这导致程序无法无缝使用额外的内存. 取而代之的是, 处理器必须明确地执行存储体切换操作, 以访问额外的内存区间. MBC 还有一些其它的限制, 因此它并非是万能的.

MBC 在许多视频游戏机中都有应用. 例如, Atari 2600 只能寻址 4KB 的 ROM, 因此, 后来的 2600 游戏卡带包含其自己的存储体切换硬件, 以便允许卡带存储更多内容, 从而允许进行更复杂的游戏(复杂的游戏通常拥有更多程序代码, 同样拥有更加大量的游戏数据, 例如图形). Nintendo Entertainment System(NES)包含一个改进的 6502 游戏卡带, 其卡带可通过称为多存储控制器的技术手段来切换来寻址, 从某方面来说是 Game Boy 的 MBC 系统的前身. 除了 Game Boy 之外, MBC 系统仍在以后的游戏系统上被广泛使用, 比如几个大小超过了 4MB 的 Sega Mega Drive 盒式磁带, 使用的也是此技术.

通过合理的使用 MBC, 在已知的已生产的游戏卡带中, 容量最大的卡带是一款叫做 Densha de Go! 的游戏, 卡带类型是 MBC5, 支持 8M 存储, 是 32k 的 256 倍. 这款游戏是一款电车模拟器, 如图所示, 玩起来似乎很带感.

![img](/img/gameboy/cartridge/memory_bank_controller/densha_de_go.png)

虽然许多游戏卡带都使用了 MBC 技术, 但是不同种类的卡带其 MBC 规格都有一些细微的差别. 常见的 Game Boy 卡带类型有 ROM Only, MBC1, MBC2, MBC3 和 MBC5 5 个种类, 后续的小节将会对此进行详细介绍.