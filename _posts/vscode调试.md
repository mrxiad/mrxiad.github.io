---
title: vscode配置
date: 2023-12-05 23:01:45
tags: 技术
categories: 
- 工程
- vscode
---



本文介绍vscode里的相关配置

# 常用快捷键



## 对于 **词** 的操作：

- 选中一个词：ctrl` + d`



## 搜索或者替换：

- ctrl` + f` ：搜索
- ctrl` + alt + f`： 替换
- ctrl` + shift + f`：在项目内搜索



## 其他

1. 通过**Ctrl + `** 可以打开或关闭终端
2. Ctrl+P 快速打开最近打开的文件
3. Ctrl+Shift+N 打开新的编辑器窗口
4. Ctrl+Shift+W 关闭编辑器
5. Home 光标跳转到行头
6. End 光标跳转到行尾
7. Ctrl + Home 跳转到页头
8. Ctrl + End 跳转到页尾
9. Ctrl + Shift + [ 折叠区域代码
10. Ctrl + Shift + ] 展开区域代码
11. Ctrl + / 添加关闭行注释
12. Shift + Alt +A 块区域注释
13. **Ctrl+Shift+P**		 **强大的**命令窗口







# Json讲解

**牢记**`{ }`中的内容表示一个对象，`[ ]`中的内容表示一个数组。



## task.json

tasks.json可以编辑多个任务，只需要在tasks后继续添加即可，如下：

```bash
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "HelloTask",
            "type": "shell",
            "command": "echo Hello"
        },
        {
            "label": "ByeTask",
            "type": "shell",
            "command": "echo bye"
        },
    ]
}
```

> 上面分别创建了HelloTask和ByeTask两个任务。







## launch.json

