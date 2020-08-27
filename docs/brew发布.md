# brew 发布

## 发布
1. 压缩文件
`$ pushd ./dist && tar -cJvf ../packages/tser_0.0.1-beta.0.tar.xz ./ && popd`

2. 上次release包到[homebrew-tser](https://github.com/tser-project/homebrew-tser/releases)
3. 删除旧Formula脚本
```bash
$ rm -rf /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/Formula/tser.rb
```
4. 创建Formula脚本
使用新Release包地址创建Formula脚本
```bash
$ brew create https://github.com/tser-project/tser/releases/download/0.0.1-beta.0/tser_0.0.1-beta.0.tar.xz
```  

这条命令会生成tser的Formula脚本，脚本内容如下:
```rb
class Tser < Formula
  desc "A real TypeScript virtual machine."
  homepage "https://github.com/tser-project/tser"
  version "0.0.1-beta.0"
  url "https://github.com/tser-project/tser/releases/download/#{version}/tser_#{version}.tar.xz"
  sha256 ""

  def install
    bin.install "bin/tser"
    lib.install Dir["lib/*"]
  end
end
```

4. 将`version`和`sha256`的值拷贝到[homebrew-tser](https://github.com/tser-project/homebrew-tser)下的`tser.rb`文件，然后更新到`master`分支

5. 再次删除本地Formula脚本
```bash
$ rm -rf /usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/Formula/tser.rb
```

6. 测试
    ```bash
    $ brew uninstall tser
    $ brew tap tser-project/tser && brew install tser
    $ tser ./input.ts
    ```
