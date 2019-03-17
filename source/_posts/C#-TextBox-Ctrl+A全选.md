---
title: 'C# TextBox Ctrl+A全选'
tags:
  - Winform
abbrlink: 15970
date: 2016-01-28 19:00:38
categories:
  - 学习笔记
---
Winform程序中光标在TextBox控件中时按下 Ctrl + A 快捷键，并不能选中全部文字，而是会发出警告音。本文给出实现方法。
<!-- more -->
在TextBox控件中使用快捷键，一般要求按下快捷键立刻产生效果，KeyUp事件显然不符合我们的要求，而KeyPress事件中不支持使用组合件，所以我们选用KeyDown事件，具体代码实现如下：
```csharp
private void tBBefore_KeyDown(object sender, KeyEventArgs e)
{
    if (e.Control && e.KeyCode == Keys.A)
    {
        ((TextBox)sender).SelectAll();
    }
}
```