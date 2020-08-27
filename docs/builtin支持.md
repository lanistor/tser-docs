# builtin支持
> builtin支持，可使用c++编写类和函数，在需要的ts模块中将函数和结构体引入，即可正常使用。

## 完整类支持
类成员函数，可拆分为：结构体+带结构体指针的函数调用。考虑如下内置类：
```c++
class A{
  public:
  int count = 0;
  int nextCount() {
    return count ++;
  }
  A() { }
  ~A() { }
}
```

在ts中使用`class A`，首先需要引入其结构体定义:
```c++
%class.A = type { i32 }
```

调用其中的方法，需要引入其方法定义（不引入实现）
```c++
define linkonce_odr i32 @_ZN1A9nextCountEv(%class.A*)
```

### 操作过程
> 一般调用过程如下
1. 首先进行实例化: `var a: A = new A();`
2. 然后调用其函数: `var b: int32 = a.nextCount();`

对应的IR编译流程为:
1. 引入`A`的结构体
2. 按照`A`的结构体申请一块内存
3. 调用`A`的构造函数初始化内存
4. 引入函数`nextCount`，同时引入其函数配置: `return_value`, `using_this`, `type`等，参见`FunctionVariableValue`定义
5. 调用函数`nextCount`，将`A`的结构体内存当做`this`传递

### 类初始化过程
按照IR中的方式，来初始化类:
1. 实例化`ClassInfo`
2. 初始化所有`properties`
3. 创建类结构体
4. 初始化所有方法
5. 创建类结构体
6. 初始化映射表
7. 


## 单个函数支持


## 作用域管理
1. 使用`GlobalScope`来保存所有的全局对象
2. 定义`GlobalScopeModuleManager`来管理引入到模块的类、函数，其作用为进行初始化操作与避免重复引入
3. 作用域变量查找，仍采用逐层查找的方式，当`ModuleScope`查找不到时，使用`GlobalScope`在`GlobalScopeModuleManager`中查找
