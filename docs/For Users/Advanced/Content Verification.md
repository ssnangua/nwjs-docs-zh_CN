# 内容验证
---

内容验证，或者叫应用签名，可以阻止正式应用加载非签名文件。提供一个密钥对，`verified_contents.json` 是应用程序文件的签名，由 `sign.py` 工具和私钥（`private_key.pem` 文件）生成，公钥内置于 NW.js 应用中。在应用目录中通过 `nw --verify-content=enforce_strict .` 来运行签名的应用，会正常显示页面。之后尝试对 index.html 文件进行一点修改，再次运行应用，NW 将反馈文件已损坏并立即退出。

!!! note "注意"
    内容验证无法阻止别人黑入您的应用，比如用其他 NW 包来加载您的应用。可以考虑使用 C++ 编写，通过 Node.js 或 NaCl 来加载，或者使用 [nwjc 编译 JS 代码](Protect JavaScript Source Code.md)。

## 给应用签名

使用密钥对给一个签名应用，需要以下步骤：

1. 进入应用目录。
2. 确保 `verified_contents.json` 或 `computed_hashes.json` 不在该目录下（如果存在，移除即可）。
3. 运行 `payload.exe` 生成 `sign.py` 所需的配置文件 `payload.json`。
4. 运行 `python sign.py > /tmp/verified_contents.json`（注意 `tmp` 目录不能是应用所在目录）
5. 将生成的 `verified_contents.json` 文件复制到应用目录，完成签名流程。

## 使用“密钥对”重新构建应用

要使用自己的密钥对，您需要重新构建 NW，并在构建时将命令行参数 `--verify-content=` 设置为 `enforce_strict`。

1. `openssl genrsa -out private_key.pem 2048` 生成密钥对（输出私钥和公钥两个文件）。
2. 运行 `python convertkey.py`，它会将公钥转换为 C 源代码。 
3. 将生成的代码复制到 `content/nw/src/nw_content_verifier_delegate.cc`，替换默认密钥。
4. 修改该文件中第 73 行的命令行参数的值为：
   `Mode experiment_value =  ContentVerifierDelegate::ENFORCE_STRICT;`。
5. 重新构建 NW。

项目仓库的 `tools/sign` 目录中有相关的工具、简单的应用示例和样例私钥，样例私钥匹配官方构建的 NW 程序的公钥。


