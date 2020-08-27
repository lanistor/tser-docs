# Antlr返回值处理
> 推荐使用unique_ptr，这样对象内存可自动销毁

## 处理普通class数据
使用示例：
```c++
antlrcpp::Any test() {
  auto v3 = make_unique<NodeValue>();
  return v3;
}

int main() {
  auto a = test();
  auto c = move(a.as<unique_ptr<NodeValue>>());
  c->value;
  return 0;
}
```
### 注意事项
1. `Any`转换为实际类型时一定要使用`move`，否则会默认调用`unique_ptr`的拷贝构造函数，而`unique_ptr`默认删除了拷贝构造函数，因而编译会出问题
2. 不能使用强制类型转换，而应该使用`Any::as<T>()`函数，错误原因也是由于拷贝构造函数不存在。如下写法编译无法通过:
```c++
auto c = (unique_ptr<NodeValue>)(move(a));
```

## 处理vector数据
### 使用栈内存拷贝
> vector大小是有24个字节，使用栈内存即可

以下是使用示例：
```c++
antlrcpp::Any test() {
  vector<unique_ptr<NodeValue>> vc;
  vc.push_back(make_unique<NodeValue>());
  vc.push_back(make_unique<NodeValue>());
  return vc;
}

int main() {
  auto a = test();
  auto c = move(a.as<vector<unique_ptr<NodeValue>>>());
  // 一定要加'&'
  for (auto &cc : c) {
    cout << cc->string_value << endl;
  }
  return 0;
}
```
### 使用堆内存
> 不推荐使用堆内存

以下是使用示例：
```c++
// 一定要返回'unique_ptr'
antlrcpp::Any test() {
  auto vc = new vector<unique_ptr<NodeValue>>();
  vc->push_back(make_unique<NodeValue>());
  vc->push_back(make_unique<NodeValue>());
  return unique_ptr<vector<unique_ptr<NodeValue>>>(vc);
}

int main() {
  auto a = test();
  auto c = move(a.as<unique_ptr<vector<unique_ptr<NodeValue>>>>());
  for (auto &cc : *c) {
    cout << cc->string_value << endl;
  }
  cout << "sizeof:" << sizeof(c) << endl;
  return 0;
}
```