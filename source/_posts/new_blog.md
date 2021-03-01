---
title: 基于hexo和github的搭建个人博客教程
date: 2021/3/1 13:26 
tags:
- 博客
- hexo
- github
categories: 教程
thumbnail: /img/02.png
---

## 配置环境

- Node的安装

  - [Node官方下载](https://nodejs.org/en/)

  - 进入node下载地址后，选择**LTS**版本进行下载

    ![node的下载](/img/node.png)

  - 下载安装，根据指示进行即可。安装完成之后，可使用命令：`node -v`来查看版本号，若出现对应版本号，则安装成功。

    ![版本号](/img/node_v.png)

- Git的安装

  - [Git官方下载](https://git-scm.com/downloads/)

  - 根据电脑系统选择相应的版本进行下载，然后一路next就行

  - 安装完成后，可通过命令：`git version`来查看是否安装成功

    ![git版本](/img/git.png)

- npm的安装

  - 由于npm在国内下载速度较慢，因选用淘宝镜像进行下载

    - 可通过命令：`npm install -g cnpm --registry=http://registry.npm.taobao.org`进行安装

      ![npm的下载](/img/npm.png)

    - 也可以打开Git Bash，输入`npm config set registry https://registry.npm.taobao.org`命令进行安装

      ![npm下载](/img/npm_gitbash.png)

  - 安装完成后，可通过命令`cnpm -v`查看安装是否完成

    ![npm版本](/img/npm_v.png)

- hexo的安装

  - 通过命令`cnpm install -g hexo-cil`进行安装，需要等待一段漫长的时间。

    ![hexo安装](/img/hexo.png)

  - 安装完成后，通过命令`hexo -v`查看是否安装成功

    ![hexo版本](/img/hexo_v.png)

------

## 搭建博客

- 初始化

  创建或者选择一个文件夹作为存放自己博客的位置，然后在文件夹中打开**Git Bash**，通过命令`hexo init`进行初始化。如果初始化失败，那么就清空文件夹，重新进行初始化即可

  ![hexo初始化](/img/hexo_init.png)

  成功后会生成如下一些文件

  ![hexo初始化成功](/img/hexo_init_finish.png)

- 启动本地博客

  上述过程完成后，我们就可以启动博客了，依旧在博客文件夹下打开**Git Bash**，然后输入命令`hexo s`，通过网址`http://localhost:4000`即可访问我们亲手搭建好的博客

  ![hexo s命令](/img/hexo_s.png)

  ![4000](/img/hexo_s_4000.png)

------

## 将博客部署到github

- Github创建仓库

  我们需要在GitHub上新建一个仓库来存放我们的博客。注意仓库的名称有特定的格式要求，必须是个人昵称+github.io，之后我们就可以用这个地址去访问我们的博客。

  - 首先需要拥有**github**账号，没有的号进入[github官网](https://github.com/)进行注册

  - 在个人主页右上角，点击New repository创建

    ![创建](/img/new_rep.png)

  - 然后在**Repository name**填上个人昵称+github.io，再点击下方**Create repository**按钮就能创建好仓库

    ![repository](/img/rep.png)

- 将Git和Github账号绑定

  - 打开**Git Bash**，设置user.name和user.email配置信息

    ```
    git config --global user.name "你的GitHub用户名"
    git config --global user.email "你的GitHub注册邮箱"
    ```

  - 使用以下命令生成ssh密钥文件，然后直接三个回车，即默认不需要设置密码

    ```
    ssh-keygen -t rsa -C "你的GitHub注册邮箱"
    ```

  - 生成时会说明.ssh文件夹的位置，打开.ssh文件夹中的**id_rsa.pub**文件(可用记事本打开)，然后复制所有内容

  - 进入**Github**中[添加密钥](https://github.com/settings/ssh/new)的网页，Title可随意填写，并把刚才**id_rsa.pub**文件的内容粘贴到Key里面，点击**Add SSH key**按钮即可

    ![key](/img/key.png)

  - 在**Git bash**中检测公钥是否设置成功，输入命令`ssh git@github.com`，若出现如下所示，则说明成功

    ![公钥设置成功](/img/ssh.png)

- 配置博客

  - 在博客所在文件夹安装一个Git支持插件，打开**Git Bash**用命令`npm install hexo-deployer-git --save`

  - 然后打开你的博客文件夹的**_config.yml**文件，修改文件最后的**deploy**属性，修改为：

    ```
    deploy:
      type: git
      repo: https://github.com/YourGithubName/YourGithubName.github.io.git
      branch: master
    ```

    其中**YourGithubName**是你**Github**的名字

  - 然后再打开**Git Bash**，输入下面命令即可

    ```
    hexo clean
    hexo hexo generate //可以换成hexo g
    hexo server //可以换成hexo s 
    hexo deploy //可以替换成 hexo d
    ```

    其中`hexo s`命令为可选，上面有说到。`hexo d`命令则是同步到**Github**上，同时你的博客就会更新，第一次输入完，会让你输入github账号和密码，输入完成即可。

------

## 博客主题的更换

进入**hexo**官网的[主题](https://hexo.io/themes/)中，挑选自己喜欢的主题，再按照文档说明进行设置即可

