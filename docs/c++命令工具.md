c++命令工具

## c++
### c++filt编译函数还原
> 编译后的函数还原成c++定义  

使用方式：`c++filt -n xxx`，比如`c++filt -n _Znwm`，会输出：
```c++
operator new(unsigned long)
```

### 查看符号表
> 查看目标文件内的符号表

`nm -g filename`

#### 参数
- -a，显示所有符号，包括那些专门用来调试的符号。
- -g，只显示全局符号，不显示局部符号。
- -n，按照数字而不是默认的字符排序。
- -p，不排序，按照符号在符号表中出现的次序显示。
- -r，符号按照反序显示（默认就是按照符号名字字符排序的反序，如果带上-p参数就是按照在符号表中出现次序的反序，如果带上-n参数就是按照符号名数字排序的反序）。
- -u，只显示未定义的符号。
- -U，不显示未定义的符号，与-u的作用刚好相反。
- -j，只显示符号的名字，而不显示符号对应的数值和类型。
- -s segname sectname: 只解析位于文件中segname段里sectname节内的符号。
- -arch archtype

## Clang参数详解
> 参考: [https://www.cnblogs.com/piterzhang/p/8971617.html](https://www.cnblogs.com/piterzhang/p/8971617.html)
[https://clang.llvm.org/docs/CommandGuide/clang.html](https://clang.llvm.org/docs/CommandGuide/clang.html)


### 查看内存模型与虚表模型
```
clang++ -cc1 --std=c++17 -emit-llvm -fdump-record-layouts -fdump-vtable-layouts oper.cpp
```


### include配置
- 查看include路径: `clang++ -v -x c++ -E -`
- Mac系统路径: `/Library/Developer/CommandLineTools/SDKs/MacOSX.sdk/usr/include`，务必添加到`CPLUS_INCLUDE_PATH`
- C: `export C_INCLUDE_PATH=XXXX:$C_INCLUDE_PATH`
- CPP: `export CPLUS_INCLUDE_PATH=XXX:$CPLUS_INCLUDE_PATH`

### 查看使用的动态链接库
>[https://blog.csdn.net/lovechris00/article/details/81561627](https://blog.csdn.net/lovechris00/article/details/81561627)
- 动态链接库: `otool -L build/main`

## GCC命令
> [https://blog.csdn.net/zhaoyun_zzz/article/details/82466031](https://blog.csdn.net/zhaoyun_zzz/article/details/82466031)

## llvm-config
> 除了默认的/usr/include, /usr/local/include等include路径外，还可以通过设置环境变量来添加系统include的路径：

## 自动生成.clang-format格式化文件
`clang-format -style=llvm -dump-config > .clang-format`