---
title: 调试神器-Charles抓包工具的使用
date: 2020-11-14 17:22:26
tags:
  - 测试
  - 运维
---

## Charles

Charles 是一个抓包软件，能够抓取电脑以及手机上的 http/https 的网络包，对于我们调试接口来说，十分方便。

Charles 提供的功能：

- 抓包
- 修改请求，再次发送
- 并发、重复发起请求，压力测试
- 限流，调试弱网情况
- 修改请求、以及响应

<!-- more -->

charles 的原理是，启动一个代理服务器，让请求包都经过 charles 启动的代理服务器，然后再请求对应的 url。

## 抓包

### 启动 charles 的 http 代理服务

1. charles 菜单栏 `proxy` - `proxy-setting`
   ![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20201114184335.png)

2. 启动 charles 的代理服务器，并设置端口（此处设置为 9999）
   ![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20201114184517.png)

### Mac 上的代理

1. 菜单栏上 `proxy` - `Mac Proxy` ，启动 Mac 上的代理

这时候 Mac 上的 http 请求已经可以正常抓取到了，但是 https 还是一堆乱码甚至无法访问，因为没有还没有安装 charles 的安全证书

2. 安装 charles 的安装证书，点击 charles 菜单栏 - `help` - `SSL Proxying` - `Install Charles Root Certificate`
   ![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20201114185032.png)

安装证书
![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20201114185250.png)

在钥匙串里面搜索到 charles 的证书 - 双击 - 在 trust 下面 设置为 `always trust`
![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20201114185718.png)

3. 这时候 Mac 上的 http 以及 https 的网络包都可以正常抓取了

### ios 上的代理（安卓同理）

1. 手机连上 Mac 同局域网下的 wifi，将 wifi 的代理设置为 charles 的代理服务即可

`wifi` - 点击与 Mac 同样的 wifi - 滚动到最后，`配置代理` - 设置为手动 - 代理服务器写 Mac 的 ip 地址（charles 菜单栏 - help - Local IP Address 可以获取） - 端口写 charles 的代理端口（这里是 9999）
<img src="https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20201114190842.png" style="height:600px; margin: 0 auto;" />

2. 手机上安装证书，charles 菜单栏 `help` - `SSL Proxying` - 选择在手机上安装证书`
   <img src="https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20201114190934.png" style="margin: 0 auto;" />

3. 手机浏览器访问对话框指定的 url，下载证书（这里是`chls.pro/ssl`)
   <img src="https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20201114191023.png" style="margin: 0 auto;" />

4. 下载完成后，打开手机设置 - `通用` - 滑到最后，选择`描述文件` - 安装刚下载的证书

5. 然后回到 `通用` - 滑到最前，选择`关于本机` - 然后滑到最后，选择`证书信任设置` - 选择信任刚刚安装好的证书

6. 这时候手机上 http 以及 https 的网络包已经可以正常抓取了（调试完毕后记得关掉代理，否则会无法上网，为了安全，可将证书暂时设置为不信任，调试的时候再信任即可）

## 筛选
网络请求过多，不易找到我们想要监听的网络包，可以使用一下方法：
- 搜索：`edit` - `find` 或者 `Command + F`快捷键筛选
- 设置 filter
- 将要监听的 url 设置为 focus
- 设置监听选项：`Proxy` - `Recording Settings`里面配置 include 或者 exclude

## 限流
在 `Proxy` -  Throttle` 可以限流，模拟各种网络请求速度，这对于我们模拟弱网条件十分有用

## 编辑请求，重新发送请求
- compose：编辑请求的内容，再次发送请求，这在测试接口的时候就非常有用
- repeat：再次发送请求
- repeat advance: 可以设置请求次数，并发数，可用于压测

## 修改响应
- Map Remote: 将指定url映射到指定url，在调试微信支付域名校验的时候十分有用，可以将正式服的域名映射到测试服中，调试测试服
- Map Local: 将响应映射到本地文件
- Map Rewrite: 将匹配到的内容重写为指定内容，（可正则匹配）
- BreakPoint: 设置请求的断点，可单个调试请求，修改请求和响应