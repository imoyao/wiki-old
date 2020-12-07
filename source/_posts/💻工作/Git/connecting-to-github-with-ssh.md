---
title: 使用 SSH 连接到 GitHub
date: 2020-12-06 10:16:16
tags:
- GitHub
- Git
categories:
  - "\U0001F4BB 工作"
  - Git
---
## 前言
终于厌烦了每一次往远程仓库推送代码时都要手动输入用户名和密码验证个人信息，所以配置了一下 SSH 认证。

## [关于 SSH](https://docs.github.com/cn/free-pro-team@latest/github/authenticating-to-github/about-ssh)

{% note info %}

SSH 为 Secure Shell 的缩写，由 IETF 的网络小组（Network Working Group）所制定；SSH 为建立在应用层基础上的安全协议。

{% endnote %}

使用 SSH 协议可以连接远程服务器和服务并向它们验证。 利用 SSH 密钥可以连接 GitHub，而无需在每次访问时都提供用户名和个人访问令牌。


## [检查现有 SSH 密钥→](https://docs.github.com/cn/free-pro-team@latest/github/authenticating-to-github/checking-for-existing-ssh-keys)

在生成 SSH 密钥之前，您可以检查是否有任何现有的 SSH 密钥。

## [生成新 SSH 密钥并添加到 ssh-agent→](https://docs.github.com/cn/free-pro-team@latest/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent)

检查现有 SSH 密钥后，您可以生成新 SSH 密钥以用于身份验证，然后将其添加到 ssh-agent。

## [新增 SSH 密钥到 GitHub 帐户→](https://docs.github.com/cn/free-pro-team@latest/github/authenticating-to-github/adding-a-new-ssh-key-to-your-github-account)

要配置 GitHub 帐户使用新的（或现有）SSH 密钥，您还需要将其添加到 GitHub 帐户。

## [测试 SSH 连接→](https://docs.github.com/cn/free-pro-team@latest/github/authenticating-to-github/testing-your-ssh-connection)

设置 SSH 密钥并将其添加到您的 GitHub 帐户后，您可以测试连接。

## [使用 SSH 密钥密码→](https://docs.github.com/cn/free-pro-team@latest/github/authenticating-to-github/working-with-ssh-key-passphrases)

您可以保护 SSH 密钥并配置身份验证代理，这样您就不必在每次使用 SSH 密钥时重新输入密码。

## 其他

### 之前使用https克隆

对于已经克隆到本地的 https 类型仓库，我们可以通过修改 git 配置的方式实现 SSH 连接到远程仓库。
```plain
vim .git/config
```
像下面改 url
```plain
[remote "origin"]
url = git@github.com:hpcpp/hello-world.git
```

### Permission denied
```bash
ssh -T git@github.com
git@github.com: Permission denied (publickey).
```
直接添加刚才新增的key
```bash
ssh-add ~/.ssh/id_imoyao
```

## 参考链接
[使用 SSH 连接到 GitHub - GitHub Docs](https://docs.github.com/cn/free-pro-team@latest/github/authenticating-to-github/connecting-to-github-with-ssh)
[GitHub 教程 SSH keys 配置_LolitaSian-CSDN 博客](https://blog.csdn.net/qq_36667170/article/details/79094257)