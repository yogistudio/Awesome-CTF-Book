# 从任意地址写到控制执行流的方法总结

## 0x00 前言

任意地址写（Arbitrary Address Write, AAW），是指在漏洞利用过程中，攻击者拥有对任意指定地址上的内容进行修改的能力。

产生任意地址写漏洞的原因可能很多，如栈指针溢出、格式化字符串漏洞、UNLINK等等，但是从任意地址写到控制执行流，往往还有很长的路要走。以下是一些常用套路的总结。

## 0x01 GOT表覆写

这是最常见，也是最简单的一种。

通过覆写动态链接程序中，某个函数的GOT表，就能控制对应函数的指向，使这个函数在被调用的时候，运行我们覆写的地址处的代码。

关于GOT、PLT和延迟绑定技术，参考：《[通过 GDB 调试理解 GOT/PLT](http://rickgray.me/2015/08/07/use-gdb-to-study-got-and-plt.html)》

## 0x02 .fini\_array段覆写

在程序中，也许在漏洞产生后就不再有库函数的调用，因此GOT覆写就没有意义了。此时可以通过覆写.fini\_array来控制执行流。

.fini\_array是ELF程序中一个特殊的段，它存放着在程序即将结束时需要执行的一系列代码的地址。因此修改了这个数组，就能在程序结束的时候得到一次代码执行的机会。

## 0x03 覆盖函数指针

这也是比较简单的一种方法。一般是覆盖全局结构体中的函数指针，或者覆盖类的虚表中的函数指针，调用的时候就能执行我们写入的代码。

## 0x04 覆写栈上的返回地址

由于栈中存放着返回地址，因此可以通过覆写返回地址来控制执行流。

但是由于任意地址写一般和栈溢出不同，栈溢出只要考虑返回地址和溢出点的相对位置，而任意地址写漏洞往往需要知道地址的确定值，因此在通过任意地址写漏洞来覆写返回地址的时候，一般需要先泄露一个栈上的地址，得到栈在内存空间中的位置，通过计算得到返回地址所在的内存地址。

可以通过调试，来得到返回地址和泄露的地址之间的偏移。当每次启动程序，执行的指令相同，虽然栈被分布在不同的内存页上，但栈中的相对位置往往是不变的。所以可以通过偏移来计算返回地址的位置，覆写即可。

## 0x05 覆写\_\_free\_hook

当开启RELRO或PIE保护时，ELF程序部分中很难利用，甚至连地址都难以获得，只能通过libc做文章。

只要先泄露libc的基址，然后覆写libc中的\_\_free\_hook就能对free函数进行劫持。


