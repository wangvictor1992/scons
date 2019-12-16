### 第五章 编译节点对象

​	在SCons内部，所有的文件和路径都被看作是节点（*Nodes*），我们可以采用这种方式让您的SConscript脚本更加便于迁移和阅读。

#### 5.1 返回目标节点列表的编译方法

​	所有的构建方法返回一个节点对象列表，这些节点可以被用作其他构建方法的参数。

​	举例而言，如果我们想要构建两个中间文件，他们采用不同的编译参数：

```python
Object('hello.c', CCFLAGS='-DHELLO')
Object('goodbye.c', CCFLAGS='-DGOODBYE')
```

​	将上述两个中间文件连接在一起，构建最终的输出文件，一种方式是将上述输出中间文件放入一个列表中，作为**Program**的参数：

```python
Object('hello.c', CCFLAGS='-DHELLO')
Object('goodbye.c', CCFLAGS='-DGOODBYE')
Program(['hello.o', 'goodbye.o'])
```

​	这样做在跨平台方面会存在问题，因为在Windows平台上，生成的中间文件是hello.obj和goodbye.obj，而非hello.o和goodby.o。

​	较好的处理方式是将**Object**的构建输出存入变量中，这样我们可以不断在列表后追加新的内容，将其作为**Program**的输入：

```python
hello_list = Object('hello.c', CCFLAGS='-DHELLO')
goodbye_list = Object('goodbye.c', CCFLAGS='-DGOODBYE')
Program(hello_list + goodbye_list)
```

​	这样SConstruct脚本的跨平台性可以得到保证，其在Linux平台输出如下：

```bash
> scons -Q
cc -o goodbye.o -c -DGOODBYE goodbye.c
cc -o hello.o -c -DHELLO hello.c
cc -o hello hello.o goodbye.o
```

​	Windows平台输出如下：

```bash
C:\>scons -Q
cl /Fogoodbye.obj /c goodbye.c -DGOODBYE
cl /Fohello.obj /c hello.c -DHELLO
link /nologo /OUT:hello.exe hello.obj goodbye.obj
embedManifestExeCheck(target, source, env)
```

​	后续将会介绍更多的编译节点的使用方法。

#### 5.2 明确创建文件和路径节点

​	值得一提的是，SCons明确了文件节点和路径节点的不同，SCons支持**File** 和**Dir**两个函数，用于返回文件和路径节点：

```python
hello_c = File('hello.c')
Program(hello_c)

classes = Dir('classes')
Java(classes, 'src')
```

​	通常情况下，您不需要手动调用File或Dir，因为调用构建方法时，会自动将输入作为文件或目录的名称，并将其转换为一个Node对象。除非您需要明确SCons的节点类型，比如在调用构建方法或者需要调用特殊文件或路径树的时候。

​	有时候，您可能在无法明确编译系统环境的前提下，需要一个输入，这可能是文件也可能是一个路径，此时SCons提供了一个**Entry**函数，用于返回一个文件节点或者路径节点。

```python
xyzzy = Entry('xyzzy')
```

​	这里返回的xyzzy节点，将会在其第一次被调用的时候，转化为文件节点或路径节点。

#### 5.3 打印节点文件名称

​	大多数情况下我们需要调用节点去打印其内部的文件名称，但是请注意此时的对象是节点列表，而非文件对象，因此打印的时候需要增加下标访问：

```python
object_list = Object('hello.c')
program_list = Program(object_list)
print("The object file is: %s"%object_list[0])
print("The program file is: %s"%program_list[0])
```

​	在POSIX系统输出如下：

```bash
> scons -Q
The object files is: hello.o
The program file is: hello
cc - o hello.o -c hello.c
cc -o hello hello.o
```

​	在Windows系统输出如下：

```bash
C:\>scons -Q
The object file is: hello.obj
The program file is: hello.exe
cl /Fohello.obj /c hello.c /nologo
link /nologo /OUT:hello.exe hello.obj
embedManifestExeCheck(target, source, env)
```

​	请注意，在上面的示例中，object_list[0]从列表中提取了一个实际的Node对象，Python的print语句将该对象转换为要打印的字符串。

#### 5.4 将节点对象转换为字符串

​	如上节介绍所示，我们可以直接打印节点的文件信息，但是如果您想得到一个节点字符串，而非列表，则可以通过python内置的*str*函数实现。举例而言，如果您希望使用Python的*os.path.exists*来确认文件是否存在，您可以采用如下方式：

```python
import os.path
program_list = Program('hello.c')
program_name = str(program_list[0])
if not os.path.exists(program_name):
    print("%s doses not exist!"%program_name)
```

​	此时，在POSIX系统中将会得到如下输出：

```bash
> scons -Q
hello does not exist!
cc -o hello.o -c hello.c
cc - o hello hello.o
```

#### 5.5 GetBuildPath: 通过节点或字符串获取路径

​	*env.GetBuildPath(file_or_list)*返回节点或者字符串（描述路径）的路径，当然输入参数也可以是列表，这将会返回路径的列表信息。如果输入是单个节点的话，则相当于调用了*str（node）*。字符串参数可以被初始化在*Environment*函数参数列表中，此时将会被传递到后续的调用函数中：

```python
env = Environment(VAR="value")
n = File("foo.c")
print(env.GetBuildPath([n, "sub/dir/$VAR"]))
```

​	打印信息如下：

```bash
> scons -Q
['foo.c', 'sub/dir/value']
scons: '.' is up to date
```

​	*GetBuildPath*函数也可以被默认的*Environment*环境调用，而非特殊指定。