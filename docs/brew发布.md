# brew 发布

## 发布
1. 压缩文件
`$ pushd ./dist && tar -cJvf ../packages/mac64_0.0.2.tar.xz ./ && popd`

2. 上传release包到[homebrew-tser](https://github.com/tser-project/tser/releases)
3. 删除旧Formula脚本
```bash
$ rm -rf /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/Formula/tser.rb
```
4. 创建Formula脚本
使用新Release包地址创建Formula脚本
```bash
$ brew create https://github.com/tser-project/tser/releases/download/v0.0.2/mac64_0.0.2.tar.xz
```  

这条命令会生成tser的Formula脚本，修改脚本使其内容如下:
```rb
class Tser < Formula
  desc "A TypeScript virtual machine."
  homepage "https://github.com/tser-project/tser"
  version "0.0.2"
  url "https://github.com/tser-project/tser/releases/download/v#{version}/mac64_#{version}.tar.xz"
  sha256 ""

  def install
    bin.install "bin/tser"
    lib.install Dir["lib/*"]
  end
end
```

4. 将`version`和`sha256`的值拷贝到[homebrew-tser/tser.rb](https://github.com/tser-project/homebrew-tser/blob/master/tser.rb)文件，然后更新到`master`分支

5. 执行本地测试
  ```bash
  $ brew install tser
  $ tser ./input.ts
  ```

6. 测试通过后，删除本地安装和Formula脚本
```bash
$ brew uninstall tser
$ rm -rf /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/Formula/tser.rb
```

7. 测试
```bash
$ brew tap tser-project/tser && brew install tser
$ tser ./input.ts
```
