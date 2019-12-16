### 第三章 简化构建过程

​	在本章节中，通过几个简单的SCons工程构建示例，向您展示无论是哪种编程语言，亦或哪种系统平台，软件构建都将会是一件非常简单的事情。

#### 3.1 指定编译输出文件

​	当调用**Program**构建方法时，它会构建和源代码文件（多份则采取第一个文件）相同名称的输出文件，因此如果源代码是hello.c，则构建出来的可执行文件则为hello（POSIX平台），或者hello.exe（WINDOWS平台）。

```python
Program('hello.c')
```

​	如果您想编译出不同名称的可执行文件，则需要在源文件的前面，指明输出名称即可（如new_hello）：

```python
Program('new_hello', 'hello.c')
```

​	（Scons需要先指明目标文件名称，后面才是源文件，这样做的目的是和大多数的编程语言保持一致，python也是如此：“program = source files”）

​	现在执行scons指令则会得到new_hello的编译输出：(POSIX平台)

```bash
> scons -Q
cc -o hello.o -c hello.c
cc -o new_hello hello.o
```

​	得到new_hello.exe的编译输出：（WINDOWS平台）

```bash
C:\> scons -Q
cl /Fohello.obj /c hello.c /nologo
link /nologo /OUT:new_hello.exe hello.obj
embedManifestExeCheck(target, source, env)
```

#### 3.2 编译多个源文件

​	现在知道了如何编译单个源文件，但是大多数情况下，需要构建的工程，是由多个源文件构成，而非单一文件，因此多源文件编译时，需要将文件名放入一个python列表（list）中：

```python
Program(['prog.c', ['file1.c', 'fil2.c'])
```

​	编译输出为：（POSIX平台）

```bash
> scons -Q
cc -o file1.o -c file1.c
cc -o file2.o -c file2.c
cc -o prog.o -c prog.c
cc -o prog prog.o file1.o file2.o
```

​	（WINDOWS平台）

```bash
C:\>scons -Q
cl /Fofile1.obj /c file1.c /nologo
cl /Fofile2.obj /c file2.c /nologo
cl /Foprog.obj /c prog.c /nologo
link /nologo /OUT:program.exe prog.obj file1.obj file2.obj
embedManifestExeCheck(target, source, env)
```

#### 3.3 采用Glob创建列表

​	您也可以采用Glob函数匹配所有符合条件的源文件，匹配原则采用标准脚本通配符“*”，“?”。[abc]将匹配a,b,c；[!abc]将匹配除abc意外的任意字符。这使得多个源文件的构建变得简单了很多。

```python
Program('program', Glob('*.c'))
```

​	SCons的帮助手册有更多关于Glob函数的具体用法细节，可以参考。

#### 3.4 单一文件构建 VS 文件列表构建

​	构建工程目前有两种形式，一种是采用文件列表，一种是采用单一文件直接构建的方式：

```python
# 文件列表
Program('hello', ['file1.c', 'file2.c'])

# 单一文件
Program('hello', 'hello.c')
```

​	当然您也可以在文件列表中，只存储一份文件：

```python
Program('hello', ['hello.c'])
```

​	其实SCons内部都是将源文件以列表的形式存储，只是为了方便起见，允许用户直接输入单一文件。

####   重要提示

​	尽管SCons支持多种输入方式，但是python语言更为严格，因此在进行list和string格式的文件相加则会产生python语法错误：

```python
common_sources = ['file1.c', 'file2.c']
# 下述代码将会报错， string和list相加
Program('program1', common_sources + 'program1.c')
# 下述代码正确， list与list相加
Program('program2', common_sources + ['program2.c'])
```

#### 3.5 让列表文件易于读写

​	对于文件列表而言，其缺点是每一个文件名称都需要用引号包起来，无论是单引号还是双引号，当列表比较长时，看起来都比较麻烦。SCons提供了很多工具函数使得这一工作变得很简单。

​	SCons提供了Split函数用于分割字符串文件中的一个或多个空格，亦或一个或多个tab，分割后以文件列表的形式存储：

```python
Program('program', Split('main.c file1.c file2.c'))
```

​	（如果您熟悉python的话，您会发现这和python标准字符串string模板的split()函数很相似，但是和split()不同的是，Split()不需要输入一定为string，其他输入也可以（如列表），它会自动将其进行分割成列表，确保输出格式的正确性。）

​	直接将Split()函数嵌入到Program中，显得比较粗鲁，可以进一步采用临时变量增加代码的可读性：

```python
src_files = Split('main.c file1.c file2.c')
Program('program', src_files)
```

​	最后，Split函数不关系输入中有多少个空格，因此您可以跨行编辑列表，进一步提高可读性：

```python
src_file = Split("""" main.c
                      file1.c 
                      file2.c""")
Program('program', src_files)
```

​	请注意，在此示例中，我们使用了Python的“三重引用”语法，该语法允许一个字符串包含多行。三个引号可以是单引号或双引号。

#### 3.6 关键字参数

​	SCons同样支持输入或输出文件的关键字定义。输出文件的关键字是*target*，输入源文件的关键字是*source*:

```python
src_files = Split('main.c file1.c file2.c')
Program(target='program', source=src_files)
```

​	因为关键字已经指明了该参数的含义，因此关键字的顺序不做要求：

```python
src_files = Split('main.c file1.c file2.c')
Program(source=src_files, target='program')
```

​	是否选择使用关键字，以及采用关键字时的顺序指定，这些都是是个人喜好，SCons不做特殊要求。

#### 3.7 编译多个工程

​	如果您想采用同一个SConstruct文件，构建多个工程，最简单的方式是调用Program多次构建：

```python
Program('foo.c')
Program('bar', ['bar1.c', 'bar2.c'])
```

​	SCons则会输出如下：

```bash
> scons -Q
cc -o bar1.o -c bar1.c
cc -o bar2.o -c bar2.c
cc -o bar bar1.o bar2.o
cc -o foo.o -c foo.c
cc -o foo foo.o
```

​	请注意，SCons不一定按照您在SConstruct文件中指定程序的顺序来构建。 但是，SCons确实认识到必须先构建各个目标中间文件，然后才能构建最终输出程序。

#### 3.8 多工程编译共享中间文件

​	代码复用是常见的编程方式，创建库文件（动态或静态库）是广为人知的一种方式。具体库文件的创建参见后续章节。

​	同一份源码文件对应多个工程，最直接的方式是每个工程都加入该源码：

```python
Program(Split('foo.c common1.c common2.c'))
Program('bar', Split('bar1.c bar2.c common1.c common2.c'))
```

​	当采用这种方式构建时，SCons会意识到common1.c和common2.c的复用，因此会将其仅仅编译一份中间文件，尽管生成的目标文件分别链接到各自的中间文件：

```bash
% scons -Q
cc -o bar1.o -c bar1.c
cc -o bar2.o -c bar2.c
cc -o common1.o -c common1.c
cc -o common2.o -c common2.c
cc -o bar bar1.o bar2.o common1.o common2.o
cc -o foo.o -c foo.c
cc -o foo foo.o common1.o common2.o
```

​	如果两个或多个程序共享许多公共源文件，则可以将公共源文件创建为一个列表，以便维护和更改。同时采用python的*+*运算符进行其他列表的扩充：

```python
common = ['common1.c', 'common2.c']
foo_files = ['foo.c'] + common
bar_files = ['bar1.c', 'bar2.c'] + common
Program('foo', foo_files)
Program('bar', bar_files)
```

​	这和前面的写法是完全等价的。

#### 3.9 构建方法参数的覆盖

​	您也可以通过在构建方法（如**Program**等）中增加对应参数，或关键字参数，来指定相关的参数，最终参数会以此为准。

```python
env.Program('hello', 'hello.c', LIBS=['gl', 'glut'])
```

​	或者生成一个非标准后缀的动态库:

```python
env.SharedLibrary('word', 'word.cpp',
SHLIBSUFFIX='.ocx',
LIBSUFFIXES=['.ocx'])
```

​	您也可以采用*parse_flags*关键字将命令行样式参数合并到适当的构造变量中。本例将’include‘增加到CPPPATH变量中, 将‘EBUG’加入到CPPDEFINES变量中，以及将'm'加入到LIBS变量中:

```python
enc = Program('hello', 'hello.c', parse_flags='-Iinclude -DEBUG -lm')
```

​	在对构建方法的调用中，不会克隆编译环境，而是通过调用OverrideEnvironment(), 它会创建一个整个Environment()更轻便编译环境。