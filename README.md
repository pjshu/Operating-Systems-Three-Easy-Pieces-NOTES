如果你想写一个简单的操作系统,可以参考我的项目 [quarkOS](https://github.com/pjshu/quarkOS),我会慢慢补充一些开发资料
😂

仓库并非标准答案,只是作者在阅读操作系统导论时的解答,以及对部分 README 的翻译

本人翻译水平有限,见谅。欢迎指出错误

部分章节课后习题在中文翻译书上是没有的,但在原版上存在 :
- [第十章-多处理器调度](./10.第十章-多处理器调度)
- [第十三章-抽象:地址空间](./13.第十三章-抽象:地址空间)
- [第二十一章-超越物理内存:机制](./21.第二十一章-超越物理内存:机制)
- [第二十七章-插叙:线程 API](./27.第二十七章-插叙:线程API)
- [第二十九章-基于锁的并发数据结构](./29.第二十九章-基于锁的并发数据结构)
- [第三十章-条件变量](./30.第三十章-条件变量)
- [第三十一章-信号量](./31.第三十一章-信号量)
- [第三十二章-常见并发问题](./32.第三十二章-常见并发问题)
- [第三十三章-基于事件的并发机制(进阶)](./33.第三十三章-基于事件的并发机制(进阶))
- [第四十一章-局部性和快速文件系统](./41.第四十一章-局部性和快速文件系统)
- [第四十二章-崩溃一致性:FSCK 和日志文件系统](./42.第四十二章-崩溃一致性:FSCK和日志文件系统)
- [第四十三章-日志结构文件系统](./43.第四十三章-日志结构文件系统)
- [第四十四章-数据完整性和保护](./44.第四十四章-数据完整性和保护)
- [第四十七章-分布式系统](./47.第四十七章-分布式系统)
- [第四十八章-Sun 的网络文件系统](./48.第四十八章-Sun的网络文件系统)


英文原版:
http://pages.cs.wisc.edu/~remzi/OSTEP/

书籍翻译:
https://github.com/remzi-arpacidusseau/ostep-translations

## 代码无法运行
本书代码测试环境为 Intel x64/Ubuntu

gcc 版本为:`gcc version 9.3.0 (Ubuntu 9.3.0-17ubuntu1~20.04)`

Python2 版本:`Python 2.7.18`

Python3 版本: `Python 3.8.5`

第三方 Python3 库: `pip3 install matplotlib numpy`

遇到无法运行的代码请检查是否为版本问题