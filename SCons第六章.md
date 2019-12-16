### 第六章 依赖关系

​	到目前为止，我们通过SCons已经可以成功构建工程，构建的另一个关键在于，不能浪费时间在未更改的文件上，换句话说，我们只需要构建有改动的文件。当重复构建同一个工程时，你会发现SCons完全满足这个要求：

```python
Program('hello.c')
```

编译输出如下：

```bash
> scons -Q
cc -o hello.o -c hello.c
cc -o hello hello.o
> scons -Q
scons: '.' is up to date.
```

​	当第二次构建时，SCons意识到hello工程已经是最新了，因此提示不再需要编译。通过指明构建工程，可以更加清晰地看出这个过程：

```bash
> scons -Q hello
cc -o hello.o -c hello.c
cc -o hello hello.o
> scons -Q hello
scons: `hello' is up to date.
```

