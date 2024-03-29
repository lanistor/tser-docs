# BlockScope生成IR设计
`BlockScope`通常与`指令分支块`对应，在LLVM中对应`br label`指令，分支结束后，要跳转到正确的目标分支（通常为主干分支），这个设计还比较简单，但当和`continue`、`break`、`return`语句结合后，会变得复杂一些，不做特殊处理，会在一个代码块中出现多个`br label`指令。

## 为什么会出现多个`br label`指令
1. 分支正常结束后，需要跳转到正确的主干分支，因此分支末尾，一定要插入`br lable [thunk]`来跳转到目标分支；为实现此目的，我们在每个分支块的末尾，会自动插入`br lable [thunk]`指令
2. `continue`、`break`、`return`指令，都会立即结束当前指令分支块，并需要正确跳转到目标分支块

这样如果一个分支块中有上述3个指令，便会出现2个`br lable`的情况

## 如何避免出现多个`br label`指令
1. 在含有`continue`、`break`、`return`指令的代码中，标记已经有`跳转分支指令`，然后在块访问结束时，检测是否已经有`跳转分支指令`，若有则不添加。
2. `跳转分支指令`需要清除地知道自己要跳转的分支，因此需要记录`主干分支`(`if`语句使用)、`条件分支`(`for`, `while`语句使用)
3. `指令分支块`与`BlockScope`绑定，将上述内容记录到`BlockScope`中

## 特殊的If语句
> 我们会对if语句绑定一个`BlockScope`，可以绑定在If上，也可以绑定在`if -> statemen`上，基于语法规范，我们绑定在`if -> statemen`上。

`If`比较特殊，因为`else`语句有`else if`和`else`两种，如果直接将`BlockScope`绑定在`else`的`statement`上，当`statement`是另一个`IfStatement`（即`else if`场景）时，进入到子`IfStatement`会再次新建`BlockScope`，这时会出现两个`BlockScope`，但其中一个是无用的。
为了避免无用的`BlockScope`，我们在可以首先判断下`else`的`statement`是否是`IfStatement`，如果是，则不新建`BlockScope`

## For指令
正常的for指令，由4部分都成: `初始化指令`、`条件指令`、`内容块指令`、`后置指令`，还有最终的`主干代码块`。

正常的访问和分支跳转逻辑为: `初始化指令`->`条件指令`->`内容块指令`/`主干代码块`->`后置指令`->`条件指令`。

但在for指令中，所有的部分都是可以没有的，所以逻辑会变得复杂一些。

### 初始化指令
1. 若有，则直接访问
2. 若没有，则直接跳过。

### 条件指令
1. 若有，则根据条件分别跳转到`内容块指令`或`主干代码块`
2. 若没有，则直接跳转`内容块指令`

### 内容块指令
1. 若有，则访问，并在结束时跳转到`后置指令`
2. 若没有，则默认新建，内容为空，只留跳转指令

### 后置指令
1. 若有，则访问，并在结束时跳转到`条件指令`
2. 若没有，则跳过

### 访问设计
> 逻辑较为复杂，对所有组合做枚举难度较大，我们设计一种简化的处理流程：当自身块不存在时，直接指向自己要跳转到的下一个指令块

1. 先处理`初始化指令`、`主干代码块`（必须有）
2. `内容块指令`是必须有的，可以直接创建`内容块指令`
3. `条件指令`若不存在，则直接指向`内容块指令`
4. `后置指令`若不存在，则直接指向`条件指令`

这样设计，若`条件指令`或`后置指令`任何一个不存在，逻辑仍然可以按顺序进行
