# Homework 2
<p align=right><b>宋林恺 PB21051079</b></p>
<p align=right><b>罗胤玻 PB21111627</b></p>

## T1

## T2

### a

salt时一个随机数，在hash前与password链接。所以不同的密码即使相同，hash值也不同。这样即使两个用户有相同的密码，hash值也不同。这样使得攻击者难以使用rainbow table破解密码。

### b

增加salt的位数可以减少hash碰撞的概率这样攻击者需要更多次的尝试才能还原可能的plain text用于通过password验证，所以提高密码的安全性。

## T3


### a

子进程继承父进程的user id 也为x。

### b

1.euid=y>0, suid=m, ruid=m, 此时没有root权限。 setuid(newuid) 的newuid是m (可以等于0)或y, 将euid设为newuid否则错误。

2.euid=0, suid=m, ruid=m, 此时有root权限。将euid, ruid, suid均设置为setuid(newuid)中newuid的值。

### c

1. 通过分配不同的uid，可以实现不同应用之间的隔离，避免应用之间的相互干扰。且防止一个应用窃取或破坏另一个应用的数据。
2. 通过分配不同的uid，可以实现不同应用之间的权限控制，避免应用之间的权限泄露。并且可以细粒度的控制应用的权限。
3. 通过分配不同的uid，可以实现不同应用之间的资源隔离，避免应用之间的资源争夺。且防止一个应用独占资源。

### d

1. 新进程的uid为root (0)，因为zygote进程是以root权限运行的新创建的进程将继承zygote的uid。
2. 重设uid是为了降低进程的权限，避免进程以root权限运行，提高系统的安全性。这样即使进程被攻击，或者该进程是恶意程序，也因权限较低仅有访问该进程资源的权限，而无法对系统进行破坏。

### e

1. passwd程序的setuid位应该设置为root，这样用户可以通过passwd程序修改自己的密码，因为密码只能被拥有root权限的程序修改程序权限为root则可以修改密码。
2. 因为passwd程序以root权限运行，对于系统资源的访问几乎不受限，防止被恶意程序compromise或者自身代码有错对系统破坏所以应该仔细编写。

## T4

### a

首先恶意文件创建一个符号链接`file.data`但不指向任何文件该代码并不会return。在该代码休眠时，将符号链接指向如`/etc/passwd`非常重要且只能被拥有root权限更改的文件。那么该代码会将`Hello world`写入指向的文件，造成该文件被修改。

### b

并不能完全避免，因为用户程序并不能保证在`stat("f./ile.data", buf)`和`fopen("file.data)`之间恶意程序不能修改。因为1.用户程序的操作并不是原子的恶意程序仍然有时间改变符号链接。2.用户程序可能休眠，在此期间恶意程序有时间改变符号链接。

### c

在Linux中可以加入写保护，如可以在操作前将`file.data`所有者和组设为root，非所有者仅能rx不能w。这样恶意程序则不能在`stat("f./ile.data", buf)`和`fopen("file.data)`之间修改符号链接。完成操作后再重新设为可写。

## T5

### a

1. 优点： 通过提供更多的模式，可以更细粒度的控制进程的权限，提高系统的安全性。更好的控制进程对系统资源的访问，避免进程对系统资源的滥用。
2. 缺点：提供更多模式的安全会增加系统的复杂性，另外因为需要更频繁地切换privilege的模式，将会引入更多的开销（如上下文切换中时间的开销以及每种模式都需要有独立堆栈造成的内存开销），降低系统的性能。

### b
特权指令和 VAX/VMS 相似：

- Kernel: Executes the kernel of the VMS operating system, which includes mem-
ory management, process management
- IO: Executes I/O operations and interrupt handling, such as device drivers
- Executive: Executes many of the operating system service calls, including file and
record (disk and tape) management routines
- Supervisor: Executes other operating system services, such as responses to user
commands
- User: Executes user programs, plus utilities such as compilers, editors, linkers,
and debuggers

不过将IO以及interrupt分开，再kernel模块中仅保留操作系统最核心的内存管理和进程管理。其余如IO和中断处理单独分为IO模式。

## T6

## T7

## Contribution
**宋林恺：**
**罗胤玻：**