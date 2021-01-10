---
title: docker搭建gitlab记录
date: 2021-01-09 22:38:22
tags:
- 运维
- docker
---

gitlab 是大多数公司代码管理的工具。之前也很好奇gitlab是怎么搭建的，这次尝试了使用docker搭建gitlab,其中也踩了不少的坑。大家照着我这个过程搭建应该不是什么问题。

<!-- more -->

首选需要搭建docker的环境，这里就不介绍了。大家自行去查找。本篇主要介绍怎么使用docker搭建 gitlab。

## 安装
### 个人主机配置
个人配置环境如下：
- 操作系统：CentOS 8.0 64位
- CPU：1核
- 内存：4GB

gitlab官方推荐至少配置 4gb的内存，若内存不足，则有可能会提示502等错误提示，若小内存主机搭建gitlab，可以设置虚拟内存，参考 https://blog.csdn.net/rex1129/article/details/110119830

### 安装镜像
由于官方的镜像站比较慢，可以先设置加速镜像库，参考：
- 腾讯云：https://cloud.tencent.com/document/product/1207/45596
- 阿里云镜像加速：https://help.aliyun.com/document_detail/60750.html

然后拉取镜像
```
docker pull gitlab/gitlab-ce
```

然后准备创建好对应的文件夹
- /opt/gitlab/data
- /opt/gitlab/config
- /opt/gitlab/logs

```
docker run \
    --detach \
    --publish 36443:443 \
    --publish 3680:80 \
    --publish 3622:22 \
    --name gitlab-ce \
    --restart unless-stopped \
    -v /opt/gitlab-ce/config:/etc/gitlab \
    -v /opt/gitlab-ce/logs:/var/log/gitlab \
    -v /opt/gitlab-ce/data:/var/opt/gitlab \
    gitlab/gitlab-ce
```

## 配置 gitlab
进入容器
```
docker exec -it gitlab-ce /bin/bash
```
修改文件
```
vim /etc/gitlab/gitlab.rb
```
修改内容
```rb
# 修改外部路径，回影响创建项目的路径
external_url 'http://42.193.174.122:3680'

# 设置 ssh 主机名，影响创建项目的 ssh 地址
gitlab_rails['gitlab_ssh_host'] = '42.193.174.122'

# 设置 ssh 的端口
gitlab_rails['gitlab_shell_ssh_port'] = 3622


# 设置邮箱
 gitlab_rails['smtp_enable'] = true
 gitlab_rails['smtp_address'] = "smtp.qq.com"
 gitlab_rails['smtp_port'] = 465
 gitlab_rails['smtp_user_name'] = "47*****56@qq.com"
 # 腾讯邮箱的是授权码，具体可以看邮箱的文档
 gitlab_rails['smtp_password'] = "ehq*********cbaf"
 gitlab_rails['smtp_domain'] = "smtp.qq.com"
 gitlab_rails['smtp_authentication'] = "login"
 gitlab_rails['smtp_enable_starttls_auto'] = true
 gitlab_rails['smtp_tls'] = true
 gitlab_rails['smtp_openssl_verify_mode'] = 'none'

 # 设置发送的邮箱
gitlab_rails['gitlab_email_from'] = '47*****56@qq.com'
```

修改完后，需要重新配置 gitlab
```
gitlab-ctl reconfigure
```

邮箱测试，需要进入到 gitlab-rails 的终端
```
gitlab-rails console
```
发送邮箱测试
```
Notify.test_email("myweico@qq.com","title","gitlab").deliver_now
```