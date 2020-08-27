# this指针
this指针根据函数的调用方式，访问场景有两种: 实例直接访问、转存后访问。
考虑如下的this代码: 
```ts
class A {
  public b: B;
}
class B {
  c:number = 3;
  getC() {
    return this.c;
  }
}
```
1. 实例直接访问:
```ts
var a = new A();
a.b.getC();
```
2. 转存后访问
```ts
var a = new A();
var cc = a.b.getC;
cc();

var d = {
  cc: a.b.getC,
};
d.cc();
```

## 实例直接访问
实例直接访问的场景比较简单，函数左侧即是我们需要的this指针，直接将其load，然后传入函数参数即可。

## 转存后访问
转存后访问，处理比较复杂，需要在转存的地方存储绑定实例内存（即this指针）与函数，调用函数时，将绑定的实例地址传入函数参数。
### 触发场景
1. 函数赋值给其他变量
2. 函数作为返回值返回
3. 函数作为返回值中的一部分，比如:
```ts
return {
  cc: a.b.getC
}
```
