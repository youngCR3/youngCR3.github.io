---
title: Mac M1 VScode配置C++
date: 2022-04-09 22:04:29
tags: 软件安装
categories: Tools
---

Mac M1配置VSCode的C++环境

<!--more-->

# 注意点

- 直接搜索Mac的配置方法大多是Intel芯片版本的方法，因此搜索时应使用Mac M1作为关键词进行搜索

# 插件安装

- C/C++
- CodeLLDB（Mac M1必须）

# 配置launch.json

```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "clang++ - 生成和调试活动文件",
            "type": "lldb",
            "request": "launch",
            "program": "${fileDirname}/${fileBasenameNoExtension}",
            "args": [],
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": false,
            "MIMode": "lldb",
            "preLaunchTask": "C/C++: clang++ 生成活动文件"
        }
    ]
}
```



# 配置task.json

```json
{
    "tasks": [
        {
            "type": "cppbuild",
            "label": "C/C++: clang++ 生成活动文件",
            "command": "/usr/bin/clang++",
            "args": [
                "-g",
                "${file}",
                "-o",
                "${fileDirname}/${fileBasenameNoExtension}"
            ],
            "options": {
                "cwd": "${fileDirname}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "detail": "调试器生成的任务。"
        }
    ],
    "version": "2.0.0"
}
```

