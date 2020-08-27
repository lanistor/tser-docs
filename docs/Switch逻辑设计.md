# Switch逻辑设计

## Switch特性
1. 允许break，不允许continue，break生效范围为当前case
2. `case`遵循严格的代码排列顺序，未加`break`的`case`执行结束后，会直接进入下一条`case`(会跳过case的执行条件)
3. 所有case均未匹配时，才执行`default`
4. `default`执行结束后，再次执行第2条规则


## 整型Switch
> 对于整型`switch`，直接使用LLVM的`switch`语句即可
```ts
var a: number = 11;
switch (a) {
  case 1:
    a = a + 100;
  break;
  case 2:
    a = a + 200;
  default:
    a = a + 1000;
  break;
  case 3:
    a = a + 300;
}
```
生成的IR:
```c++
%1 = load i32, i32* @a
switch i32 %1, label %8 [
  i32 1, label %2
  i32 2, label %5
  i32 3, label %11
]

2:                                                ; preds = %0
  %3 = load i32, i32* @a
  %4 = add i32 %3, 100
  store i32 %4, i32* @a
  br label %14

5:                                                ; preds = %0
  %6 = load i32, i32* @a
  %7 = add i32 %6, 200
  store i32 %7, i32* @a
  br label %8

8:                                                ; preds = %0, %5
  %9 = load i32, i32* @a
  %10 = add i32 %9, 1000
  store i32 %10, i32* @a
  br label %14

11:                                               ; preds = %0
  %12 = load i32, i32* @a
  %13 = add i32 %12, 300
  store i32 %13, i32* @a
  br label %14

14:                                               ; preds = %2, %8, %11
  ret i32 0
```
## 非整型Switch
对非整型Switch，LLVM未直接支持（不支持非常量整型的比较），处理比较麻烦，处理方式目前有2种: 
1. 使用if-else来代替
2. case部分转换为case序列的索引来对比

### if-else代替方案
该方案可行，将所有case语句转化为if-else语句。
#### 优势
1. 最容易想到的方案，现有语句的组合既可以实现
#### 劣势
1. 无法复用LLVM的switch逻辑，需要手动处理的逻辑比较多
2. case语句与case的statementList语句是2个单独的分支（无break语句时要跳过下一条case检测，而直接进入下一条的statementList），if-else逻辑很复杂

### case部分转换为case序列的索引的方案
该方案可行，将case部分抽离出来进行比较，得到匹配的index，然后再使用index对比

#### 原理
对于字符串或引用类型，对比其起始地址是否相同

#### 优势
1. 可复用LLVM的switch逻辑
2. 将case的对比提前于switch语句，复杂度进行了拆分，逻辑较简单

```ts
var a: number = 11;
var b: string = "aa";
switch (b) {
  case "aa":
    a = a + 100;
    break;
  case "bb":
    a = a + 200;
    break;
  default:
    a = a + 300;
}
```
生成的IR为：
```c++
@a = linkonce_odr global i32 11
@aa = private unnamed_addr constant [3 x i8] c"aa\00", align 1
@bb = private unnamed_addr constant [3 x i8] c"bb\00", align 1
@b = linkonce_odr global i8* getelementptr inbounds ([3 x i8], [3 x i8]* @aa, i32 0, i32 0)

define i32 @main() {
  %1 = alloca i32
  %2 = load i8*, i8** @b
  %3 = icmp eq i8* %2, getelementptr inbounds ([3 x i8], [3 x i8]* @aa, i32 0, i32 0)
  br i1 %3, label %4, label %5

4:                                                ; preds = %0
  store i32 0, i32* %1
  br label %5

5:                                                ; preds = %4, %0
  %6 = load i8*, i8** @b
  %7 = icmp eq i8* %6, getelementptr inbounds ([3 x i8], [3 x i8]* @bb, i32 0, i32 0)
  br i1 %7, label %8, label %9

8:                                                ; preds = %5
  store i32 1, i32* %1
  br label %9

9:                                                ; preds = %8, %5
  %10 = load i32, i32* %1
  switch i32 %10, label %17 [
    i32 0, label %11
    i32 1, label %14
  ]

11:                                               ; preds = %9
  %12 = load i32, i32* @a
  %13 = add i32 %12, 100
  store i32 %13, i32* @a
  br label %20

14:                                               ; preds = %9
  %15 = load i32, i32* @a
  %16 = add i32 %15, 200
  store i32 %16, i32* @a
  br label %20

17:                                               ; preds = %9
  %18 = load i32, i32* @a
  %19 = add i32 %18, 300
  store i32 %19, i32* @a
  br label %20

20:                                               ; preds = %11, %14, %17
  ret i32 0
}
```