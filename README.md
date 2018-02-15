
# CTF Wiki

[![Build Status](https://travis-ci.org/ctf-wiki/ctf-wiki.svg?branch=master)](https://travis-ci.org/ctf-wiki/ctf-wiki)
[![Requirements Status](https://requires.io/github/ctf-wiki/ctf-wiki/requirements.svg?branch=master)](https://requires.io/github/ctf-wiki/ctf-wiki/requirements/?branch=master)
[![BearyChat](https://img.shields.io/badge/bearychat-join_chat-green.svg)](https://ctf-wiki.bearychat.com)

欢迎来到 **CTF Wiki**。

**CTF**（Capture The Flag，夺旗赛）起源于 1996 年 **DEFCON** 全球黑客大会，是网络安全爱好者之间的 **Game**。

在最初学习 **CTF** 时，我们往往会搜索大量资料，然而这些资料的层次性并不是很良好，这使得我们在入门学习的过程中，较为吃力。为了能够使得更多热爱 **CTF** 的小伙伴们更快地入门 **CTF**，**CTF Wiki** 应运而生。

**CTF Wiki** 是一个**自由**的站点，主要包含了 **CTF** 中的**基础知识、常见题型、解题思路以及常用工具等**，希望可以帮助你更快地了解 **CTF** 竞赛以及网络安全相关知识。

当然，**CTF Wiki** 并不受限于基础知识，在不断完善的过程中，也正在不断扩充 CTF 中进阶知识。

## 我能收获什么？

* 一个不一样的思考方式以及一颗乐于解决问题的心
* 锻炼你的快速学习能力，不断学习新事物
* 一些有趣的安全技术与相应的挑战
* 一段充实奋斗的时光

在阅读 Wiki 之前，我们希望能给予你几点建议：

* 至少掌握一门编程语言，比如 Python
* 阅读短文 [提问的智慧](http://www.jianshu.com/p/60dd8e9cd12f)
* 善用 Google 搜索能帮助你更好地提升自己
* 动手实践比什么都要管用
* 保持对技术的好奇与渴望并坚持下去

> 世界很大，互联网让世界变小，真的黑客们应该去思考并创造，无论当下是在破坏还是在创造，记住，未来，那条主线是创造的就对了。 ——by 余弦

安全圈很小，安全的海洋很深。安全之路的探险，不如就从 **CTF Wiki** 开始！

## How to build？

本文档目前采用 [mkdocs](https://github.com/mkdocs/mkdocs) 部署在 https://ctf-wiki.github.io/ctf-wiki/。当然本文档也可以部署在本地，具体方式如下

### 安装依赖

```shell
# mkdocs
pip install mkdocs
# extensions
pip install pymdown-extensions
# theme
pip install mkdocs-material
```

### 本地部署

```shell
# generate static file in site/
mkdocs build
# deploy at http://127.0.0.1:8000
mkdocs serve
```

**mkdocs 本地部署的网站是动态更新的，即当你修改 md 文件时，网页也会尽可能的动态更新。**

## 想要帮助 Wiki 更加完善？

我们非常欢迎你为 Wiki 编写内容，将自己的所学所得与大家分享，具体的贡献方式请参见 [CONTRIBUTING](.github/CONTRIBUTING.md)。 

**在你决定要贡献内容之前，请你务必看完这些内容**。我们期待着你的加入。
