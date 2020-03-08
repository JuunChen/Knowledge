目标文件是源代码在编译后，还未进行链接的文件。  
它与可执行文件的结构几乎是一样的，所以可以把它和可执行文件看作是一种类型的文件。 
 
- 在 Windows 下，统称为 PE 文件。
- 在 Linux 下，统称为 ELF 文件。
- 在 Mac OS 下，统称为 MachO 文件。

查看文件的类型：

```
$ file hello.c
> hello.c: c program text, ASCII text
$ file hello.i
> hello.i: c program text, ASCII text
$ file hello.s
> hello.s: assembler source text, ASCII text
$ file hello.o
> hello.o: Mach-O 64-bit object x86_64
$ file a.out
> a.out: Mach-O 64-bit executable x86_64
```
