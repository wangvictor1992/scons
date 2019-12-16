### 第二章 简单例程

​	本章会介绍几个简单的SCons构建工程，以演示使用SCons在不同类型的系统上根据几种不同的编程语言来构建程序是多么容易。

#### 2.1 构建简单的C/C++程序

​	这是一个C语言版的“Hello, World” (文件名：hello.c)：

```c
int main() {
    printf("Hello, world!\n");
    return 0;
}
```

​	下面介绍如何使用SCons对其进行构建。新建一个SConstruct文件，输入一下内容：

```python
Program("hello.c")
```

​	这个简单的SCons配置描述了两条信息：工程的构建类型（可执行文件、库文件等）和待编译文件（hello.c）。**Program**是一种编译方法，告诉SCons需要构建一种可执行文件。

​	就这样，现在运行scons命令来构建整个工程。在POSIX平台系统上，例如Linux或UNIX，你将会看到如下输出：

```bash
> scons
scons: Reading SConscript files ...
scons: done reading SConscript files.
scons: Building targets ...
cc -o hello.o -c hello.c
cc -o hello hello.o
scons: done building targets. 
```

​	在windows平台，采用Micosoft Visual C++编译器，你将会看到如下输出：

```bash
C:\> scons: Reading SConscript files ...
scons: done reading SConscript files.
scons: Building targets ...
cl /Fohello.obj /c hello.c /nologo
link /nologo /OUT:hello.exe hello.obj
embedManifestExeCheck(target, source, env)
scons: done building targets.
```

​	首先，您需要指定源文件的名称，并且SCons会根据源文件名正确推断出要构建的对象和可执行文件的名称。

​	其次，相同的输入SConstruct文件，在两个系统上会生成正确的输出文件名。（POSIX：hello.o和hello；WINDWOS：hello.obj和hello.exe）。这是一个简单的例子，来说明SCOns如何编写并构建简单的程序。

（请注意，对于本指南中所有的示例，我们不会同时提供windows和Linux两种输出示例，除非有特殊说明，否则输出效果在两个平台上同样有效）

#### 2.2 构建中间文件

​	**Program**构建方法只是SCons提供的众多构建方法之一，另外一种构建方法是**object**，用以生成中间文件：

```python
Object('hello.c')
```

​	现在你可以执行scons命令，去构建整个工程，在POSIX工程中这将会仅仅构建hello.o。

```bash
> scons
scons: Reading SConscript files ...
scons: done reading SConscript files.
scons: Building targets ...
cc -o hello.o -c hello.c
scons: done building targets.
```

​	在windows平台上，将会仅仅构建出来hello.obj(采用Microsoft Visual C++ 编译器)：

```bash
C:\>scons
scons: Reading SConscript files ...
scons: done reading SConscript files.
scons: Building targets ...
cl /Fohello.obj /c hello.c /nologo
scons: done building targets.
```

#### 2.3 简单Java构建

​	SCons构建Java工程也很简单，不像**Program**和**Object**这些构建方法， **Java**构建方法需要你指明构建的输出目标路径：

```bash
Java('classes', 'src')
```

​	如果src路径包含了一个hello.java文件，那么执行scons命令后的输出将会如下：

```bash
> scons
scons: Reading SConscript files ...
scons: done reading SConscript files.
scons: Building targets ...
javac -d classes -sourcepath src src/hello.java
scons: done building targets.
```

​	我们会覆盖更多的java编译细节，包含java架构和其他类型的文件，详见第26章。

#### 2.4 编译后清除

​	采用SCons我们不需要增加特殊的指令在构建后执行清除操作，相反，你可以简单使用-c或者--clean选项，此时SCons会自动删除构建的文件，所以你可以采用scons -c进行构建后的清理工作，在POSIX平台上输出如下：

```bash
> scons
scons: Reading SConscript files ...
scons: done reading SConscript files.
scons: Building targets ...
cc -o hello.o -c hello.c
cc -o hello hello.o
scons: done building targets.

> scons -c
scons: Reading SConscript files ...
scons: done reading SConscript files.
scons: Cleaning targets ...
Removed hello.o
Removed hello
scons: done cleaning targets.
```

​	在windows平台上输出如下：

```bash
C:\>scons
scons: Reading SConscript files ...
scons: done reading SConscript files.
scons: Building targets ...
cl /Fohello.obj /c hello.c /nologo
link /nologo /OUT:hello.exe hello.obj
embedManifestExeCheck(target, source, env)
scons: done building targets.

C:\>scons -c
scons: Reading SConscript files ...
scons: done reading SConscript files.
scons: Cleaning targets ...
Removed hello.obj
Removed hello.exe
scons: donw cleaning targets.
```

​	请注意， SCons通过改变输出来告诉你，它在Cleaning targets ... 和 donw cleaning targets.

#### 2.5 SConstruct文件

​	SConstruct文件，类似于Make系统中的Makefile文件，这是SCons读取并控制编译构建的输入文件。

##### 2.5.1 SConstruct文件是Python脚本

​	SConstruct和Makefile的最大不同是：SConstruct文件是python脚本，如果你不太熟悉python，也不用担心，这本指南会详细介绍Python的相关用法，保证您会高效地使用SCons，而且python也是非常简单易学的。

​	使用Python作为脚本语言的一个方面是，您可以在SConstruct中使用Python的注释约定，将注释放入文件中。也就是说，“＃”和行尾之间的所有内容都将被忽略：

```python
# Arrange to build the "hello" program.
Program('hello.c') # "hello.c" is the source file.
```

​	在本指南剩余的部分您会发现，使用脚本语言可以大大简化复杂的构建过程。

##### 2.5 SCons函数和构建顺序无关

​	SConstruct有一点和一般python脚本不同，SCons函数的书写调用顺序，并不影响SCons真实的构建顺序，这点和Makefile有点相似。换句话说，当你调用**Program**构建时（或者其他构建方法），SCons并非在此刻构建可执行文件，相反，这仅仅是告诉SCons你想要获得一个可执行文件的构建结果。举例来说，需要构建hello.c的源文件，SCons也仅仅是获取了构建可执行文件hello和源码hello.c之间的关系。（在第6章会详细介绍）

​	我们可以通过打印输出，来区分SCons仅仅是调用构建方法如**Program**获取构建信息，还是真实构建可执行文件。通过python的print函数，我们可以看到SCons的**Program**构建过程：

```python
print("Calling Program('hello.c')")
Program('hello.c')
print("Calling Program('goodbye.c')")
Program('goodbye.c')
print("Finished calling Program()")
```

​	当执行scons指令时，我们可以看到输出如下：

```bash
> scons
scons: Reading SConscript files ...
Calling Program('hello.c')
Calling Program('goodbye.c')
Finished calling Program()
scons: done reading SConscript files.
scons: Building targets ...
cc -o goodbye.o -c goodbye.c
cc -o goodbye goodbye.o
cc -o hello.o -c hello.c
cc -o hello hello.o
scons: done building targets.
```

​	注意到，SCons先构建了goodbye，即使在SConstruct中优先定义了hello。

#### 2.6 降低SCons输出详细程度

​	你已经知道了SCons构建时的一些输出信息：

```bash
C:\>scons
scons: Reading SConscript files ...
scons: done reading SConscript files.
scons: Building targets ...
cl /Fohello.obj /c hello.c /nologo
link /nologo /OUT:hello.exe hello.obj
embedManifestExeCheck(target, source, env)
scons: done building targets.
```

​	这些输出信息描述了SCons的工作顺序，SCons会先读取所有的配置文件（SConstruct和SConscript文件等），其次才是构建具体的目标。这些输出信息还会描述一些错误信息，包括【配置文件读取过程中的出错和编译执行过程中的出错等。

​	当然，这些输出结果会导致输出比较混乱，我们可以通过-Q参数来禁用：

```bash
C:\>scons -Q
cl /Fohello.obj /c hello.c /nologo
link /nologo /OUT:hello.exe hello.obj
embedManifestExeCheck(target, source, env)
```

​	因为我们希望本指南用户重点关注SCons实际在做什么，所以我们将使用该-Q选项，从本指南中所有其他示例的输出中删除这些消息。