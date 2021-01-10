---
title: Jenkins自动化部署前端项目实战
date: 2021-01-09 12:03:53
tags: 
  - 运维
  - 前端工程化
---

在早期，我部署项目的过程是，现在本地打好包，然后连同 dist 文件夹一起推到 gitlab 上面，然后再登录到服务器上拉取。简单的项目还好，但是若有好几个前端项目，然后项目又分开发、测试、正式环境，那前面的过程就要重复很多遍，效率很低过程也很繁琐。

因此，就尝试了使用 Jenkins 自动化部署前端项目。弄好了之后，发现真香！效率大大提高，解放双手！舒服！下面记录了我从0到1的过程，还没自动化部署的伙伴们，赶紧试一试！

<!-- more -->
## 自动化部署方案
我的自动化部署方案是
- 所有 dev 分支的更新都会自动触发 Jenkins 构建，自动部署到测试服务器
- master的更新，需要到 Jenkins 手动点击构建后，才会自动构建部署到正式服务器
- Jenkins 可以选择 tag 或者分支进行构建，以便出现问题回滚
- Jenkins 自动部署到服务器，会将 dist 保存备份，出现问题可以选择对应的备份包回滚
- Jenkins 打包的进行状态、打包时间、打包结果都通过钉钉群通知成员

## Jenkin的安装
这里我主要是基于 docker 安装 jenkins，其他的安装方式可以自行查看[官方文档](https://www.jenkins.io/zh/doc/book/installing/)。

环境：
- 系统：`centos 8.0`
- docker 版本：`20.10.2`
- docker 镜像：`jenkinsci/blueocean`

### 拉取镜像
搜索镜像
```
docker search jenkins
```
docker 官方的 jenkins 镜像太老了，很多plugins都无法使用，这里使用官网文档推荐的 `jenkinsci/blueocean`
```
docker pull jenkinsci/blueocean
```
### 创建容器
首先创建一个目录专门存放 jenkins 相关数据
```
mkdir -p /opt/jenkins
```
实例化容器
```
docker run -u root --restart=always -d -p 8081:8080 -p 8082:50000 -v /opt/jenkins:/var/jenkins_home -v /var/run/docker.sock:/var/run/docker.sock jenkinsci/blueocean
```

## Jenkins 初始化
Jenkins 容器创建完毕后，就可以登录8081端口登录了

登录的时候会叫你输入初始密码，密码的位置可以通过创建容器映射的目录获取
```
cat /opt/jenkins/secrets/initialAdminPassword
```
后续还会引导设置admin用户以及安装推荐的插件

## Jenkins 插件配置
本次前端自动化部署需要安装以下插件：
- `nodejs`：在 jenkins 里面安装 nodejs 环境，用于项目打包
- `Git Parameter Plugin-In`：用于让项目可以根据 tag 或者 branch 进行构建
- `GitLab`：可以设置Gitlab等配置，拉取gitlab的代码、通过gitlab的webhook触发 Jenkins 构建等
- `Publish Over SSH`：用于将 Jenkins 打包好的 dist 文件夹传送到服务器
- `DingTalk`：设置钉钉的机器人，在钉钉群通知 Jenkins 的打包状态

安装步骤：
1. 点击首页左侧菜单的 `Manage Jenkins`
2. 选择 `Manage Plugins`
3. 点击 Tab - `Available`
4. 在搜索框搜索对应的插件
5. 选中插件，然后点击`Intall without restart`，等待安装成功即可

### Nodejs 插件配置
点击首页的 `Manage Jenkins`

然后选择 `Global Tool Configuration`

滑动到下面 NodeJS 栏目，点击添加 NodeJS

最好添加一个跟服务器版本一致的 NodeJS 版本，避免因为版本差异出现的问题，这里选择的是 `10.15.0`

![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20210110143724.png)

### Gitlab 配置
点击首页的 `Manage Jenkins`，点击 `Configure System`，查找到 gitlab 配置项

设置：
- Connection Name：Gitlab 连接的名字，随便填
- Gitlab host URL：Gitlab 的URL，（若 Jenkins 和 Gitlab 在同一个内网内，可以填写内网地址，这样不走外网，拉取速度会快很多！）
- Credentials：安全验证，选择可以连接到gitlab的验证方式，一开始没有的话可以点击旁边的添加，添加一个安全验证

连接到Gitlab的安全验证的添加：
- 这里类型选择了 `Gitlab API Token`（自己也可以选择密钥等其他类型）
- 登录到 gitlab，进入到用户设置，选择 `Access Setting`，设置名字、过期时间、权限，点击创建即可获得权限token
![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20210110161701.png)
- 在 Jenkins 的`Add Credentials`页面中的`API token`填写刚刚 Gitlab 生成的 token
![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20210110162206.png)

添加完信息后，即可点击下面 `Test Connection`，若出现 Success 则说明配置成功
![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20210110162623.png)

### Publish Over SSH 插件配置
设置步骤：
- 点击首页主菜单的 `Manage Jenkins`
- 点击 `Configure System`
- 设置 `Publish over SSH` 下的栏目

在下面添加自己想要部署到的 Server，配置项有
- `Name`：Server的名字，用于标识服务器的，可以自己随便起个好听的名字
- `HostName`：Server的ip地址或者域名
- `Username`：ssh 登录的用户名
- `Remote Directory`：服务器的目录，只能部署到这里目录下面（目录需要提前创建）
- `Passphrase / Password`：ssh 登录的密码，这里选择了密码登录，若使用密钥登录，可以设置 `Path to key` 和 `Key`

> 若有不懂的地方，可以点击配置右边的问号图标，Jenkins 就会出现对应的提示，十分的友好，点赞！ o(￣▽￣)ｄ

配置完后，可以点击下面的 `Test Configuration`，若提示 `Success`，则说明配置成功，若失败了，可以看一下自己是否参数配置错误了

![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20210110145042.png)

### DingTalk 插件配置
首先在钉钉创建好群，给群添加一个通知机器人
> 小技巧：可以使用【面对面建群】创建只有自己一个人的群，然后就可以添加私人机器人了

机器人添加步骤：
- 点击钉钉PC端头像，选择机器人管理
- 选择添加 自定义的机器人
- 设置头像、名字、通知的群，加密我这里选择了签名 Singnure（**需要提前记下来签名后面配置插件需要用到**）
![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20210110154401.png)
- 点击完成后，就会提供对应的 webhook，这个也会用到
![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20210110152059.png)

准备好后，配置 Jenkins 的 DingTalk 插件：
- 点击首页菜单的 `Manage Jenkins`
- 选择 `Configure System`
- 配置钉钉下面的配置项，可以配置通知时机
- 点击添加机器人

钉钉插件的机器人配置项如下：
- id：机器人id，我这里随便填了
- 名称：机器人名字，我这里随便填了
- webhook：填写上面创建机器人的 webhook
- 关键字：加密方式为关键字的，可以将关键字写在这，因为加密方式我选择了签名，所以我这里空着
- 加密：添加创建机器人时候的 Signature

设置完后，可以点击下面的测试，若钉钉群接受到机器人的通知，则说明配置成功

![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20210110152859.png)

## Jenkin自动化部署前端项目
### 添加项目
点击首页主菜单的 `New Item`，填写项目名字，选择 `Freestyle Project`，点击 OK

![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20210110163103.png)

### 总体配置（General)
#### 设置钉钉通知
在钉钉机器人栏目下面，选择刚刚添加好的机器人`Jenkins通知`即可，高级配置里面可以选择 @所有人，配置描述等

#### Gitlab 项目配置
勾选 Github Project，Project Url 填写，要部署项目的Gitlab地址（内网地址可以使用的填写内网指定对应的url,这样拉取回快很多）

GitLab Connection 填写之前配置好的 Gitlab 连接

#### 配置项目的构建参数
勾选 `This Project is parameterized` - `Git Parameter`
![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20210110165444.png)

Git Parameters 配置如下：
- Name：参数的名称
- Description：变量的描述
- Parameter Type：变量的类型，因为根据分支以及Tag构建，所以这里选择 `Brand or Tag`
- Default Value：变量默认值，这里填写默认进行构建的分支，即 dev

![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20210110172541.png)

> 注意：还需要后面，将构建的分支设置为该参数变量才生效
### 源代码管理配置（Source Code Management)
`Source Code Management`，选择 `Git`

配置参数：
- Repository URL：填写项目的 Url 地址，可选 ssh 地址或者 http 地址（有内网地址最好替换为内网对应的地址）
- Credentials：git 的认证方式，若 url 写的是 ssh 地址，则认证方式为密钥方式，若 url 填写 http 地址，则认证方式需要是 gitlab 账户和密码的方式
- Branches to build：构建的分支，这里填写上面的 `BrandAndTag` 参数变量

![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20210110172826.png)

### 构建触发器配置（Build Triggers）
正式服的构建任务，因为是需要在 Jenkins 手动点击构建才自动构建并部署到正式服务器（这样可避免失误合并到master分支就自动部署到正式环境了），所以可以跳过此步骤

而测试服的构建任务是需要一旦检测到 dev 分支有更新，就会自动构建部署到测试服，所以这里需要配置 Gitlab 的 webhook 触发 Jenkins 构建

勾选 `Build when a change is pushed to GitLab`。
其后面有一个有用的参数，即 GitLab webhook URL: **`http://**.**.**.**:8081/project/VueProject`**

可以根据自己的需求进行配置触发的时机

点击下面的 `Advance`，里面有一项 `Secret token`，点击生成，可得到 **token**
![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20210110175221.png)

#### 配置 gitlab 的 webhook
进入到 gitlab 对应的项目目录，点击 `Setting` - `Webhooks`

配置：
- URL：刚刚 Jenkins 给的 webhook url 参数
- Secret Token：刚刚 Jenkins 生成的 `Secret Token`
- Trigger：配置触发的时机

![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20210110174439.png)

配置完后，点击`Add webhook`即可，后续可点击 Test 测试是否成功

### 构建环境配置（Build Environment）
构建环境使用Node，所以勾选 `Provide Node & npm bin/ folder to PATH`

选择好之前配置好的 Node 环境即可

![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20210110175555.png)

### 构建配置（Build)
点击 `Add build step` - 选择 `Execute shell`
<!-- ![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20210110175819.png) -->

在 Command 项里面填写打包的命令即可，我的打包命令为
```sh
#!/bin/bash
# 设置淘宝镜像源
npm i --registry=https://registry.npm.taobao.org

# 打包
npm run build

# 压缩好dist文件夹的内容，准备推送到服务端
cd ./dist
tar -zcf dist.tar.gz *
cd ..
```
![](https://blog-1257268092.cos.ap-guangzhou.myqcloud.com/notes/20210110180522.png)

### 部署到服务端配置（Post-build Actions)
点击 `Add post-build action` - 选择 `Send build artifacts over SSH`

选择想要部署的服务器（可选项就是之前 Publish Over SSH 插件配置好的项目），可以自行添加多台部署服务器

这里只部署一台，也就是之前配置好的 OtherServer

配置项有：
- `Source files`：发送到服务端的软件，这里的路径是相对项目中的根目录
- `Remove prefix`：移除的路径，这里移除 dist 路径
- `Remote directory`：发送到的服务端路径（必须要在该服务器配置的Remote Directory下面）
- `Exec command`：发送成功后执行的命令

我这里的执行命令为：
```sh
# 注意，这里当前路径为ssh登录后的路径（默认为用户目录，即 ~/），而不是项目的目录
# 打开项目目录
cd /mount/release/qw_cms

# 删除dist文件夹
rm -rf ./dist

# 创建dist文件夹
mkdir dist

# 解压到dist文件夹
tar -zxf dist.tar.gz -C dist

# 重命名 dist.tar.gz保存记录，以防有错误下解压来实现回滚
mv dist.tar.gz `date +%Y%m%d%H%M%S`${TAG_NAME}.tar.gz

# Todo：清除最近10次以外的备份包
```