---
title: 基于GitLab Pages搭建一个优雅的静态博客工作流
date: 2020-01-07 14:39:21
updated: 2020-01-07 15:26:20
categories:
- 其它

tags:
- 修复问题
---
# 前言
使用Hexo作为博客的静态页面生成器，总觉得使用GitHub Pages工作流不够优雅。不优雅的地方：每次都需要自己在本地手动`hexo generate`生成页面，并且相关资源必须host在公开仓库才可以对外访问。

最近，在搭建一个项目博客的时候，了解到GitLab Pages工作流可以完美避免上述问题，符合我对于"优雅"的定义。期间踩了几个坑，了解到这方面的内容较少，遂记录分享一下。

<!-- more -->
# 使用git submodules管理主题
根据自己的需求对选用的主题进行自定义后，如在其基础之上新增一些特性或自定义配置等。为了便于管理，我们需要维护自己的主题代码仓库。而将自定义的主题整合到我们自己的博客生成过程中，使用`git submodule`进行管理再好不过。

## 有用的代码片段
```shell
# 在博客项目文件夹下，初始化submodule配置
git submodule init

# 拉取主题
# git@gitlab.com:lezen1/blog-theme.git 是对应自定义主题的地址
# themes/hipaper 是对应主题的文件夹路径
git submodule add git@gitlab.com:lezen1/blog-theme.git themes/hipaper

# 更新主题
git submodule update --recursive --remote

# 删除主题
# themes/hipaper 为对应模块所在目录
git submodule deinit themes/hipaper
# 手动删除.gitmodules文件中的对应内容
vim .gitmodules 
git rm --cached themes/hipaper
# 手动删除.git/config文件中的对应内容
vim .git/config
rm -rf .git/modules/themes/hipaper
git commit -am "Removed submodule themes/hipaper"
rm -rf themes/hipaper
```

但是为了配合后续的`gitlab-ci`，我们需要在`.gitmodules`文件中，将其设置为相对引用。

```.gitmodules
# themes/hipaper 是对应主题的文件夹路径
# url是对应git仓库的相对引用路径
[submodule "themes/hipaper"]
	path = themes/hipaper
	url = ../blog-theme.git
```

## 对应gitlab-ci.yml配置
主要参考Hexo官方[GitLab Pages](https://hexo.io/docs/gitlab-pages)配置，增加了对于`git`命令的支持。

<details>
<summary>gitlab-ci.yml</summary>

```yaml
image: node:10-alpine
cache:
  paths:
    - node_modules/

variables:
  GIT_SUBMODULE_STRATEGY: recursive

before_script:
  - apk update 
  - apk add --no-cache git
  - git submodule sync --recursive
  - git submodule update --init --recursive
  - npm install hexo-cli -g
  - npm install

pages:
  script:
    - hexo generate
  artifacts:
    paths:
      - public
  only:
    - master
```
</details>

# 配合Cloudflare绑定域名
之前因为没搞懂GitLab Pages的工作流导致为了绑定域名踩了挺多坑。

如果你也使用Cloudflare作为DNS，可以参考这篇官方的教程：[Setting up GitLab Pages with CloudFlare Certificates](https://about.gitlab.com/blog/2017/02/07/setting-up-gitlab-pages-with-cloudflare-certificates/)进行配置。

# 参考资料
- [git中submodule子模块的添加、使用和删除](https://blog.csdn.net/guotianqing/article/details/82391665)
- Hexo官方[GitLab Pages](https://hexo.io/docs/gitlab-pages)配置
- [Setting up GitLab Pages with CloudFlare Certificates](https://about.gitlab.com/blog/2017/02/07/setting-up-gitlab-pages-with-cloudflare-certificates/)

# Changelog
- 2020/01/07：第一版内容
