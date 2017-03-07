---
title: androidstudio快捷键整理
date: 2017-02-27 14:05:54
tags: androidstudio
---
因为工作需要,在Windows,Mac,Linux三个不同的环境下使用Studio.最蛋疼的就是环境变量和配置和快捷键的配置不同.
网上很多关于Studio快捷键的文章,通常都是使用默认的快捷键.不同平台的默认键是不同的.如果像我这样蛋疼的有不同的平台环境,还是得统一快键配置.当然有钱人最好是使用MAC环境.
通常的辅助键位是CTRL,SHIFT,ALT.MAC下ALT对应的是option
在MAC下多了个核心键CMD,所以MAC平台下有四个辅助键位.因为我经常外接机械键盘,所以在键位的配置上使用ALT键当CMD,win键当option.
在windows和Linux下,win键通常都是定义了系统级操作,所以建议不要在win键上分配快键.
<!-- more -->
这里把快键按功能分一下类
## 编辑
### 光标移动
* 移动光标到单词左右 move caret to word start/end
* 移动光标到行首尾  move caret to line start/end  因为MAC没有HOME?END键,建议统一  
* 移动光标到代码块首尾 move caret to codeblock start/end

### 选择
* 选择光标所在行
* 选择至光标所在行首尾 move caret to line start/end with selection 不同平台不一样.建议统一
* 选择光标所在单词,重复时扩大/缩小选择 Extend/Shrink selection  不同平台不一样,建议统一
* 向左/右按单词选选择

### 删除
* 删除当前行 Delete line at caret  不同平台不一样,建议统一.
* 删除至行首尾 Delete to line start/end at caret 不同平台不一样,建议统一

### 自动完成
* 基本自动完成  Basic Code Completion  默认Ctrl+空格.
* 智能自动完成  Smart Code Completion  默认Ctrl+Sthif+空格.

## 重构
### 提取
* 提取成员变量 Extract Field    默认CTRL+ALT/CMD+option +F
* 提取临时变量 Extract Variable 默认CTRL+ALT/CMD+option +V
* 生成静态变量 Extract Field    默认CTRL+ALT/CMD+option +C
* 生成方法参数 Extract Field    默认CTRL+ALT/CMD+option +P
* 生成方法 Extract Field        默认CTRL+ALT/CMD+option +M

### 优化
* 代码格式化 Reformat code 默认CTRL+ALT+L
* 优化导入 Optimize improts 默认CTRL+ALT+O

## 查看
### 搜索
* 全局搜索 Search everywhere 双击SHIFT
* 指令搜索 Search Ation 默认CTRL/CMD+SHIFT+A
* 当前文件内搜索 Find 默认CTRL+F
* 全局搜索 Find in Path 默认CTRL+SHIFT+F
* 查看前/后一个搜索结果 Find next/previous 默认F3/SHIFT+F3
* 当前文件内替换 Replace 默认 CTRL+R
* 全局替换 Replace in Path 默认CTRL+SHIFT+R
* 文件内搜索方法调用 Find usages 默认 ALT/option+F7
* 全局搜索方法的调用 Find usages in file 默认 CTRL/CMD+F7
* 搜索类 Class navigatie 类搜索 默认 CTRL/CMD+N
* 文件搜索 File navigatie 文件搜索  默认 CTRL/CMD+SHIFT+N
* 查看大纲 File structure 默认CTRL/CMD+F12

### 查看
* 查看最近文件 Recent file 默认CTRL/CMD+E
* 看看最近修改的文件 Recent change files 默认CTRL/CMD+Sthif+E
* 查看文件修改记录 Recent changes
* 查看选择的方法结构 Method hierarchy 默认CTRL/CMD+SHIFT+H
* 查看选择的类的结构 Type hierarchy 默认CTRL/CMD+H
* 查看选择的方法调用栈 Call hierarchy 默认CTRL/CMD+ALT/option+H
* 查看方法参数定义 Parameter info 默认CTRL/CMD+P
* 查看方法内容 Declaration 默认CTRL/CMD+B或点击
* 查看文档 Documentation 不同平台不一样,建议统一

## 导航

## 运行


## GIT
