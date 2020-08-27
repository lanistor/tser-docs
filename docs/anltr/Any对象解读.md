# Any对象解读

## 强制类型转换如何进行
1. visitor模式要求返回Any类型，当我们返回任意类型时，会默认转换为Any类型，等同于调用`(Any)value`
2. 当调用`(Any)value`，会调用`Any.h`中定义的如下构造函数:
```cpp
template<typename U>
Any(U&& value)
  : _ptr(new Derived<StorageType<U>>(std::forward<U>(value))) {
}
```
3. 当把Any类型的value赋值给其他类型的左值时，会调用被重载的`强制类型转换操作符`:
```cpp
template<class U>
operator U() {
  return as<StorageType<U>>();
}
```

## 内存管理如何进行
1. Any支持copy构造函数、move构造函数
2. 真实的value对象，存储于使用`new Derived()`创建的对象中，其指针由Any对象存储
3. 使用value对象创建Any对象时，value是否copy取决于其是否是右值
4. 当调用copy构造函数时，Derived对象会进行copy，value是否copy取决于其是否是右值
5. 当调用转移构造函数时，Derived对象不进行销毁，其指针直接传递给新的Any对象
6. 当销毁Any对象时，Derived对象销毁，value若指向堆则其指向对象不会销毁

## Any对象内value读取的内存管理
1. 当读取Any对象的value时，value会进行copy操作（猜测是为保证Any对象内的value仍然可用）
2. 最好往Any对象中存储指针，以减少拷贝操作
3. 不要往Any中存储unique_ptr，因为unique_ptr不支持copy构造函数，编译会出异常

## Any对象内value的内存管理
> 当使用new创建对象并作为visitor的返回值时，对象的销毁需要自行管理

### 普通对象管理
1. visitor返回值可使用普通指针，保证取值时copy的代价最小
2. 读取visitor返回值后，存储到unique_ptr来进行管理

### 容器对象管理
> vector/array等对象，对象销毁时，其内数据会清空，所以内部如果使用了智能指针，其元素可自动清理内存；如果未使用智能指针，需要手动清除
1. visitor返回值不能使用unique_ptr（比如vector<unique_ptr<A>>），否则在copy容器时会编译出错
2. 容器中存储指针，在读取返回值后，创建一个新的对象，将旧对象使用`unique_ptr`包装后存储，比如
```cpp
vector<ParameterArg *> func_args_src = visitFormalParameterList(ctx->formalParameterList());
vector<unique_ptr<ParameterArg>> func_args;
for (ParameterArg *arg : func_args_src) {
  func_args.push_back(unique_ptr<ParameterArg>(arg));
}
```
