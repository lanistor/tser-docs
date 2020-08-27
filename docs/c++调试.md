# c++调试

## Mac自带工具调试
在.vscode/launch.json中添加配置:
```
{
  "name": "Debug",
  "type": "cppdbg",
  "request": "launch",
  "program": "${workspaceRoot}/build/main.o",
  "args": [],
  "stopAtEntry": false,
  "cwd": "${workspaceFolder}",
  "environment": [],
  "externalConsole": false,
  "MIMode": "lldb"
}
```
如果运行有问题，尝试添加: `"miDebuggerPath": "/Users/<yourname>/lldb-mi/usr/local/bin/lldb-mi"`
如果还有问题，使用`CodeLLDB插件调试`

## CodeLLDB插件调试
1. 下载VSCode插件CodeLLDB
2. 在.vscode/launch.json中添加配置:
```
{
  "name": "Debug",
  "type": "lldb",
  "request": "launch",
  "program": "${workspaceRoot}/build/tser",
  "args": [
    "input/step/step1.ts"
  ],
}
```