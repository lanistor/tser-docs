# c++基础

## 交叉编译
> https://blog.csdn.net/qq_23599965/article/details/90901235

使用方式: `clang++ --target=i386-apple-macosx10.15.1 -emit-llvm -S -c -O0 oper.cpp`

## 数据类型

|类型/宽度|32位|64位|
|----          |----    |----    |
|char          |8       |8       |
|short         |16      |16      |
|int           |32      |32      |
|long          |32      |64      |
|unsinged long |32      |64      |
|long long     |64      |64      |
|int64_t       |64      |64      |
|new (type)     |32      |64      |

## 名词解释
### POD类类型
POD类类型是指class、struct、union，其必须满足以下条件:
- 不具有用户定义的构造函数、析构函数、拷贝算子、赋值算子
- 不具有继承关系，因此没有基类；不具有虚函数，所以就没有虚表；
- 非静态数据成员没有私有或保护属性的、没有引用类型的
- 没有非POD类类型的（即嵌套类都必须是POD）
- 没有指针指到成员类型

