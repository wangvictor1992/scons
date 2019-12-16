### 第四章 编译链接库文件

​	大型软件工程一般都是由很多库文件构成，采用SCons编译库文件是一件非常简单的事情。

#### 4.1 编译库文件

​	您只需要采用**Library**的构建方法代替**Program**即可：

```python
Library('foo', ['f1.c', 'f2.c', 'f3.c'])
```

​	SCons会自动根据系统来创建合适的库文件前缀和后缀，因此在POSIX或者Linux系统上，上述示例将会构建以下内容（ranlib可能在某系系统中不会被调用）：

```bash
> scons -Q
cc -o f1.o -c f1.c
cc -o f2.o -c f2.c
cc -o f3.o -c f3.c
ar rc libfoo.a f1.o f2.o f3.o
ranlib libfoo.a
```

​	在Windows系统中，编译输出将会如下所示：

```bash
C:\>scons -Q
cl /Fof1.obj /c f1.c /nologo
cl /Fof2.obj /c f2.c /nologo
cl /Fof3.obj /c f3.c /nologo
lib /nologo /OUT:foo.lib f1.obj f2.obj f3.obj
```

​	目标文件的构建规则和**Program**方法类似，如果你不特别指定目标库文件名称，SCons将会从源文件列表中选择第一个作为库文件名称，同时SCons还会自动给库文件加入前缀和后缀。

##### 4.1.1 通过源文件或中间文件编译

​	上述示例介绍了通过源文件列表构建库文件，SCons同样也支持通过中间文件构建，或者源文件和中间文件混在一起编译也可以。

```python
Library('foo', ['f1.c', 'f2.o', 'f3.c', 'f4.o'])
```

​	同时SCons也会意识到，只有源文件才需要进一步构建成中间文件：

```bash
> scons -Q
cc -o f1.o -c f1.c
cc -o f3.o -c f3.c
ar rc libfoo.a f1.o f2.o f3.o f4.o
ranlib libfoo.a
```

​	当然，无论源文件还是中间文件，要向构建成功，它们都必须真实存在。下一章节将会进一步介绍关于*Node Objects*的相关内容。

##### 4.1.2 构建静态库文件：StaticLibrary方法

​	**Library**构建方法构建的是传统静态库，如果您想特别声明构建的是静态库文件，则可以通过调用**StaticLibrary**来进一步显示声明：

```python
StaticLibrary('foo', ['f1.c', 'f2.c', 'f3.c'])
```

​	对于**Library**和**StaticLibrary**，他们完全等价，没有任何区别。

##### 4.1.3 构建动态库文件(DLL)：SharedLibrary方法

​	如果您想构建动态库文件（POSIX系统， WINDOWS系统是DLL文件），您可以采用**SharedLibrary**方法：

```python
SharedLibrary('foo', ['f1.c', 'f2.c', 'f3.c'])
```

​	在POSIX系统上的输出为：

```bash
> scons -Q
cc -o f1.os -c f1.c
cc -o f2.os -c f2.c
cc -o f3.os -c f3.c
cc -o libfoo.so -shared f1.os f2.os f3.os
```

​	在Windows系统上输出为：

```bash
C:\>scons -Q
cl /Fof1.obj /c f1.c /nologo
cl /Fof2.obj /c f2.c /nologo
cl /Fof3.obj /c f3.c /nologo
link /nologo /dll /out:foo.dll /implib:foo.lib f1.obj f2.obj f3.obj
RegServerFunc(target, source, env)
embedManifestDllCheck(target, source, env)
```

​	SCons在构建时，会自动添加-shared（POSIX）或者/dll（Windows系统）编译选项，以保证编译的正确性。

#### 4.2 链接库文件

​	链接是库文件使用的最后一步，您可以通过指明LIBS变量关键字，来指定需要链接的库文件；通过指明LIBPATH变量关键字，来指定库文件的查找路径：

```python
Library('foo', ['f1.c', 'f2.c', 'f3.c'])
Program('prog.c', LIBS=['foo', 'bar'], LIBPATH='.')
```

​	请注意，您不需要特别声明库文件的前缀（如lib），或者后缀（如.a或.lib），SCons会自动根据系统来查找相关前缀或后缀。

​	在POSIX或Linux系统中，上述编译过程如下：

```bash
> scons -Q
cc -o f1.o -c f1.c
cc -o f2.o -c f2.c
cc -o f3.o -c f3.c
ar rc libfoo.a f1.o f2.o f3.o
ranlib libfoo.a
cc -o prog.o -c prog.c
cc -o prog prog.o -L. -lfoo -lbar
```

​	在Windows系统中，编译输出如下：

```bash
C:\>scons -Q
cl /Fof1.obj /c f1.c /nologo
cl /Fof2.obj /c f2.c /nologo
cl /Fof3.obj /c f3.c /nologo
lib /nologo /OUT:foo.lib f1.obj f2.obj f3.obj
cl /Foprog.obj /c prog.c /nologo
link /nologo /OUT:prog.exe /LIBPATH:. foo.lib bar.lib prog.obj
embedManifestExeCheck(target, source, env)
```

​	另外，如果依赖的库文件只有一个，您也可以直接指定LIBS变量，而不是列表形式：

```python
Program('prog.c', LIBS='foo', LIBPATH='.')
Program('prog.c', LIBS=['foo'], LIBPATH='.')
```

​	上述两种写法完全等价。

#### 4.3 查找库文件：LIBPATH变量

​	默认情况下，连接器只会在系统路径中查找库文件，SCons可以通过用户指定的LIBPATH变量，来查找用户定义路径：

```python
Program('prog.c', LIBS = 'm',
       			  LIBPATH = ['/usr/lib', 'usr/local/lib'])
```

​	这里推荐使用python的列表（list），因为python是跨平台的，这要迁移起来比较方便。当然您也可以将搜索路径放到一个字符串中，采用系统指定的分割符分开，如POSIX系统中采用冒号，Windows系统中采用分号：

```python
# POSIX
LIBPATH = '/usr/lib:/usr/local/lib'
# Windows
LIBPATH = 'C:\\lib;D:\\lib'  
```

​	请注意，python在Windows路径中要求采用反斜杠转义符。

​	当链接器执行时，SCons会自动创建合适的标志，以便在和工程文件相同目录下查找，在POSIX或Linux系统中，上述示例编译输出如下：

```bash
> scons -Q
cc -o prog.o -c prog.c
cc -o prog prog.o -L/usr/lib -L/usr/local/lib -lm
```

​	在Windows系统中，输出如下：

```bash
C:\>scons -Q
cl /Foprog.obj /c prog.c /nologo
link /nologo /OUT:prog.exe /LIBPATH:\usr\lib /LIBPATH:\usr\local\lib m.lib prog.obj
embedManifestExeCheck(target, source, env)
```

​	整体而言，SCons会自动根据系统不同，而创建不同的编译选项，以保证编译输出的正确性。