# Visit节点返回值
使用Antlr的visit模式访问节点时，返回值可以是任意类型。

## 可使用返回类型
1. `unique_ptr<NodeValue>`: 通用返回类型，内部有具体类型和依赖信息
2. `TypeSignInfo *`: 用于`typeSign`节点的返回
3. `vector<unique_ptr<NodeValue>>` : 用于`arguments`节点返回