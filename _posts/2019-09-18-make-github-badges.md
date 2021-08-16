---
priority: 0.6
title: Github Badges
excerpt: 制作 Github 徽记
categories: [Summarise]
background-image: climb.jpeg
tags:
  - Travis
  - CI
  - Github
---

现在互联网从业人员应该没有没用过Github的人, 不知道在访问一些优秀的开源项目的时候,有没有注意README开头那一系列的徽记.如图

![tags.png](/assets/img/topic/tags.png)

对travis有了解的人都知道build_pass徽记的来历,那其他的呢? 怎么给自己的项目加上这些加分的徽记?

虽然我很想借此水一水, 但是在答案太过简单——https://shields.io/

不过不少人刚访问这个网站可能是懵的,真是将简洁方便贯彻到底.

这个网站制作徽记实际上注册登录都不需要,直接选取你想制作的徽记类型, 根据你项目所在选择相应的URL并填上所需信息即可,比如github的licence徽记/github/license/:user/:repo, 其中:user和:repo毋庸置疑, 就是你的github账号和仓库名,只需要填上即可,如图

![licence_example](/assets/img/topic/licence_example.png)]

有些同学可能得到的是notfound, 这是因为这徽记是根据项目中的LICENSE.md文件生成的,同理其他很多徽记,如cocoapods相关的一系列徽记都是基于podspec文件生成, 都需要制定的工程目录下确实存在所需的文件.

还有些同学找了一圈没看到自己需要的,不怕, 创建个自定义的徽记也是手到擒来

通过指定label message 和color可以轻松创建新的静态徽记.

如果想通过参数创建动态的徽记也可以,只是参数稍复杂.

感谢https://shields.io/的创建者~!
