# c++ link

## 链接单个函数
```cpp
#include "llvm/ExecutionEngine/Orc/Core.h"
auto jit =
    ExitOnErr(LLJITBuilder()
      .setJITTargetMachineBuilder(std::move(JTMB))
      .create());

MangleAndInterner Mangle(jit->getExecutionSession(), jit->getDataLayout());
SymbolMap ffi_funcs;
ffi_funcs[ Mangle("add") ] =
    JITEvaluatedSymbol(pointerToJITTargetAddress(&add), JITSymbolFlags::Exported);
jit->getMainJITDylib().define(absoluteSymbols(ffi_funcs));
```

## 链接静态库、目标文件
`clang++ ffi.cpp -c -o build/ffi`
```cpp
auto jit =
    ExitOnErr(LLJITBuilder()
                  .setJITTargetMachineBuilder(std::move(JTMB))
                  // 这句一定要加，不然会报link异常
                  .setObjectLinkingLayerCreator([&](ExecutionSession &ES, const Triple &TT) {
                    return make_unique<ObjectLinkingLayer>(
                        ES, make_unique<jitlink::InProcessMemoryManager>());
                  })
                  .create());

auto ffi = ExitOnErr(errorOrToExpected(MemoryBuffer::getFileAsStream("build/ffi")));
jit->addObjectFile(std::move(ffi));
```

## 链接动态库
```cpp
#include "llvm/ExecutionEngine/JITSymbol.h"

char ffi_file[] = "build/ffi.dylib";
jit->getMainJITDylib().addGenerator(ExitOnErr(
    DynamicLibrarySearchGenerator::Load(ffi_file, jit->getDataLayout().getGlobalPrefix())));
```