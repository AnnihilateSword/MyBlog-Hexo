---
title: 【UE5 C++】材质自定义表达式中引用USF
tags: [UE5, C++]
cover: /res/img/post/【UE5 C++】材质自定义表达式中引用USF/cover.jpg
top_img: /res/img/post/【UE5 C++】材质自定义表达式中引用USF/cover.jpg
date: 2024-03-09 21:53:00
---
Generate Visual Studio project files

<img src="/res/img/post/【UE5 C++】材质自定义表达式中引用USF/1.png" width=800px>

# 添加（自定义模块的）文件映射路径

<img src="/res/img/post/【UE5 C++】材质自定义表达式中引用USF/2.png" width=800px>

<img src="/res/img/post/【UE5 C++】材质自定义表达式中引用USF/3.png" width=800px>

<img src="/res/img/post/【UE5 C++】材质自定义表达式中引用USF/4.png" width=800px>

# 重新生成项目

<font color=red>为了防止文件冲突，删除掉 Binaries 和 Intermediate 文件夹之后，再次点击 Uproject 右键 Generate Visual Studio project files

重新生成之后，进入解决方案 Build 成功之后，在材质表达式中引用 USF 文件了</font>

<img src="/res/img/post/【UE5 C++】材质自定义表达式中引用USF/5.png" width=800px>

The End.