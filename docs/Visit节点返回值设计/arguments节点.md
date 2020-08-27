# arguments节点

## 支持场景
使用arguments的场景如下: 
1. `func_a()`
2. `new A().method_a()`、`a.method_a()`

## 左侧返回值类型
左侧节点为`singleExpression`，子节点可选类型有：
1. `Identifier`: 返回`NodeValue`类型，成员类型为`string`
2. `MemberDot` : 返回`NodeValue`类型，成员类型为`VariableValue(FunctionVariableValue)`
