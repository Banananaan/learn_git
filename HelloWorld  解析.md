# HelloWorld  解析

​		helloworld是一个经典的demo，也是很多人学习初学某种语言时实现的第一个程序。C语言的helloworld虽然只需几行代码，但是为了实现最终在终端输出“helloworld”，计算机仍需在背后做许多看不见的工作。本文分析了helloworld demo的编译，链接，执行过程，旨在更完整地描述这一demo的实现原理。

```c
#include<stdio.h> 
int main(){
    printf("helloworld!\n");
    return 0;
}
```

```bash
jaja@jaja-host:~/demo$ gcc ./helloworld.c 
jaja@jaja-host:~/demo$ ./a.out 
helloworld!
```

# 1.编译

#include 是一个预处理指令。它表示在编译的时候将<stdio.h>表示的h文件替换到#include<stdio.h>这一行。stdio.h中声明了printf的原型即输入输出形式，编译器可以根据根据函数原型确定调用函数时执行的参数入栈和参数返回动作，另外需要根据函数名称确定符号名用于链接。

我们可以单独编译helloworld.c这个文件：

```bash
jaja@jaja-host:~/demo$ gcc -fno-builtin -c helloworld.c
```

gcc -c命令可以将c文件编译成单独的目标文件而不进行链接步骤，-fno-builtin选项可以防止编译时进行自动的优化（否则这里会将printf函数优化为puts函数）

可以用readelf命令解析编译生成的目标文件。

```bash
jaja@jaja-host:~/demo$ readelf -a  helloworld.o
ELF Header:
  Magic:   7f 45 4c 46 02 01 01 00 00 00 00 00 00 00 00 00 
  Class:                             ELF64
  Data:                              2's complement, little endian
  Version:                           1 (current)
  OS/ABI:                            UNIX - System V
  ABI Version:                       0
  Type:                              REL (Relocatable file)
  Machine:                           Advanced Micro Devices X86-64
  Version:                           0x1
  Entry point address:               0x0
  Start of program headers:          0 (bytes into file)
  Start of section headers:          688 (bytes into file)
  Flags:                             0x0
  Size of this header:               64 (bytes)
  Size of program headers:           0 (bytes)
  Number of program headers:         0
  Size of section headers:           64 (bytes)
  Number of section headers:         13
  Section header string table index: 10

Section Headers:
  [Nr] Name              Type             Address           Offset
       Size              EntSize          Flags  Link  Info  Align
  [ 0]                   NULL             0000000000000000  00000000
       0000000000000000  0000000000000000           0     0     0
  [ 1] .text             PROGBITS         0000000000000000  00000040
       000000000000001a  0000000000000000  AX       0     0     1
  [ 2] .rela.text        RELA             0000000000000000  00000200
       0000000000000030  0000000000000018   I      11     1     8
  [ 3] .data             PROGBITS         0000000000000000  0000005a
       0000000000000000  0000000000000000  WA       0     0     1
  [ 4] .bss              NOBITS           0000000000000000  0000005a
       0000000000000000  0000000000000000  WA       0     0     1
  [ 5] .rodata           PROGBITS         0000000000000000  0000005a
       000000000000000d  0000000000000000   A       0     0     1
  [ 6] .comment          PROGBITS         0000000000000000  00000067
       0000000000000036  0000000000000001  MS       0     0     1
  [ 7] .note.GNU-stack   PROGBITS         0000000000000000  0000009d
       0000000000000000  0000000000000000           0     0     1
  [ 8] .eh_frame         PROGBITS         0000000000000000  000000a0
       0000000000000038  0000000000000000   A       0     0     8
  [ 9] .rela.eh_frame    RELA             0000000000000000  00000230
       0000000000000018  0000000000000018   I      11     8     8
  [10] .shstrtab         STRTAB           0000000000000000  00000248
       0000000000000061  0000000000000000           0     0     1
  [11] .symtab           SYMTAB           0000000000000000  000000d8
       0000000000000108  0000000000000018          12     9     8
  [12] .strtab           STRTAB           0000000000000000  000001e0
       000000000000001a  0000000000000000           0     0     1
Key to Flags:
  W (write), A (alloc), X (execute), M (merge), S (strings), l (large)
  I (info), L (link order), G (group), T (TLS), E (exclude), x (unknown)
  O (extra OS processing required) o (OS specific), p (processor specific)

There are no section groups in this file.

There are no program headers in this file.

Relocation section '.rela.text' at offset 0x200 contains 2 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000000005  00050000000a R_X86_64_32       0000000000000000 .rodata + 0
00000000000f  000a00000002 R_X86_64_PC32     0000000000000000 printf - 4

Relocation section '.rela.eh_frame' at offset 0x230 contains 1 entries:
  Offset          Info           Type           Sym. Value    Sym. Name + Addend
000000000020  000200000002 R_X86_64_PC32     0000000000000000 .text + 0

The decoding of unwind sections for machine type Advanced Micro Devices X86-64 is not currently supported.

Symbol table '.symtab' contains 11 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
     1: 0000000000000000     0 FILE    LOCAL  DEFAULT  ABS helloworld.c
     2: 0000000000000000     0 SECTION LOCAL  DEFAULT    1 
     3: 0000000000000000     0 SECTION LOCAL  DEFAULT    3 
     4: 0000000000000000     0 SECTION LOCAL  DEFAULT    4 
     5: 0000000000000000     0 SECTION LOCAL  DEFAULT    5 
     6: 0000000000000000     0 SECTION LOCAL  DEFAULT    7 
     7: 0000000000000000     0 SECTION LOCAL  DEFAULT    8 
     8: 0000000000000000     0 SECTION LOCAL  DEFAULT    6 
     9: 0000000000000000    26 FUNC    GLOBAL DEFAULT    1 main
    10: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT  UND printf

No version information found in this file.

```

其中可以看到生成的ELF文件内容信息：包括elf头以及各个分段信息。在符号表的尾端有两个需要参与链接的全局符号main和printf，其中printf处于未定义状态（UND）。

# 2链接

可以看出当我们编译helloworld.c文件时，我们实际上定义了main函数的实现，以及确定了printf函数的函数名，传入参数以及返回方式。而printf函数的具体实现则定义在别的目标文件中。

接下来可以观察gcc编译和链接helloworld.c的整个过程

```bash
jaja@jaja-host:~/demo$  gcc -static --verbose -fno-builtin helloworld.c
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/x86_64-linux-gnu/5/lto-wrapper
Target: x86_64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu 5.4.0-6ubuntu1~16.04.12' --with-bugurl=file:///usr/share/doc/gcc-5/README.Bugs --enable-languages=c,ada,c++,java,go,d,fortran,objc,obj-c++ --prefix=/usr --program-suffix=-5 --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --with-sysroot=/ --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-default-libstdcxx-abi=new --enable-gnu-unique-object --disable-vtable-verify --enable-libmpx --enable-plugin --with-system-zlib --disable-browser-plugin --enable-java-awt=gtk --enable-gtk-cairo --with-java-home=/usr/lib/jvm/java-1.5.0-gcj-5-amd64/jre --enable-java-home --with-jvm-root-dir=/usr/lib/jvm/java-1.5.0-gcj-5-amd64 --with-jvm-jar-dir=/usr/lib/jvm-exports/java-1.5.0-gcj-5-amd64 --with-arch-directory=amd64 --with-ecj-jar=/usr/share/java/eclipse-ecj.jar --enable-objc-gc --enable-multiarch --disable-werror --with-arch-32=i686 --with-abi=m64 --with-multilib-list=m32,m64,mx32 --enable-multilib --with-tune=generic --enable-checking=release --build=x86_64-linux-gnu --host=x86_64-linux-gnu --target=x86_64-linux-gnu
Thread model: posix
gcc version 5.4.0 20160609 (Ubuntu 5.4.0-6ubuntu1~16.04.12) 
COLLECT_GCC_OPTIONS='-static' '-v' '-fno-builtin' '-mtune=generic' '-march=x86-64'
 /usr/lib/gcc/x86_64-linux-gnu/5/cc1 -quiet -v -imultiarch x86_64-linux-gnu helloworld.c -quiet -dumpbase helloworld.c -mtune=generic -march=x86-64 -auxbase helloworld -version -fno-builtin -fstack-protector-strong -Wformat -Wformat-security -o /tmp/ccPs6gsK.s
GNU C11 (Ubuntu 5.4.0-6ubuntu1~16.04.12) version 5.4.0 20160609 (x86_64-linux-gnu)
	compiled by GNU C version 5.4.0 20160609, GMP version 6.1.0, MPFR version 3.1.4, MPC version 1.0.3
GGC heuristics: --param ggc-min-expand=100 --param ggc-min-heapsize=131072
ignoring nonexistent directory "/usr/local/include/x86_64-linux-gnu"
ignoring nonexistent directory "/usr/lib/gcc/x86_64-linux-gnu/5/../../../../x86_64-linux-gnu/include"
#include "..." search starts here:
#include <...> search starts here:
 /usr/lib/gcc/x86_64-linux-gnu/5/include
 /usr/local/include
 /usr/lib/gcc/x86_64-linux-gnu/5/include-fixed
 /usr/include/x86_64-linux-gnu
 /usr/include
End of search list.
GNU C11 (Ubuntu 5.4.0-6ubuntu1~16.04.12) version 5.4.0 20160609 (x86_64-linux-gnu)
	compiled by GNU C version 5.4.0 20160609, GMP version 6.1.0, MPFR version 3.1.4, MPC version 1.0.3
GGC heuristics: --param ggc-min-expand=100 --param ggc-min-heapsize=131072
Compiler executable checksum: 8087146d2ee737d238113fb57fabb1f2
COLLECT_GCC_OPTIONS='-static' '-v' '-fno-builtin' '-mtune=generic' '-march=x86-64'
 as -v --64 -o /tmp/cchuELOv.o /tmp/ccPs6gsK.s
GNU assembler version 2.26.1 (x86_64-linux-gnu) using BFD version (GNU Binutils for Ubuntu) 2.26.1
COMPILER_PATH=/usr/lib/gcc/x86_64-linux-gnu/5/:/usr/lib/gcc/x86_64-linux-gnu/5/:/usr/lib/gcc/x86_64-linux-gnu/:/usr/lib/gcc/x86_64-linux-gnu/5/:/usr/lib/gcc/x86_64-linux-gnu/
LIBRARY_PATH=/usr/lib/gcc/x86_64-linux-gnu/5/:/usr/lib/gcc/x86_64-linux-gnu/5/../../../x86_64-linux-gnu/:/usr/lib/gcc/x86_64-linux-gnu/5/../../../../lib/:/lib/x86_64-linux-gnu/:/lib/../lib/:/usr/lib/x86_64-linux-gnu/:/usr/lib/../lib/:/usr/lib/gcc/x86_64-linux-gnu/5/../../../:/lib/:/usr/lib/
COLLECT_GCC_OPTIONS='-static' '-v' '-fno-builtin' '-mtune=generic' '-march=x86-64'
 /usr/lib/gcc/x86_64-linux-gnu/5/collect2 -plugin /usr/lib/gcc/x86_64-linux-gnu/5/liblto_plugin.so -plugin-opt=/usr/lib/gcc/x86_64-linux-gnu/5/lto-wrapper -plugin-opt=-fresolution=/tmp/cc5ZoCoh.res -plugin-opt=-pass-through=-lgcc -plugin-opt=-pass-through=-lgcc_eh -plugin-opt=-pass-through=-lc --sysroot=/ --build-id -m elf_x86_64 --hash-style=gnu --as-needed -static -z relro /usr/lib/gcc/x86_64-linux-gnu/5/../../../x86_64-linux-gnu/crt1.o /usr/lib/gcc/x86_64-linux-gnu/5/../../../x86_64-linux-gnu/crti.o /usr/lib/gcc/x86_64-linux-gnu/5/crtbeginT.o -L/usr/lib/gcc/x86_64-linux-gnu/5 -L/usr/lib/gcc/x86_64-linux-gnu/5/../../../x86_64-linux-gnu -L/usr/lib/gcc/x86_64-linux-gnu/5/../../../../lib -L/lib/x86_64-linux-gnu -L/lib/../lib -L/usr/lib/x86_64-linux-gnu -L/usr/lib/../lib -L/usr/lib/gcc/x86_64-linux-gnu/5/../../.. /tmp/cchuELOv.o --start-group -lgcc -lgcc_eh -lc --end-group /usr/lib/gcc/x86_64-linux-gnu/5/crtend.o /usr/lib/gcc/x86_64-linux-gnu/5/../../../x86_64-linux-gnu/crtn.o

```

在输出结果的末尾展示了最终链接到可执行文件的目标文件：

libgcc.a,libgcc_eh.a,libc.a(注意l是linux下lib的简写),crt1.o,crti.o,crtbeginT.o,cchuELOv.o,crtend.o,crtn.o

其中的lib.a定义了printf函数。另外值得注意的时crt1.o中定义了_start函数，这是gcc生成的可执行文件的默认入口函数，在_main真正执行前，_start需要完成一些初始化的工作比如将main的参数推入栈中。

```bash
Symbol table '.symtab' contains 16 entries:
   Num:    Value          Size Type    Bind   Vis      Ndx Name
     0: 0000000000000000     0 NOTYPE  LOCAL  DEFAULT  UND 
     1: 0000000000000000     0 SECTION LOCAL  DEFAULT    2 
     2: 0000000000000000     0 SECTION LOCAL  DEFAULT    1 
     3: 0000000000000000     0 SECTION LOCAL  DEFAULT    4 
     4: 0000000000000000     0 SECTION LOCAL  DEFAULT    5 
     5: 0000000000000000     0 SECTION LOCAL  DEFAULT    7 
     6: 0000000000000000     0 SECTION LOCAL  DEFAULT    8 
     7: 0000000000000000     0 SECTION LOCAL  DEFAULT    9 
     8: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT  UND __libc_csu_fini
     9: 0000000000000000    42 FUNC    GLOBAL DEFAULT    2 _start
    10: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT  UND __libc_csu_init
    11: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT  UND main
    12: 0000000000000000     0 NOTYPE  WEAK   DEFAULT    7 data_start
    13: 0000000000000000     4 OBJECT  GLOBAL DEFAULT    4 _IO_stdin_used
    14: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT  UND __libc_start_main
    15: 0000000000000000     0 NOTYPE  GLOBAL DEFAULT    7 __data_start

```

gcc最终的生成的可执行文件依然是一个ELF格式文件，只具有print一行函数的helloworld.c最终静态链接生成的可执行文件大小有837370byte，加上text，data，bss外的段达到了912760byte，符号表中的符号数量超过1800个。可以看出在本文介绍的内容之外，链接器以及c运行库还完成了许多工作。

# 3执行

```bash
jaja@jaja-host:~/demo$ gcc ./helloworld.c 
jaja@jaja-host:~/demo$ ./a.out 
helloworld!
```

当我们在终端输入了./a.out后shell会fork一个新的进程，并且在进程中调用一个exec函数，该函数会读入可执行文件的elf头信息获取该文件的类型以及入口地址，接着会触发一个缺页异常因为入口地址所在的那一部分文件并没有读入内存，处罚缺页异常后操作系统会将文件对应的区域读入到内存当中，程序便开始执行起来。

每一个进程会维护自己的打开文件描述符表。进程在初始化后会有3个打开文件描述符：stdin,stdout,stderr。其中stdin 从键盘读取输入，stdout,stderr向终端输出。在不改变程序的打开文件描述符时，printf会向stdout输出“helloworld！”。之后main调用return 0，完成main结束后的收尾工作， 回退到_start中最后结束进程。

