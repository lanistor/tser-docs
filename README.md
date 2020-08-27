# 虚拟机开发

## 环境配置

### Antlr
1. 配置`ANTLR_RUNTIME`，指向antlr的运行时（包含头文件，静态链接库）  
  如: `/C/P/compile/antlr/runtime`
2. 配置Antlr工具
```bash
alias antlr4='java -jar /usr/local/lib/antlr-4.8-complete.jar'
alias grun='java org.antlr.v4.gui.TestRig'
export ANTLR_EXECUTABLE=/usr/local/lib/antlr-4.8-complete.jar
export CLASSPATH=".:/usr/local/lib/antlr-4.8-complete.jar:$CLASSPATH"
```

### 编译LLVM

### 下载LLVM仓库
> [llvm-project](https://github.com/llvm-project/llvm)

### 编译LLVM
> [getting-started-with-llvm](https://llvm.org/docs/GettingStarted.html#getting-started-with-llvm)

1. `cd llvm-project && mkdir build && cd build`
2. `cmake -G Ninja -DCMAKE_BUILD_TYPE=Release ../llvm`
3. `cmake --build .`
4. `ninja check-all`

#### 重要参数解析
- `-DLLVM_ENABLE_PROJECTS="clang;libcxx;libcxxabi"`
- `-DCMAKE_BUILD_TYPE=`: Debug, Release, RelWithDebInfo, and MinSizeRel

### 配置LLVM
- 配置`LLVM`与`Clang`，如:
```bash
export CC=/usr/bin/clang
export CXX=/usr/bin/clang++
export PATH="/Users/vifird/C/compile/llvm-project/build/bin:$PATH"
```

## 资源
### antlr runtime
  - `https://www.antlr.org/download/`

## 开发规范
- 命名规范  
  - 参考：`https://zh-google-styleguide.readthedocs.io/en/latest/google-cpp-styleguide/naming/#id3`
