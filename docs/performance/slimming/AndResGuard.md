# [AndResGuard](https://github.com/shwenzhang/AndResGuard)

## 资源混淆

- resources.arsc

资源索引文件 resources.arsc 需要记录资源文件的名称与路径，使用混淆后的短路径 res/s/a，可以减少整个文件的大小。

- metadata 签名文件

[签名文件 MF 与 SF](https://cloud.tencent.com/developer/article/1354380) 需要记录所有文件的路径以及它们的哈希值，使用短路径可以减少这两个文件的大小。

- ZIP 文件索引

ZIP 文件格式里面也需要记录每个文件 Entry 的路径、压缩算法、CRC、文件大小等信息。使用短路径，本身就可以减少记录文件路径的字符串大小。

## 极限压缩

- 更高的压缩率

7-Zip 的大字典优化，APK 的整体压缩率可以提升 3% 左右。

- 压缩更多的文件

支持针对 resources.arsc、PNG、JPG 以及 GIF 等文件的强制压缩。
