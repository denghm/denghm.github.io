---
layout: article
title: Python的Import与JavaScript的import
modified: 2020-05-13
author: Voyager
toc: false
comments: true
categories: technology
summary: 比较不同的importd的区别
---

最近在编码过程中，使用了python与javascript，因为语法类似，而且对这两种语言我都没有专门花时间学习，都是边用边学，所以很容易就混淆了。其中import的语法特别类似，把区别总结如下

## Python的Import
### 语法
#### import
每次只导入包里的特定模块，例如
```python
import sound.efforts.echo 
```
这样就导入了 sound.effects.echo 子模块。它必须通过完整的名称来引用
```python
sound.effects.echo.echofilter(input, output, delay=0.7, atten=4) 
```

#### from... import
加载了模块, 例如
```python
from sound.effects import echo
```
这样就加载了 echo 子模块，并且使得它在没有包前缀的情况下也可以使用，可以如下方式调用
```python
echo.echofilter(input, output, delay=0.7, atten=4) 
```
还有另一种变体用于直接导入函数，变量或对象, 例如
```python
from model.dao import Dao
```
总之，采用from package import item    方式导入包时，这个子项item既可以是子包也可以是其他命名，如函数、类、变量等

#### import *
import * 这样的语句理论上是希望文件系统找出包中所有的子模块，然后导入它们。这可能会花长时间，并出现边界效应等。Python 解决方案是提供一个明确的包索引。

这个索引由\_\_init\_\_.py  定义 \_\_all\_\_ 变量，该变量为一列表，如上例 sound/effects 下的 \_\_init\_\_.py 中，可定义  \_\_all\_\_ = ["echo","surround","reverse"] 

这意味着，  from sound.effects import *   会从对应的包中导入以上三个子模块； 尽管提供 import * 的方法，仍不建议在生产代码中使用这种写法。

#### 包内引用
如果是子包内的引用，可以按相对位置引入子模块 以 echo 模块为例，可以引用如下
```python
from . import reverse              # 同级目录 导入 reverse
from .. import frormats            # 上级目录 导入 frormats
from ..filters import equalizer    # 上级目录的filters模块下 导入 equalizer
```

### import的对象
import的对象是模块，而模块包括以下类型
* 模块文件（.py文件）
* C或C++扩展（已编译为共享库或DLL文件）
* 包（包含多个模块）
* 内建模块（使用C编写并已链接到Python解释器

#### 包
包就是文件夹，但该文件夹下必须存在 \_\_init\_\_.py 文件
通常 \_\_init\_\_.py文件为空，但是我们还可以为它增加其他的功能。我们在导入一个包时，实际上是导入了它的\_\_init\_\_.py文件。这样我们可以在\_\_init\_\_.py文件中批量导入我们所需要的模块，而不再需要一个一个的导入。

```python
# package
# __init__.py
import re
import urllib
import sys
import os
```

### 搜索路径
当导入模块时，解释器按照sys.path列表中的目录顺序来查找导入文件。IDE中，因为一般设置了

>add content root to PYTHONPATH

所以用根目录开始的目录作为包名没有问题

### import的逻辑
模块通常为单独的.py文件，可以用import直接引用，可以作为模块的文件类型有.py、.pyo、.pyc、.pyd、.so、.dll

在导入模块时，解释器做以下工作：

* 已导入模块的名称创建新的命名空间，通过该命名空间就可以访问导入模块的属性和方法。
* 在新创建的命名空间中**执行**源代码文件。
* 创建一个名为源代码文件的对象，该对象引用模块的名字空间，这样就可以通过这个对象访问模块中的函数及变量

import 语句可以在程序的任何位置使用，你可以在程序中多次导入同一个模块，但**模块中的代码仅仅在该模块被首次导入时执行**。后面的import语句只是简单的创建一个到模块名字空间的引用而已。


>**关于.pyc 文件 与 .pyo 文件**
.py文件的汇编,只有在import语句执行时进行，当.py文件第一次被导入时，它会被汇编为字节代码，并将字节码写入同名的.pyc文件中。后来每次导入操作都会直接执行.pyc 文件（当.py文件的修改时间发生改变，这样会生成新的.pyc文件），在解释器使用-O选项时，将使用同名的.pyo文件，这个文件去掉了断言（assert）、断行号以及其他调试信息，体积更小，运行更快。（使用-OO选项，生成的.pyo文件会忽略文档信息）



## JavaScript的import


### export导出模块
不同于Python的模块，javascript的模块必须要导出才能使用。export有两种模块导出方式：**命名式导出（名称导出）**和 **默认导出（定义式导出）**，命名式导出每个模块可以多个，而默认导出每个模块仅一个。

exports变量与module.exports

### 模块的查找路径
import后面的from指定模块文件的位置，可以是相对路径，也可以是绝对路径，.js后缀可以省略。如果只是模块名，不带有路径，那么必须有配置文件，告诉 JavaScript 引擎该模块的位置。
根据模块引用是相对的还是非相对的，模块导入会以不同的方式解析。

#### 相对导入和非相对导入
相对导入是以/，./或../开头的。 下面是一些例子：

```javascript
import Entry from "./components/Entry";
import { DefaultHeaders } from "../constants/http";
import "/mod";
```
所有其它形式的导入被当作非相对的。 下面是一些例子：

```javascript
import * as $ from "jQuery";
import { Component } from "@angular/core";
```

相对导入在解析时是相对于导入它的文件，并且不能解析为一个外部模块声明。 你应该为你自己写的模块使用相对导入，这样能确保它们在运行时的相对位置。

非相对模块的导入可以相对于baseUrl或通过下文会讲到的路径映射来进行解析。 它们还可以被解析成 外部模块声明。 使用非相对路径来导入你的外部依赖。

### import

import是静态执行，这与python不同, 带来的影响有2个
1) import命令具有提升效果，会提升到整个模块的头部，首先执行。
2) 不能使用表达式和变量，这些只有在运行时才能得到结果的语法结构。

### import()

|  | Python | JavaScript |
| ------ | ------ | ------ |
|语法 | import | 稍微长一点的文本 |
|  | 短文本 | 中等文本 |