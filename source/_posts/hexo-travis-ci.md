---
title: hexo基于travis自动构建发布到github和coding的实现方案
tags:
  - hexo
  - Travis-CI
categories:
  - 技术
---
现在的静态博客是越来越火了，hexo就是其中的一个，我们有很多人都在用，原因无非是全静态的我们可以部署在github上，不需要自己购买主机托管，而且还可以时不时提交。但是全静态的Blog有个问题，就是我们需要在自己的机器上面部署个构建环境，如果换了台没有环境的机器，我们是没办法写。所以，这里主要讲的是如何使用travis-ci自动构建并且提交到我们的github上。这里假设你已经有了一个hexo博客，并且对travis-ci有一定的了解。

### 安装hexo
安装好后，提交到github。这些都一一省略，最主要的是如何在travis里面实现构建完成后自动发布到github和coding

### 使用github的账号授权登陆CI
访问 [travis-ci](https://travis-ci.org)，使用github登陆到travis-ci，登录后你会发现你github里面的项目都被同步到travis里面来了，然后你可以指定你的某个项目需要开启CI

### 本地安装Travis的命令行工具
可以百度Travis CI Command Line Client的安装方法

### 本地命令行登录Travis
在我们本地的travis CI里登陆我们的travis账户，其实也就是你的github账户

```
xxxdeMacBook-Pro:~ Johnny$ travis login 

We need your GitHub login to identify you.
This information will not be sent to Travis CI, only to api.github.com.
The password will not be displayed.

Try running with --github-token or --auto if you don't want to enter your password anyway.

Username: xxx@gmail.com
Password for xxx@gmail.com: **********
Successfully logged in as xxx!
```

查看我们登陆的信息

```
xxxdeMacBook-Pro:~ Johnny$ travis whoami
Outdated CLI version, run `gem install travis`.
You are JohnnyWei188
```

### 加密私钥
使用Travis命令行工具加密我们的私钥

```
xxxdeMacBook-Pro:.travis Johnny$ travis encrypt-file id_rsa 
Outdated CLI version, run `gem install travis`.
Detected repository as xxx/xxx.github.io, is this correct? |yes| yes
encrypting id_rsa for xxx/xxx.github.io
storing result as id_rsa.enc
storing secure env variables for decryption

Please add the following to your build script (before_install stage in your .travis.yml, for instance):

    openssl aes-256-cbc -K $encrypted_ffdfc123d95c_key -iv $encrypted_ffdfc123d95c_iv -in id_rsa.enc -out id_rsa -d

Pro Tip: You can add it automatically by running with --add.

Make sure to add id_rsa.enc to the git repository.
Make sure not to add id_rsa to the git repository.
Commit all changes to your .travis.yml.

```
注意，我们加密好的文件应该放在你的hexo的根目录下的.travis目录里面，另外，刚刚我们加密了本机的私钥，所以，我们也需要将我们本地的公钥添加到github和coding里面

### 添加到.travis.yml配置文件 
我们加密了我们的私钥之后，命令行工具上会有一条openssl的解密命令，它生成了2个环境变量`$encrypted_ffdfc123d95c`和`encrypted_ffdfc123d95c_iv`，这2个环境变量在你登陆到travis-ci后，在你的项目设置里也可以看到，它们就是用来在你的CI环境里面解密你的私钥的。接着我们copy刚刚openssl这条信息到我们的.travis.yml文件中，放在before_script这个stage里面，如下

```
before_script:
  - openssl aes-256-cbc -K $encrypted_ffdfc123d95c_key -iv $encrypted_ffdfc123d95c_iv -in .travis/id_rsa.enc -out ~/.ssh/id_rsa -d
  - chmod 600 ~/.ssh/id_rsa
  - eval $(ssh-agent)
  - ssh-add ~/.ssh/id_rsa
  - cp .travis/ssh_config ~/.ssh/config
  - git config --global user.name 'xxx'
  - git config --global user.email xxx@gmail.com
```
### 修改我们的hexo的站点配置
在我们的站点配置文件_config.yml总，在我们的deploy这一项stage里，添加发布到githup和coding的git路径

```
deploy:
  type: git
  name: xxx
  email: xxx@gmail.com
  repo: 
    github: git@github.com:xxx/xxx.github.io.git,master
    coding: git@git.coding.net:xxx/xxx.git,master

```

### 结束
这时候在Travis-CI使用hexo d -g 的话就可以免密码的自动提交到github或者coding里面了

整个流程其实就是：

* 提交你的文件到github
* Travis-CI检测到你有提交，然后把你的github上的代码拉下来进行一些操作，比如构建，打包，执行一些命令
* 构建完成之后再扔给github或者coding

这篇文章主要讲的就是，在Travis-CI里面如何提交还给github，因为CI里它仅仅只是一个docker环境。总不能直接在_config.yml里写上你的github或者coding的账户密码吧。

### 写在最后
这应该算是真正意义上的第一篇博客，我大多数都是写云笔记里面自己看，对外可能看起来有点语无伦次，见谅。


参考资料：

https://blog.travis-ci.com/2013-01-14-new-client/
http://www.huangyijie.com/2016/09/20/blog-with-github-travis-ci-and-coding-net-1/
http://www.huangyijie.com/2016/10/05/blog-with-github-travis-ci-and-coding-net-2/