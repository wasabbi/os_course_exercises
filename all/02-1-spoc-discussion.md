# lec 3 SPOC Discussion

## **提前准备**
（请在上课前完成）


 - 完成lec3的视频学习和提交对应的在线练习
 - git pull ucore_os_lab, v9_cpu, os_course_spoc_exercises  　in github repos。这样可以在本机上完成课堂练习。
 - 仔细观察自己使用的计算机的启动过程和linux/ucore操作系统运行后的情况。搜索“80386　开机　启动”
 - 了解控制流，异常控制流，函数调用,中断，异常(故障)，系统调用（陷阱）,切换，用户态（用户模式），内核态（内核模式）等基本概念。思考一下这些基本概念在linux, ucore, v9-cpu中的os*.c中是如何具体体现的。
 - 思考为什么操作系统需要处理中断，异常，系统调用。这些是必须要有的吗？有哪些好处？有哪些不好的地方？
 - 了解在PC机上有啥中断和异常。搜索“80386　中断　异常”
 - 安装好ucore实验环境，能够编译运行lab8的answer
 - 了解Linux和ucore有哪些系统调用。搜索“linux 系统调用", 搜索lab8中的syscall关键字相关内容。在linux下执行命令: ```man syscalls```
 - 会使用linux中的命令:objdump，nm，file, strace，man, 了解这些命令的用途。
 - 了解如何OS是如何实现中断，异常，或系统调用的。会使用v9-cpu的dis,xc, xem命令（包括启动参数），分析v9-cpu中的os0.c, os2.c，了解与异常，中断，系统调用相关的os设计实现。阅读v9-cpu中的cpu.md文档，了解汇编指令的类型和含义等，了解v9-cpu的细节。
 - 在piazza上就lec3学习中不理解问题进行提问。

## 第三讲 启动、中断、异常和系统调用-思考题
(有些题参考了2018年的答案)
## 3.1 BIOS
-  x86中BIOS从磁盘读入的第一个扇区是是什么内容？为什么没有直接读入操作系统内核映像？
第一个扇区, 即主引导扇区(Master Boot Record)的内容包括加载程序和扇区表信息. 读内核之前应该先做一些准备操作: 初始化环境(寄存器等), 使能保护模式&段机制 等.
- 比较UEFI和BIOS的区别。
参考了: https://www.zhihu.com/question/21672895
BIOS启动流程：系统开机 - 上电自检（Power On Self Test 或 POST）。POST过后初始化用于启动的硬件（磁盘、键盘控制器等）。BIOS会运行BIOS磁盘启动顺序中第一个磁盘的首440bytes（MBR启动代码区域）内的代码。启动引导代码从BIOS获得控制权，然后引导启动下一阶段的代码（如果有的话）（一般是系统的启动引导代码）。再次被启动的代码（二阶段代码）（即启动引导）会查阅支持和配置文件。根据配置文件中的信息，启动引导程序会将内核和initramfs文件载入系统的RAM中，然后开始启动内核。 
UEFI启动流程：系统开机 - 上电自检（Power On Self Test 或 POST）。UEFI 固件被加载，并由它初始化启动要用的硬件。固件读取其引导管理器以确定从何处（比如，从哪个硬盘及分区）加载哪个 UEFI 应用。固件按照引导管理器中的启动项目，加载UEFI 应用。已启动的 UEFI 应用还可以启动其他应用（对应于 UEFI shell 或 rEFInd 之类的引导管理器的情况）或者启动内核及initramfs（对应于GRUB之类引导器的情况），这取决于 UEFI 应用的配置。
简单说, BIOS主要通过MBR启动代码区域执行. UEFI主要通过引导管理器执行.

- 理解rcore中的Berkeley BootLoader (BBL)的功能。
参考了: https://www.lowrisc.org/docs/untether-v0.2/bootload/
"The major function of the BBL is to initialize all peripherals, set up the page table and virtual memory, load the Linux kernel from SD to virtual memory, and finally boot the kernel.
During the kernel execution, BBL continues running underneath the kernel as a hypervisor, serving all peripheral requests from Linux using the actual FPGA hardware."
简单说
1)初始化外围设备 
2)设置页表, 虚拟内存
3)从SD加载Linux内核到虚拟内存
4)启动内核

## 3.2 系统启动流程

- x86中分区引导扇区的结束标志是什么？
0X55AA

- x86中在UEFI中的可信启动有什么作用？
通过数字签名检查保证启动戒指的安全性

- RV中BBL的启动过程大致包括哪些内容？
1)初始化外围设备 
2)设置页表, 虚拟内存
3)从SD加载Linux内核到虚拟内存
4)启动内核


## 3.3 中断、异常和系统调用比较
- 什么是中断、异常和系统调用？
中断: 由CPU外部设备引起的外部事件如I/O中断、时钟中断、控制台中断等是异步产生的（即产生的时刻不确定），与CPU的执行无关，我们称之为异步中断(asynchronous interrupt)也称外部中断,简称中断(interrupt)
异常: 而把在CPU执行指令期间检测到不正常的或非法的条件(如除零错、地址访问越界)所引起的内部事件称作同步中断(synchronous interrupt)，也称内部中断，简称异常(exception)
系统调用: 把在程序中使用请求系统服务的系统调用而引发的事件，称作陷入中断(trap interrupt)，也称软中断(soft interrupt)，系统调用(system call)简称trap(陷阱)


-  中断、异常和系统调用的处理流程有什么异同？
相同: 都会进入异常服务例程，切换为内核态。
不同: 源头不同，中断源是外部设备，异常和系统调用源是应用程序；响应方式不同，中断是异步的，异常是同步的，系统调用异步和同步都可以。

- 以ucore/rcore lab8的answer为例，ucore的系统调用有哪些？大致的功能分类有哪些？
进程管理：包括 fork/exit/wait/exec/yield/kill/getpid/sleep
文件操作：包括 open/close/read/write/seek/fstat/fsync/getcwd/getdirentry/dup
内存管理：pgdir命令
外设输出：putc命令

## 3.4 linux系统调用分析
- 通过分析[lab1_ex0](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex0.md)了解Linux应用的系统调用编写和含义。(仅实践，不用回答)
- 通过调试[lab1_ex1](https://github.com/chyyuu/ucore_lab/blob/master/related_info/lab1/lab1-ex1.md)了解Linux应用的系统调用执行过程。(仅实践，不用回答)


## 3.5 ucore/rcore系统调用分析 （扩展练习，可选）
(完全参考了2018年答案)
-  基于实验八的代码分析ucore的系统调用实现，说明指定系统调用的参数和返回值的传递方式和存放位置信息，以及内核中的系统调用功能实现函数。
在 ucore 中，执行系统调用前，需要将系统调用的参数出存在寄存器中。
eax表示系统调用类型，其余参数依次存在 edx, ecx, ebx, edi, esi 中。

```c
...
int num = tf->tf_regs.reg_eax;
...
arg[0] = tf->tf_regs.reg_edx;
arg[1] = tf->tf_regs.reg_ecx;
arg[2] = tf->tf_regs.reg_ebx;
arg[3] = tf->tf_regs.reg_edi;
arg[4] = tf->tf_regs.reg_esi;
...
```

- 以ucore/rcore lab8的answer为例，分析ucore 应用的系统调用编写和含义。
```c
syscall(int num, ...) {
    va_list ap;
    va_start(ap, num);
    uint32_t a[MAX_ARGS];
    int i, ret;
    for (i = 0; i < MAX_ARGS; i ++) {
        a[i] = va_arg(ap, uint32_t);
    }
    va_end(ap);

    asm volatile (
        "int %1;"
        : "=a" (ret)
        : "i" (T_SYSCALL),
          "a" (num),
          "d" (a[0]),
          "c" (a[1]),
          "b" (a[2]),
          "D" (a[3]),
          "S" (a[4])
        : "cc", "memory");
    return ret;
}
```
这段代码是用户态的syscall函数，其中num参数为系统调用号，该函数将参数准备好后，通过 `SYSCALL` 汇编指令进行系统调用，进入内核态，返回值放在 `eax` 寄存器，传入参数通过 `eax` ~ `esi` 依次传递进去。
在内核态中，首先进入 `trap()` 函数，然后调用 `trap_dispatch()`进入中断分发，当系统得知该中断为系统调用后，OS调用如下的 `syscall` 函数：

```c
void
syscall(void) {
    struct trapframe *tf = current->tf;
    uint32_t arg[5];
    int num = tf->tf_regs.reg_eax;
    if (num >= 0 && num < NUM_SYSCALLS) {
        if (syscalls[num] != NULL) {
            arg[0] = tf->tf_regs.reg_edx;
            arg[1] = tf->tf_regs.reg_ecx;
            arg[2] = tf->tf_regs.reg_ebx;
            arg[3] = tf->tf_regs.reg_edi;
            arg[4] = tf->tf_regs.reg_esi;
            tf->tf_regs.reg_eax = syscalls[num](arg);
            return ;
        }
    }
    print_trapframe(tf);
    panic("undefined syscall %d, pid = %d, name = %s.\n",
            num, current->pid, current->name);
}
```

该函数得到系统调用号 `num = tf->tf_regs.reg_eax;`，通过计算快速跳转到相应的 `sys_` 开头的函数，最终在内核态中，完成系统调用所需要的功能。

- 以ucore/rcore lab8的answer为例，尝试修改并运行ucore OS kernel代码，使其具有类似Linux应用工具`strace`的功能，即能够显示出应用程序发出的系统调用，从而可以分析ucore应用的系统调用执行过程。
利用 `trap.c` 的 `trap_in_kernel()` 函数判断是否是用户态的系统调用，调用 `syscall()` 时传入此参数

```c
    case T_SYSCALL:
        syscall(trap_in_kernel(tf));
        break;
```

更改 `syscall()` 的函数原型为

```c
    void syscall(bool);
```

之后在 `syscall(bool)` 中加入输出即可

```c
    int num = tf->tf_regs.reg_eax;
    if (num >= 0 && num < NUM_SYSCALLS) {
        if (syscalls[num] != NULL) {
            arg[0] = tf->tf_regs.reg_edx;
            arg[1] = tf->tf_regs.reg_ecx;
            arg[2] = tf->tf_regs.reg_ebx;
            arg[3] = tf->tf_regs.reg_edi;
            arg[4] = tf->tf_regs.reg_esi;

	    if (!in_kernel) {
	    	cprintf("SYSCALL: %d\n", num);
	    }
            tf->tf_regs.reg_eax = syscalls[num](arg);
            return ;
        }
    }
```

下面是qemu运行的输出结果片段，可以看出在用户程序输出前调用了 `SYS_open`，输出 `sh is running` 的过程中调用了 `SYS_write`

```
Iter 1, No.0 philosopher_sema is thinking
kernel_execve: pid = 2, name = "sh".
SYSCALL: 100
SYSCALL: 100
SYSCALL: 103
....
```

## 3.6 请分析函数调用和系统调用的区别
- 系统调用与函数调用的区别是什么？
首先指令有区别: 系统调用是int, iret; 函数调用时call, ret
安全上区别: 系统调用有堆栈和特权级的转换过程，函数调用没有这样的过程，系统调用相对更为安全.
性能上区别: 系统调用比函数调用要做更多和特权级切换的工作，所以需要更多的时间开销

- 通过分析x86中函数调用规范以及`int`、`iret`、`call`和`ret`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？
首先系统调用的压栈是伴随特权级切换, 但函数调用的压栈是没有特权级切换. 并且函数调用时只输入参数, 返回地址, old ebp, 但系统调用时还输入一些其他内容:EFLAGS, CS, EIP等等.

- 通过分析RV中函数调用规范以及`ecall`、`eret`、`jal`和`jalr`的指令准确功能和调用代码，比较x86中函数调用与系统调用的堆栈操作有什么不同？
//参考了: https://github.com/shzhxh/v9-doc/blob/master/composition-principle/RISC-V.md
"ecall, eret分别是向更高特权级发起请求(用于产生对执行环境的请求), 返回到自陷产生的特权级. jal, jalr是无条件跳转. "
通过上面的内容, 可以发现系统调用是需要特权级转换, 函数调用是没有特权级转换.

## 课堂实践 （在课堂上根据老师安排完成，课后不用做）
### 练习一
通过静态代码分析，举例描述ucore/rcore键盘输入中断的响应过程。

### 练习二
通过静态代码分析，举例描述ucore/rcore系统调用过程，及调用参数和返回值的传递方法。
