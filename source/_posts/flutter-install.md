---
title: flutter尝鲜：flutter的安装
date: 2020-12-15 14:22:58
tags: 
- flutter
- 移动端
---

## Mac端
1. 下载 Flutter SDK，在[flutter.cn](https://flutter.cn/docs/get-started/install)下载的速度会快很多

2. 下载好后，解压到文件夹内

<!-- more -->

3. 由于在国内访问flutter可能会受到限制，Flutter官方为中国开发者搭建了临时镜像，将如下环境变量加入到用户环境变量中即可：
```
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
```
同时，需要将flutter的命令也添加到命令行中
```
export PATH=${这里替换为Flutter的解压目录}/flutter/bin:$PATH
```

## IOS设置
### 配置模拟器
使用以下命令打开模拟器
```
open -a Simulator
```

### 安装到真机
1. 安装[homebrew](http://brew.sh/)
2. 使用homebrew安装以下工具
```
brew update
brew install --HEAD libimobiledevice
brew install ideviceinstaller ios-deploy cocoapods
pod setup
```

## 安卓配置
### 模拟器
Android Studio 安装好模拟器后，可以使用以下命令启动模拟器(`Nexus_5_API_22`为模拟器的名字)  
先设置zsh里面的emulator别名
```
alias emulator="/Users/wmy/Library/Android/sdk/emulator/emulator"
```
然后启动模拟器
```
emulator -avd Nexus_5_API_22
```