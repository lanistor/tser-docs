# class内存模型
class内存模型，与结构体类似，但由于支持多态的特性，内存模型略复杂。

## 基础内存模型
如果不考虑多态，class内存模型与结构体基本一致，均是线性结构；对于继承的实现也比较简单，仅是将父类的数据结构置于最顶部，比如如下class：
```c++
class AA {
  public:
  int    a;
  double b;
};
class BB : public AA {
  public:
  int  a;
  char c;
};
```
编译生成的IR如下:
```c++
%class.BB = type <{ %class.AA, i32, i8, [3 x i8] }>
%class.AA = type { i32, double }
```
这样可以满足继承与类型转换，因为父类的内存地址位于子类的头部，当需要转换成父类时，可直接从索引根部按照父类的方式操作内存。
## 多态内存模型
### 业界通用设计
> 参考: [virtual-table-by-llvm](https://stackoverflow.com/questions/54879921/how-to-implement-virtual-table-by-using-llvm)

多态的设计，我们参考了c++和java，二者有一个相同的设计：虚函数表。
虚函数表的维护的是虚函数的地址映射，下面我们简单介绍下：
如下c++代码：
```c++
class AA {
  public:
  int    a = 100;
  double b;
  virtual getA() {
    return this.a;
  }
};
class BB : public AA {
  public:
  int  a = 200;
  char c;
  getA() {
    return this.a;
  }
  virtual getC() {
    return this.c;
  }
};
```
生成的数据结构与内存结构如下：
```c++
// class AA 实例的内存模型 
    0 | class AA
    0 |   (AA vtable pointer) // 指向AA的虚函数表
    8 |   int a
   16 |   double b
      | [sizeof=24, dsize=24, align=8,
      |  nvsize=24, nvalign=8]

// class BB 实例的内存模型
    0 | class BB
    0 |   class AA (primary base)
    0 |     (AA vtable pointer)  // 指向BB的虚函数表
    8 |     int a
   16 |     double b
   24 |   int a
   28 |   char c
      | [sizeof=32, dsize=29, align=8,
      |  nvsize=29, nvalign=8]

// AA 虚函数表
   0 | offset_to_top (0)
   1 | AA RTTI
       -- (AA, 0) vtable address --
   2 | int AA::getA()

// BB 虚函数表
   0 | offset_to_top (0)
   1 | BB RTTI
       -- (AA, 0) vtable address --
       -- (BB, 0) vtable address --
   2 | int BB::getA()
   3 | char BB::getC()
```
先不考虑`offset_to_top`和`RTTI`。
虚函数表记录了每个虚函数的地址，父类(AA)使用数组记录自己的虚函数地址，子类(BB)将自己重写过的函数的地址改写成自己的函数，同时将自己的虚函数加入AA虚函数表的末尾。

虚函数归属于class（而非class实例），使用全局常量存储（与class相同）。

以上是c++的处理方式，在java中，所有的可继承函数，均被当做虚函数存储在虚函数表中。

### 上述设计的不足
c++和java的设计，存在一个问题，无法处理属性，而js中是可以处理的。比如在上述代码中，我们测试如下代码：
```c++
BB b;
A& a = b;
cout << a.a; // 输出100，而非200
```
上述代码在c++和java中会输出100，而非200；而在js中可以输出200。
即在js中，指向子类的父类指针，可以访问到子类的属性，这点与`dart`相同。
我们需要实现这个设计。

## 我们的方案
我们采用虚函数表的形式来处理属性，我们称之为属性表，属性表记录各个属性的偏移量（即索引）；在子类中，可以重写这些偏移量，并且将自己的属性追加到末尾。我们在属性访问的时候，不是直接访问属性的索引，而是访问其在属性表中的索引，然后拿到修正后的偏移量，再次访问真实的位置。

源代码:
```c++
class AA {
  public:
  int         a;
  double      b;
  int getA() {
    return this->a;
  }
};
class BB : public AA {
  public:
  int  a;
  char c;
  int  getA() {
    return this->a;
  }
  char getC() {
    return this->c;
  }
};

class CC : public BB {
  public:
  int  a;
  char d;
};
```
生成的内存结构:
```c++
// class AA 实例的内存模型 
    0 | class AA
    0 |   (AA virtual property table pointer) // 指向AA的属性表
    8 |   (AA virtual function table pointer) // 指向AA的函数表
   16 |   int a
   24 |   double b

// class BB 实例的内存模型
    0 | class BB
    0 |   class AA (primary base)
    0 |     (AA virtual property table pointer) // 指向BB的属性表
    8 |     (AA virtual function table pointer) // 指向BB的函数表
   16 |     int a
   24 |     double b
   32 |   int a
   36 |   char c

// class CC 实例的内存模型
    0 | class CC
    0 |   class BB (primary base)
    0 |     class AA (primary base)
    0 |       (AA virtual property table pointer) // 指向CC的属性表
    8 |       (AA virtual function table pointer) // 指向CC的函数表
   16 |       int a
   24 |       double b
   32 |     int a
   36 |     char c
   40 |   int a
   44 |   char d

// AA属性表(使用int存储索引，故步长为4)
    0 | 16 // a的偏移量
    4 | 24 // b的偏移量

// BB属性表
    0 | 32 // a
    4 | 24 // b
    8 | 36 // c

// CC属性表
    0 | 40 // a
    4 | 24 // b
    8 | 36 // c
   12 | 44 // d

```

## 内存存储
属性表与函数表同样存储在全局常量中，使用数组存储。
属性表、结构如下：
```c++
// 属性表
@A_property_table = global [2 x i32] [ i32 16, i32 24 ]
@B_property_table = global [3 x i32] [ i32 32, i32 24, i32 36 ]
@C_property_table = global [4 x i32] [ i32 40, i32 24, i32 36, i32 44 ]

// 函数表
@A_function_table = global [1 x void (...)*] [
  void (...)* bitcast (i32 (%class.A*)* @A_getA to void (...)*),
]

@B_function_table = global [2 x void (...)*] [
  void (...)* bitcast (i32 (%class.A*)* @B_getA to void (...)*),
  void (...)* bitcast (i32 (%class.A*)* @B_getC to void (...)*),
]

@C_function_table = global [2 x void (...)*] [
  void (...)* bitcast (i32 (%class.A*)* @B_getA to void (...)*),
  void (...)* bitcast (i32 (%class.A*)* @B_getC to void (...)*),
]

// 类结构表
@class.A = type<i32**, void (...)**, i32, double >
@class.B = type< @class.A, i32, i1 >
@class.C = type< @class.B, i32, i1 >
```

`@class.A`中，前2个参数分别为:
1. 第1个: 属性表指针，可为空
2. 第2个: 函数表指针，可为空

属性表和函数表同时为空或不为空。

## 例外场景
1. 标记为`private`、`static`、`final`的属性或方法，不加入映射表
2. 标记为`static`的属性，不加入类结构，而直接定义全局变量


## 函数调用流程

## 访问性能优化
1. 标记为final的class，不添加虚映射
2. 标记为private的参数，不加入虚映射
3. 标记为final的方法，不加入虚映射
