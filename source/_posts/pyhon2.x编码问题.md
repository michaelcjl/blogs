---
title: pyhon2.x 编码问题
date: 
tags: pyhon2.x,编码
categories: python

---
# pyhon2.x 编码问题
版本py2.x
其实开始也没太注意python2的编码问题，直到写代码调试的时候出现了如下报错：
> Traceback (most recent call last):
File "<stdin>", line 1, in <module>
UnicodeDecodeError: 'ascii' codec can't decode byte 0xe6 in position 0: ordinal not in range(128)

网上很快就找到了解决方法。但还是想把这个问题整明白。于是网上搜了下，研究一番，整理一下。

##   [1.什么是字符编码？](https://zh.wikipedia.org/wiki/%E5%AD%97%E7%AC%A6%E7%BC%96%E7%A0%81)
字符编码（英语：Character encoding）、字集码是把字符集中的字符编码为指定集合中某一对象（例如：比特模式、自然数序列、8位组或者电脉冲），以便文本在计算机中存储和通过通信网络的传递。常见的例子包括将拉丁字母表编码成摩斯电码和ASCII。其中，ASCII将字母、数字和其它符号编号，并用7比特的二进制来表示这个整数。通常会额外使用一个扩充的比特，以便于以1个字节的方式存储。

[字符编码笔记：ASCII，Unicode和UTF-8](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html) 

##  2.str 和 unicode

 str和unicode都是basestring的子类,所以有判断是否是字符串的方法
 

    def is_str(s):
    return isinstance(s, basestring)

 str和unicode 转换
 

    str  -> decode('the_coding_of_str') -> unicode
    unicode -> encode('the_coding_you_want') -> str
    
str是字节串，由unicode经过编码(encode)后的字节组成的
***examlpe for str :***
声明方式
```
s = '中文'
s = u'中文'.encode('utf-8')

>>> type('中文')
<type 'str'>
```
求长度(返回字节数)
```
>>> u'中文'.encode('utf-8')
'\xe4\xb8\xad\xe6\x96\x87'
>>> len(u'中文'.encode('utf-8'))
6
```
所以，unicode才是真正意义上的字符串，由字符组成


----------

***examlpe for unicode :***
声明方式
```
s = u'中文'
s = '中文'.decode('utf-8')
s = unicode('中文', 'utf-8')

>>> type(u'中文')
<type 'unicode'>
```  
求长度(返回字符数),在逻辑中真正想要用的
``` 
>>> u'中文'
u'\u4e2d\u6587'
>>> len(u'中文')
2
``` 
**搞明白要处理的是str还是unicode,使用对的处理方法(str.decode/unicode.encode)**

下面是判断是否为unicode/str的方法
``` 
>>> isinstance(u'中文', unicode)
True
>>> isinstance('中文', unicode)
False

>>> isinstance('中文', str)
True
>>> isinstance(u'中文', str)
False
``` 
**简单原则：不要对str使用encode，不要对unicode使用decode** 
**不同编码转换,使用unicode作为中间编码**
``` 
#s是code_A的str
s.decode('code_A').encode('code_B')
``` 
##  3.文件处理,IDE和控制台

 -  处理流程，可以这么使用，把python看做一个水池，一个入口，一个出口

入口处，全部转成unicode,池里全部使用unicode处理，出口处，再转成目标编码(当然，有例外，处理逻辑中要用到具体编码的情况)

>  读文件

>  外部输入编码，decode转成unicode

>  处理(内部编码，统一unicode)

>  encode转成需要的目标编码

>  写到目标输出(文件或控制台)

IDE和控制台报错，原因是print时，编码和IDE自身编码不一致导致
输出时将编码转换成一致的就可以正常输出
``` 
>>> print u'中文'.encode('gbk')
����
>>> print u'中文'.encode('utf-8')
中文
``` 
suggestion:

 -  规范编码
>  统一编码，防止由于某个环节产生的乱码
    环境编码，IDE/文本编辑器, 文件编码，数据库数据表编码

 -  源文件编码
>  这个很重要。py文件默认编码是ASCII,在源代码文件中，如果用到非ASCII字符，需要在文件头部进行编码声明 文档不声明的话，输入非ASCII会遇到的错误,必须放在文件第一行或第二行 

声明方法
```
# -*- coding: utf-8 -*-
或者
#coding=utf-8
```
若头部声明coding=utf-8, a = '中文' 其编码为utf-8

若头部声明coding=gb2312, a = '中文' 其编码为gbk

so, 同一项目中所有源文件头部统一一个编码,并且声明的编码要和源文件保存的编码一致(编辑器相关)

 - 在源代码用作处理的硬编码字符串，统一用unicode
  >  将其类型和源文件本身的编码隔离开, 独立无依赖方便流程中各个位置处理

    ```
    if s == u'中文':  #而不是 s == '中文'
        pass
    #注意这里 s到这里时，确保转为unicode
    ```

以上几步搞定后，你只需要关注两个 unicode和 你设定的编码(一般使用utf-8)

 -  处理顺序
       Decode early
       Unicode everywhere
       Encode later

##  4.相关模块及一些方法
 >  获得和设置系统默认编码
```
>>> import sys
>>> sys.getdefaultencoding()
'ascii'

>>> reload(sys)
<module 'sys' (built-in)>
>>> sys.setdefaultencoding('utf-8')
>>> sys.getdefaultencoding()
'utf-8'
``` 
>  str.encode('other_coding')

在python中，直接将某种编码的str进行encode成另一种编码str
``` 
#str_A为utf-8
str_A.encode('gbk')

执行的操作是
str_A.decode('sys_codec').encode('gbk')
这里sys_codec即为上一步 sys.getdefaultencoding() 的编码
``` 
'获得和设置系统默认编码'和这里的str.encode是相关的，但我一般很少这么用，主要是觉得复杂不可控,还是输入明确decode，输出明确encode来得简单些(个人观点)

>   chardet

文件编码检测
``` 
>>> import chardet
>>> f = open('test.txt','r')
>>> result = chardet.detect(f.read())
>>> result
{'confidence': 0.99, 'encoding': 'utf-8'}
``` 
>   \u字符串转对应unicode字符串
``` 
>>> u'中'
u'\u4e2d'

>>> s = '\u4e2d'
>>> print s.decode('unicode_escape')
中

>>> a = '\\u4fee\\u6539\\u8282\\u70b9\\u72b6\\u6001\\u6210\\u529f'
>>> a.decode('unicode_escape')
u'\u4fee\u6539\u8282\u70b9\u72b6\u6001\u6210\u529f'
``` 


----------


**最后来一个栗子，python 获取中文网页乱码转换**
首先需要重置页面编码：
``` 
import sys
reload(sys)
sys.setdefaultencoding('utf-8')
``` 
爬虫原代码是：
```
res = requests.get(url, timeout=60).text
``` 
获取到的页面源码是乱码，各种转码都不行。
换成：
``` 
res = requests.get(url, timeout=60).content
``` 
把text换成content以后，做res.decode('gbk').encode('utf-8')，终于正常。


我们理解您需要更便捷更高效的工具记录思想，整理笔记、知识，并将其中承载的价值传播给他人，**Cmd Markdown** 是我们给出的答案 —— 我们为记录思想和分享知识提供更专业的工具。 您可以使用 Cmd Markdown：

> * 整理知识，学习笔记
> * 发布日记，杂文，所见所想
> * 撰写发布技术文稿（代码支持）
> * 撰写发布学术论文（LaTeX 公式支持）

![cmd-markdown-logo](https://www.zybuluo.com/static/img/logo.png)

除了您现在看到的这个 Cmd Markdown 在线版本，您还可以前往以下网址下载：

### [Windows/Mac/Linux 全平台客户端](https://www.zybuluo.com/cmd/)

> 请保留此份 Cmd Markdown 的欢迎稿兼使用说明，如需撰写新稿件，点击顶部工具栏右侧的 <i class="icon-file"></i> **新文稿** 或者使用快捷键 `Ctrl+Alt+N`。

------

## 什么是 Markdown

Markdown 是一种方便记忆、书写的纯文本标记语言，用户可以使用这些标记符号以最小的输入代价生成极富表现力的文档：譬如您正在阅读的这份文档。它使用简单的符号标记不同的标题，分割不同的段落，**粗体** 或者 *斜体* 某些文字，更棒的是，它还可以

### 1. 制作一份待办事宜 [Todo 列表](https://www.zybuluo.com/mdeditor?url=https://www.zybuluo.com/static/editor/md-help.markdown#13-待办事宜-todo-列表)

- [ ] 支持以 PDF 格式导出文稿
- [ ] 改进 Cmd 渲染算法，使用局部渲染技术提高渲染效率
- [x] 新增 Todo 列表功能
- [x] 修复 LaTex 公式渲染问题
- [x] 新增 LaTex 公式编号功能

### 2. 书写一个质能守恒公式[^LaTeX]

$$E=mc^2$$

### 3. 高亮一段代码[^code]

```python
@requires_authorization
class SomeClass:
    pass

if __name__ == '__main__':
    # A comment
    print 'hello world'
```

### 4. 高效绘制 [流程图](https://www.zybuluo.com/mdeditor?url=https://www.zybuluo.com/static/editor/md-help.markdown#7-流程图)

```flow
st=>start: Start
op=>operation: Your Operation
cond=>condition: Yes or No?
e=>end

st->op->cond
cond(yes)->e
cond(no)->op
```

### 5. 高效绘制 [序列图](https://www.zybuluo.com/mdeditor?url=https://www.zybuluo.com/static/editor/md-help.markdown#8-序列图)

```seq
Alice->Bob: Hello Bob, how are you?
Note right of Bob: Bob thinks
Bob-->Alice: I am good thanks!
```

### 6. 绘制表格

| 项目        | 价格   |  数量  |
| --------   | -----:  | :----:  |
| 计算机     | \$1600 |   5     |
| 手机        |   \$12   |   12   |
| 管线        |    \$1    |  234  |

### 7. 更详细语法说明

想要查看更详细的语法说明，可以参考我们准备的 [Cmd Markdown 简明语法手册][1]，进阶用户可以参考 [Cmd Markdown 高阶语法手册][2] 了解更多高级功能。

总而言之，不同于其它 *所见即所得* 的编辑器：你只需使用键盘专注于书写文本内容，就可以生成印刷级的排版格式，省却在键盘和工具栏之间来回切换，调整内容和格式的麻烦。**Markdown 在流畅的书写和印刷级的阅读体验之间找到了平衡。** 目前它已经成为世界上最大的技术分享网站 GitHub 和 技术问答网站 StackOverFlow 的御用书写格式。

---

## 什么是 Cmd Markdown

您可以使用很多工具书写 Markdown，但是 Cmd Markdown 是这个星球上我们已知的、最好的 Markdown 工具——没有之一 ：）因为深信文字的力量，所以我们和你一样，对流畅书写，分享思想和知识，以及阅读体验有极致的追求，我们把对于这些诉求的回应整合在 Cmd Markdown，并且一次，两次，三次，乃至无数次地提升这个工具的体验，最终将它演化成一个 **编辑/发布/阅读** Markdown 的在线平台——您可以在任何地方，任何系统/设备上管理这里的文字。

### 1. 实时同步预览

我们将 Cmd Markdown 的主界面一分为二，左边为**编辑区**，右边为**预览区**，在编辑区的操作会实时地渲染到预览区方便查看最终的版面效果，并且如果你在其中一个区拖动滚动条，我们有一个巧妙的算法把另一个区的滚动条同步到等价的位置，超酷！

### 2. 编辑工具栏

也许您还是一个 Markdown 语法的新手，在您完全熟悉它之前，我们在 **编辑区** 的顶部放置了一个如下图所示的工具栏，您可以使用鼠标在工具栏上调整格式，不过我们仍旧鼓励你使用键盘标记格式，提高书写的流畅度。

![tool-editor](https://www.zybuluo.com/static/img/toolbar-editor.png)

### 3. 编辑模式

完全心无旁骛的方式编辑文字：点击 **编辑工具栏** 最右测的拉伸按钮或者按下 `Ctrl + M`，将 Cmd Markdown 切换到独立的编辑模式，这是一个极度简洁的写作环境，所有可能会引起分心的元素都已经被挪除，超清爽！

### 4. 实时的云端文稿

为了保障数据安全，Cmd Markdown 会将您每一次击键的内容保存至云端，同时在 **编辑工具栏** 的最右侧提示 `已保存` 的字样。无需担心浏览器崩溃，机器掉电或者地震，海啸——在编辑的过程中随时关闭浏览器或者机器，下一次回到 Cmd Markdown 的时候继续写作。

### 5. 离线模式

在网络环境不稳定的情况下记录文字一样很安全！在您写作的时候，如果电脑突然失去网络连接，Cmd Markdown 会智能切换至离线模式，将您后续键入的文字保存在本地，直到网络恢复再将他们传送至云端，即使在网络恢复前关闭浏览器或者电脑，一样没有问题，等到下次开启 Cmd Markdown 的时候，她会提醒您将离线保存的文字传送至云端。简而言之，我们尽最大的努力保障您文字的安全。

### 6. 管理工具栏

为了便于管理您的文稿，在 **预览区** 的顶部放置了如下所示的 **管理工具栏**：

![tool-manager](https://www.zybuluo.com/static/img/toolbar-manager.jpg)

通过管理工具栏可以：

<i class="icon-share"></i> 发布：将当前的文稿生成固定链接，在网络上发布，分享
<i class="icon-file"></i> 新建：开始撰写一篇新的文稿
<i class="icon-trash"></i> 删除：删除当前的文稿
<i class="icon-cloud"></i> 导出：将当前的文稿转化为 Markdown 文本或者 Html 格式，并导出到本地
<i class="icon-reorder"></i> 列表：所有新增和过往的文稿都可以在这里查看、操作
<i class="icon-pencil"></i> 模式：切换 普通/Vim/Emacs 编辑模式

### 7. 阅读工具栏

![tool-manager](https://www.zybuluo.com/static/img/toolbar-reader.jpg)

通过 **预览区** 右上角的 **阅读工具栏**，可以查看当前文稿的目录并增强阅读体验。

工具栏上的五个图标依次为：

<i class="icon-list"></i> 目录：快速导航当前文稿的目录结构以跳转到感兴趣的段落
<i class="icon-chevron-sign-left"></i> 视图：互换左边编辑区和右边预览区的位置
<i class="icon-adjust"></i> 主题：内置了黑白两种模式的主题，试试 **黑色主题**，超炫！
<i class="icon-desktop"></i> 阅读：心无旁骛的阅读模式提供超一流的阅读体验
<i class="icon-fullscreen"></i> 全屏：简洁，简洁，再简洁，一个完全沉浸式的写作和阅读环境

### 8. 阅读模式

在 **阅读工具栏** 点击 <i class="icon-desktop"></i> 或者按下 `Ctrl+Alt+M` 随即进入独立的阅读模式界面，我们在版面渲染上的每一个细节：字体，字号，行间距，前背景色都倾注了大量的时间，努力提升阅读的体验和品质。

### 9. 标签、分类和搜索

在编辑区任意行首位置输入以下格式的文字可以标签当前文档：

标签： 未分类

标签以后的文稿在【文件列表】（Ctrl+Alt+F）里会按照标签分类，用户可以同时使用键盘或者鼠标浏览查看，或者在【文件列表】的搜索文本框内搜索标题关键字过滤文稿，如下图所示：

![file-list](https://www.zybuluo.com/static/img/file-list.png)

### 10. 文稿发布和分享

在您使用 Cmd Markdown 记录，创作，整理，阅读文稿的同时，我们不仅希望它是一个有力的工具，更希望您的思想和知识通过这个平台，连同优质的阅读体验，将他们分享给有相同志趣的人，进而鼓励更多的人来到这里记录分享他们的思想和知识，尝试点击 <i class="icon-share"></i> (Ctrl+Alt+P) 发布这份文档给好友吧！

------

再一次感谢您花费时间阅读这份欢迎稿，点击 <i class="icon-file"></i> (Ctrl+Alt+N) 开始撰写新的文稿吧！祝您在这里记录、阅读、分享愉快！

作者 [@ghosert][3]     
2015 年 06月 15日    

[^LaTeX]: 支持 **LaTeX** 编辑显示支持，例如：$\sum_{i=1}^n a_i=0$， 访问 [MathJax][4] 参考更多使用方法。

[^code]: 代码高亮功能支持包括 Java, Python, JavaScript 在内的，**四十一**种主流编程语言。

[1]: https://www.zybuluo.com/mdeditor?url=https://www.zybuluo.com/static/editor/md-help.markdown
[2]: https://www.zybuluo.com/mdeditor?url=https://www.zybuluo.com/static/editor/md-help.markdown#cmd-markdown-高阶语法手册
[3]: http://weibo.com/ghosert
[4]: http://meta.math.stackexchange.com/questions/5020/mathjax-basic-tutorial-and-quick-reference



