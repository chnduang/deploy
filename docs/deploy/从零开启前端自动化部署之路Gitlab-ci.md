# 从零开启前端自动化部署之路 Gitlab-ci

> [https://mp.weixin.qq.com/s/U55cn7TFPbwdt9GP3HBPWw](https://mp.weixin.qq.com/s/U55cn7TFPbwdt9GP3HBPWw)

[面试官也在看的前端面试资料](<https://mp.weixin.qq.com/s?__biz=Mzg2NjUxOTM2Mg==&mid=2247488647&idx=2&sn=fb94c66432f08dc2d953e67d33bf474b&chksm=ce48c4aaf93f4dbc7da533737d2d877d4f15031b521e111883ec7ecbd73db545a6e9bd61ff72&scene=27&key=8adf9203f7a8000dcf40fab786ee7c8c3161f76276045499935eefce84f892349d739c1ccc6bc45cd086f11c14604be562c8008e2b15c80ab2d15e63a8d64f09bce4352aba50430c15ce61a1c36f6bfdb057ccf316e7c57dcd160759cb98d1d73c224595a5351fe5184871d151db85d06adaa1692809fa83a48a26258c3ae76b&ascene=1&uin=MTE5NzkwMDQyOA%3D%3D&devicetype=iMac+MacBookPro16%2C1+OSX+OSX+10.16+build(20G95)&version=13010510&nettype=WIFI&lang=zh_CN&fontScale=100&exportkey=AbChgqFz9pE%2FrGfqqri6dt4%3D&pass_ticket=fLP9V92MomGzP9pH3KLEfsm3bnRI8zfMRwFVqIdz48ri7Mz%2F8fv7z1lJc2RQBBLc&wx_header=0&fontgear=3.000000>)

## 目录

```
一.概念介绍
  1.1 gitlab-ci && 自动化部署工具的运行机制
  1.2 自动化部署给我们带来的好处

二.知识预备
  2.1 gitlab-ci涉及的抽象概念(Runner/PipeLine/Executor/Job )
  2.2 YML文件的基本语法规则
  2.3 .gitlab-ci.yml配置的特定关键字

三.CI实战
  3.1 编写一个gitlab-ci的“hello world”

四.坑点总结

五.gitlab-ci进阶
  5.1 YML的片段复用和模块化
  5.2 gitlab-ci提供的其他配置关键字
```

## 一.概念介绍

## 1.1 gitlab-ci && 自动化部署工具的运行机制

以 gitlab-ci 为例：

(1) 通过在项目根目录下配置**.gitlab-ci.yml**文件，可以控制 ci 流程的不同阶段，例如 install/检查/编译/部署服务器。gitlab 平台会扫描.gitlab-ci.yml 文件，并据此处理 ci 流程

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/bwG40XYiaOKnhD9PdakPWKG25O8MDWhx8rwf9bWjlcFvh0iamLSvkmbFC9fvyxiaWtBKpS41cnbmz0I0zGR2eUicmg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

(2) ci 流程在每次团队成员**push/merge**后之后触发。每当你 push/merge 一次，gitlab-ci 都会检查项目下有没有.gitlab-ci.yml 文件，如果有，它会执行你在里面编写的脚本，并完整地走一遍从**intall =>** **eslint 检查=>编译 =>部署服务器**的流程

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/bwG40XYiaOKnhD9PdakPWKG25O8MDWhx8S9VZw3JYO23zZgwIZFAEHaQ2dIohmlPj24GNxbibggp0JXkjp6ZXzWQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

(3)gitlab-ci 提供了指定 ci 运行平台的机制，它提供了一个叫**gitlab-runner**的软件，只要在对应的平台(机器或 docker)上下载并运行这个命令行软件，并输入从 gitlab 交互界面获取的 token,就可以把当前机器和对应的 gitlab-ci 流程绑定，也即：每次跑 ci 都在这个平台上进行。

(4).gitlab-ci 的所有流程都是可视化的，每个流程节点的状态可以在 gitlab 的交互界面上看到，包括执行成功或失败。如下图所示,因为它的执行看上去就和多节管道一样，所以我们通常用“pipeLine”来称呼它

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/bwG40XYiaOKnhD9PdakPWKG25O8MDWhx84VYguuiau1beZXNqFSemrDK2GYVLC3GKmiaxgs53o7ybxyE2TebdXUHw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

(5).不同 push/merge 所触发的 CI 流程不会互相影响，也就是说，你的一次 push 引发的 CI 流程并不会因为接下来另一位同事的 push 而阻断，它们是互不影响的。这一个特点方便让测试同学根据不同版本进行测试。

(6)pipeline 不仅能被动触发，也是可以手动触发的。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/bwG40XYiaOKnhD9PdakPWKG25O8MDWhx8p2sVVXfJk3nR10m2xmOBASDvlAlWibkA4CHIl9FrUkstLQiauESwn1rg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 1.2 自动化部署給我们带来的好处

自动化部署的好处体现在几个方面

**1.提高前端的开发效率和开发测试之间的协调效率**

**Before**

如果按照传统的流程，在项目上线前的测试阶段，前端同学修复 bug 之后，要手动把代码部署之后。才能通知测试同学在测试环境进行测试。

这会造成几个问题：本身手动部署服务的工作是比较繁琐的，占用了开发时间。同时开发-测试之间的环节的耦合问题，则会增加团队沟通成本。

**After**

通过 gitlab-ci，前端开发在提交代码之后就不用管了，ci 流程会自动部署到测试或集成环境的服务器。很大程度上节约了开发的时间。

同时，因为开发和测试人员可以共用 gitlab 里的 pipeline 界面, 测试同学能够随时把握代码部署的情况，同时还可以通过交互界面手动启动 pipeline，自己去部署测试，从而节约和开发之间的沟通时间。

**2.从更细的粒度把握代码质量**

我们可以把 eslint 或其他的代码检查加到 pipeline 流程中，每当团队成员提交和合并一次，pipeline 都会触发一次并对代码做一次全面检测，这样就从一个更细的粒度上控制代码质量了。

## 二.知识预备

介绍完 gitlab-ci 的基本概念，接下来我将会介绍编写一个 gitlab-ci 用例所需要的知识。这是在实战之前的一点准备工作，主要包括三部分

- gitlab-ci 涉及的抽象概念
- YML 文件的基本语法规则
- .gitlab-ci.yml 配置的特定关键字

## 2.1 gitlab-ci 涉及的抽象概念

首先要了解的是 gitlab-ci 中涉及的一些基本概念

**1.Pipeline & Job**

Pipeline 是 Gitlab 根据项目的.gitlab-ci.yml 文件执行的流程，它由许多个任务节点组成, 而这些 Pipeline 上的每一个任务节点，都是一个独立的 Job

Job 在 YML 中的配置我们将会在下面介绍，现在需要知道的是：**每个 Job 都会配置一个 stage 属性，来表示这个 Job 所处的阶段。**

**一个 Pipleline 有若干个 stage,每个 stage 上有至少一个 Job**，如下图所示：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/bwG40XYiaOKnhD9PdakPWKG25O8MDWhx8ChfsGlhN5C4Ot1VTIqgUlOLHSDZpLG4DianjuUwAaIh8XahH0szCYrA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**2.Runner**

Runner 可以理解为：**在特定机器上**根据项目的**.gitlab-ci.yml**文件，对项目执行 pipeline 的**程序**。Runner 可以分为两种：**Specific Runner** 和 **Shared Runner**

- **Shared Runner**是 Gitlab 平台提供的免费使用的 runner 程序，它由 Google 云平台提供支持，每个开发团队有十几个。对于公共开源项目是免费使用的，如果是私人项目则有每月 2000 分钟的 CI 时间上限。
- **Specific Runner**是我们自定义的，在自己选择的机器上运行的 runner 程序，gitlab 给我们提供了一个叫 gitlab-runner 的命令行软件，只要在对应机器上下载安装这个软件，并且运行 gitlab-runner register 命令，然后输入从 gitlab-ci 交互界面获取的 token 进行注册, 就可以在自己的机器上远程运行 pipeline 程序了。

> Gitlab-runner 下载链接：https://docs.gitlab.com/runner/install/

**Shared Runner 和 Specific Runner 的区别**

1. Shared Runner 是所有项目都可以使用的，而 Specific Runner 只能针对特定项目运行
2. Shared Runner 默认基于 docker 运行，没有提前装配的执行 pipeline 的环境，例如 node 等。而 Specific Runner 你可以自由选择平台，可以是各种类型的机器，如 Linux/Windows 等，并在上面装配必需的运行环境，当然也可以选择 Docker/K8s 等
3. 私人项目使用 Shared Runner 受运行时间的限制，而 Specific Runner 的使用则是完全自由的。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/bwG40XYiaOKnhD9PdakPWKG25O8MDWhx83QZRqCsgWDN46MbZdIRDxwAxRNsdm9yia3fqoibHDJPO24HqibeoEQ8mw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**3.Executor**

什么是 Executor？我们上面说过 Specific Runner 是在我们自己选择的平台上执行的，这个平台就是我们现在说到的“Executor”，我们在特定机器上通过 gitlab-runner 这个命令行软件注册 runner 的时候，命令行就会提示我们输入相应的平台类型。可供选择的平台一共有如下几种，下面是一张它们各方面特点的比较表格

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/bwG40XYiaOKnhD9PdakPWKG25O8MDWhx81hpkEvkHaAmgOhZPdOIS62e2cRI2zNibwWDz1rnNJExEOtPFF1yXgFw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

下面是表格的原文链接

> 参考链接：https://docs.gitlab.com/runner/executors/#selecting-the-executor

**为了简单起见，我下面的实践部分使用的是我自己的本地 Mac 机器作为 Executor，并且在注册时选择“Shell”作为 Executor 类型。**

## **2.2 YML 文件的基本语法规则**

CI 流程的运行控制，决定于项目根目录下编写的配置文件—— **.gitlab-ci.yml**，正因如此，我们需要掌握 YML 的基本语法规则。

YML 是一种编写配置文件的语言，比 JSON 更为简洁和方便，因此，我们首先要掌握的就是 YML 文件的编写语法。

一份简单的 YML 文件长下面这个样子：

```
install-job: # 注释
  tags:
    - sss
  stage: install
  script:
    - npm install
```

从这里我们就可以看出：

- YML 通过缩进组织层级
- YML 里允许通过#符号编写注释

在基本结构上，YML 和 JSON 也类似

- JSON 是由对象，数组，以及对象和数组的嵌套结构组成的
- YML 也是由对象，数组，以及对象和数组的嵌套结构组成的。

如下所示：

**YML 中的数组写法**

```
colors
  - red
  - blue
  - yellow
```

相当于 JSON 中的

```
{ "colors": ["red","blue","yellow"] }
```

**YML 中的对象写法：**

```
people:
  name: zhangsan
  age: 14
```

相当于 JSON 中的

```
{
  "people": {
     "name": "zhangsan"
     "age": 14
  }
}
```

当然也可以是数组和对象之间形成的嵌套结构

```
a:
  b:
    - d
  c: e
```

相当于

```
{
  "a": {
    "b": [ "d" ],
    "c": "e"
   }
}
```

**从 JSON 到 YML 之间的过渡学习的注意要点：**

- 你不再需要“{}”这种符号去区分层级边界了，你需要考虑使用缩进
- 这里可以使用注释，用#符号
- 如果不涉及特殊符号比如“[”，你一般是不需要給 YML 中的字符串加双引号或者单引号的（当然加了也可以）

了解了这些，对于编写一个 gitlab-ci 的**hello world**已经没有问题了。

当然 YML 还有着比 JSON 更为丰富的功能，比如用”&"符号和"<<:\*”符号可以实现的片段导入的功能，以及 gitlab-ci 提供的 include 关键字和 extend 关键字等提供的结构编排功能。这些我将在最后面的小节中讲解，这里暂时不多赘述

2.3 gitlab-ci.yml 配置的特定关键字

在了解了 YML 文件的语法格式后，接下来需要了解的就是 gitlab-ci 独特的配置关键字，这些关键字将在.gitlab-ci.yml 中使用，并用来控制一个 pipeline 具体的运作过程

gitlab 提供了很多配置关键字，其中最基础和常用的有这么几个

- stages
- stage
- script
- tags

**stages & stage**

stages 定义在 YML 文件的最外层，它的值是一个数组，用于定义一个 pipeline 不同的流程节点

例如我们定义如下:

```
stages: # 分段
  - install
  - eslint
  - build
  - deploy
```

则在 Gitlab 交互界面中能够看到如下展示

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/bwG40XYiaOKnhD9PdakPWKG25O8MDWhx8cUicjOrb2HWyzVlGyw2TwFhF7HibVWibiaOs3SZPX1ZBJAG7DnFmOfVOvw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

我们上面说过：**Job 是 pipeline 的任务节点，它构成了 pipeline 的基本单元**

而 stage/script/tags 这三个关键字，都是作为 Job 的子属性来使用的,如下所示，install 就是我们定义的一个 Job

```
install:
  tags:
    - sss
  stage: install
  script:
    - npm install
```

**stage**

是一个字符串，且是 stages 数组的一个子项，表示的是当前的 pipeline 节点。

当前 stage 的执行情况能在交互面板上能看的清清楚楚：

- 正在执行是蓝色
- 尚未执行是灰色
- 执行成功是绿色
- 执行失败是红色

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/bwG40XYiaOKnhD9PdakPWKG25O8MDWhx8tWM1KcDtuYsuT57SbdrOp0iaUe7rqtibWlVMuRppJnYahr3a5gfSZXLw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![图片](https://mmbiz.qpic.cn/mmbiz_png/bwG40XYiaOKnhD9PdakPWKG25O8MDWhx8OPHiazKbibIlqUH9e3px03c0zkNPJuia9saPCwKM9J86knkmyQCHtRnMw/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**script**

它是当前 pipeline 节点运行的 shell 脚本（以项目根目录为上下文执行）。

这个 script 是我们控制 CI 流程的核心，我们所有的工作：从安装，编译到部署都是通过 script 中定义的 shell 脚本来完成的。

如果脚本执行成功，pipeline 就会进入下一个 Job 节点，如果执行失败那么 pipeline 就会终止

**tags**

tags 是当前 Job 的标记，**这个 tags 关键字是很重要，因为 gitlab 的 runner 会通过 tags 去判断能否执行当前这个 Job**

例如我们在 gitlab 的面板中能看到当前激活的 runner 的信息

```
Gitlab项目首页=> setting => CI/CD => Runners
```

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/bwG40XYiaOKnhD9PdakPWKG25O8MDWhx8eGBLFLMfGzsFicvnOmf1ZeNFSSfYyGic4ay26Wvms8NPzJq2hs3wEQoQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

上面的这个 sss 就是当前 Runner 的 tags，这意味着：**这个 runner 只会执行 tag 为 sss 的 Job。**如果一个 Job 没有 tag 或者 tag 不是 sss,那么即使这个 Runner 是激活且空闲的，也不会去执行！

基本的 gitlab-ci 关键字就介绍结束了，有了这些知识对于编写一个 gitlab-ci 的”hello world”已经足够了。对于更多的关键字的相关知识，将在文章最后再进行介绍

## 三.gitlab-ci 实战

## **3.1 编写一个 gitlab-ci 的"hello world"**

好，说了这么多终于到了实践的部分了，请原谅前面的啰嗦，下面我将会展示一下如何从零开始实践一个 gitlab-ci 的 Hello world:

**1.在平台上下载并安装 Gitlab-runner 命令行**

我是在 Mac 上跑的 ci，所以下面的适用于 OSX 系统（如果是其他平台，可自行参考以下官方链接中的相关资料）

> 参考资料：https://docs.gitlab.com/runner/install/

依次运行：

```
sudo curl --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-darwin-amd64
sudo chmod +x /usr/local/bin/gitlab-runner
```

> 备注：这个下载过程可能会很慢，你可以复制上面的链接到浏览器中翻墙下载，然后下载完毕再进行后续处理

**2.初始化 gitlab-runner**

```
cd ~
gitlab-runner install
gitlab-runner start
```

最后输出如下，说明成功了。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/bwG40XYiaOKnhD9PdakPWKG25O8MDWhx8yufnwOFEPaKmml6YpMT79UfJxqgdz1XLOwUvYWoRl9ZVZicAwOBdFbQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**3.注册 Runner**

运行完 gitlab-runner start 只是完成了初始化，接下来你还要通过注册才能运行 runner

> 参考链接：https://docs.gitlab.com/runner/register/index.html

注册 runner 只需要一条运行命令：

```
sudo gitlab-runner register
```

然后写入相应的 token,url 和 tag 等相关信息，就注册完毕了。

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/bwG40XYiaOKnhD9PdakPWKG25O8MDWhx8hIJOWRhLVicELwuO14PrQgic2JfYAT5qiboUbEnDrCUaQUOsScZAQKPicg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

上面要求输入的 Runner 绑定的 token 和 url， 获取方式如下：

```
Gitlab项目首页=> setting => CI/CD => Runners => Specific Runners
```

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**4.激活 Runner**

注册完了可能还需要激活，这时我们可以看下面板，如果有个黑色的感叹号，这说明 runner 注册成功了，但是尚未激活（**如果是绿色的说明已经激活，本步骤跳过**）

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/bwG40XYiaOKnhD9PdakPWKG25O8MDWhx81TXabpvIyWJHicJrJwSPPbwSiaRHrlicX5TnUaKQ0Mv0qrffqG9iaxyhXA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

激活方法是本地运行：

```
sudo gitlab-runner verify
```

输出

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/bwG40XYiaOKnhD9PdakPWKG25O8MDWhx8dvrVH1eoWbSG2tk7EutWL9qJJccMXFkED9SyUicDuqicpXCP7avpaIcQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这时候回去看 Gitlab 面板里的 Runner 参数：

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/bwG40XYiaOKnhD9PdakPWKG25O8MDWhx8s8gS5Z8Wy8ECylS0YeTjOpTdTtavrWj9A0t53VsqQ8UIdCiaOrJty1w/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

是绿色就说明激活成功了

**5.梳理和规划 Pipeline 的不同阶段和过程**

在编写.gitlab-ci.yml 前，首先需要考虑的是我们的 pipeline 分几个阶段处理。

从前端工程师的角度出发，一个前端项目的 PipeLine 处理包括以下阶段

**<1> install 阶段**

就是执行 npm install 命令,根据 package.json 安装 node_modules 依赖包

**<2> eslint 阶段**

执行 eslint 检查，判断代码格式是否符合规范，如果不符合则 pipeline 终止。

在这之前，我先通过 npm install eslint 安装了 eslint 检查工具，然后在项目根目录下配置了.eslintrc 文件。这部分可自行参考相关资料，这里暂不多赘述。

**<3>build 阶段**

编译生成生产代码，可以通过 webpack 之类的打包工具执行编译。当然可以通过框架提供的编译命令进行编译，例如我这个示例项目是用 react-scripts 脚手架搭建的，所以通过 npx react-scripts build 进行编译。

**<4>deploy 阶段**

deploy 也就是部署阶段，也就是把刚才 bulid 阶段生成的生产代码，部署到生产访问的服务器上。这里又具体有以下两部分工作要做

**A.申请服务器 & 安装 web 服务 (准备工作)**

（1）我本次使用的是百度云的**云服务器**(每天 9 点的时候可以抢有一定免费使用期限的服务器)

（2）然后在本地终端通过 ssh 远程登陆服务器，并安装**apache**以提供 web 服务

```
sudo apt-get install apache2
```

（3）然后，安装完后的 apache 会在服务器下新增**/var/www/html/**目录， 这个目录就是存放网站资源的位置。

（4）最后我们只需要在每次部署的时候把生产的单页面拷贝到这个页面下，就能在浏览器上通过对应的 **IP+路径** 来访问 Web 页面了。

**B. 部署资源（每次 pipeline 都进行）**

我下面的示例中，是通过 **scp** 这一命令，将本地机器代码远程拷贝到云服务器上。

因为这一命令需要输入密码，所以通过 **sshpass** 命令携带密码再执行 scp：

```
sshpass -p $PASSWORD scp -r ./build $CUSTOM_USERNAME@$CUSTOM_IP:/var/www/html
```

这里说明一下，**Gitlab 有自定义变量的功能**，例如我们觉得直接在 YML 中写入密码/账号等信息不太好，那么**可以通过美元符号\$写入一个预定义的变量，然后在 Gitlab 面板上输入它**

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/bwG40XYiaOKnhD9PdakPWKG25O8MDWhx8X5Cahaqm6NFib3m4C3R3PsDlSTFAfSnicyzIoib075u22WrlvyzicGzIpw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

**7.编写.gitlab-ci.yml 配置文件**

回顾一下之前 YML 语法规则和 gitlab-ci 配置关键字的知识，就不难编写出以下 YML 文件

```
stages: # 分段
  - install
  - eslint
  - build
  - deploy

cache: # 缓存
  paths:
    - node_modules
    - build

install-job:
  tags:
    - sss
  stage: install
  script:
    - npm install

eslint-job:
  tags:
    - sss
  stage: eslint
  script:
    - npm run eslint

build-job:
  tags:
    - sss
  stage: build
  script:
    - npm run build

deploy-job:
  tags:
    - sss
  stage: deploy
  script:
    - sshpass -p $PASSWORD scp -r ./build $CUSTOM_USERNAME@$CUSTOM_IP:/var/www/html
```

**package.json**如下

```
{
  ...
  "scripts": {
    "start": "react-scripts start",
    "build": "npx react-scripts build",
    "eslint": "eslint ./src",
  },
  ....
}
```

**8.提交项目代码**

OK！终于到最后一步了，commit 然后 push

可以看到运行如下

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/bwG40XYiaOKnhD9PdakPWKG25O8MDWhx8S9VZw3JYO23zZgwIZFAEHaQ2dIohmlPj24GNxbibggp0JXkjp6ZXzWQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

跑完后

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/bwG40XYiaOKnhD9PdakPWKG25O8MDWhx8eg6xK2iaTicln4eYxFUrWgcSFvf2y5V8OSP77AzZnZVAqPohqaLJLgkA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

最后输入我们部署的 IP 看看我们部署的网页

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/bwG40XYiaOKnhD9PdakPWKG25O8MDWhx8W1pTBAGibSuCPVHgEKdOX7BHWFHu7LFaSiclComjIvz5gc8AGiaIsR7Qg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

发现已经把我们的项目代码部署上去了

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

## 四. Gitlab-ci 坑点详解

```
说多了都是泪。。。。。
```

下面总结一下使用过程中遇到的典型坑点

**1.Runner 未激活问题**

有时候注册之后，查看面板上的 Runner 信息，可能会发现 Runner 处在未激活状态

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

解决方法：

运行以下命令重新启动 runner

```
sudo gitlab-runner verify
sudo gitlab-runner restart
```

**2.Job 一直挂起，没有 Runner 来处理**

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/bwG40XYiaOKnhD9PdakPWKG25O8MDWhx8YibhhIgXp1Xxy7iaaE6g1C13YvroSYzWSXfTKjsdCk6dxHpM3PhI1LCg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

1.首先考虑的是不是 Runner 没有激活，如果没有那么按上面方式处理

2.还可能是 tag 没有匹配到，上面说过，Runner 注册时是要填写绑定 tag 的，如果你在 YML 里面编写 Job 没有带上 tag 是不会有自定义 Runner 来处理。解决方法:给 Job 加 tags

3.最后一种可能：你连续注册了多个 Runner,这些 Runner 冲突了，或者是新注册的 Runner 和旧 Runner 使用了同一个 token,这时候的解决方法如下：

**先删掉本地其他旧的 Runner**

```
sudo gitlab-runner unregister --all-runners
```

然后重置 token,并使用更新后的 token 重新注册一个 Runner

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

**3.specific Runner 被 Share Runner 抢占了 Job**

有时候你可能会发现：你的 Job 并没有被你新建的 Runner 执行，而是被 Share Runner 抢先执行了。你如果不想要 Share Runner，你可以在 Gitlab 面板上关掉

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/bwG40XYiaOKnhD9PdakPWKG25O8MDWhx8wvW2U1X4I1mPIfec9CExZL8gadmEpdoicw8qHxFz2kVDzUXfWPW3iadg/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## 五.gitlab-ci 进阶

## 5.1 YML 的片段复用和模块化

上面我们编写了 gitlab-ci 的**"hello world"**。但在实际项目的运行中，.gitlab-ci.yml 的编写可能会渐趋复杂。那么这个时候 YML 的一些其他语法功能就派上用场了，上面我们将 JSON 和 YML 的基本结构做比较并发现它们的相似之处，但实际上，**YML 提供了比 JSON 更为丰富的功能。**

**YML 的片段复用功能**

**试思考**：如果有一段配置片段会被很多 Job 使用，那么如果重复写入片段，则会使我们的 YML 文件变得过分冗长。

而如果能把这段提前进行定义，并根据别名进行导入，就能让 YML 文件简洁很多了。

**YML 的语法天然提供了这个功能:**

- 使用 **&**符号可以定义一个片段的别名
- 使用 **<<**符号和 **\*** 符号可以将别名对应的 YML 片段导入

```
.common-config: &commonConfig
  only: # 表示仅在develop/release分支上执行
    refs:
      - develop
      - release

install-job:
  # 其他配置 ....
  <<: *commonConfig
build-job:
  # 其他配置 ....
  <<: *commonConfig
```

**YML 的模块化功能**

试思考，如果我们配置脚本很长的话，我们一定要把它写在.gitlab-ci.yml 这单独一个文件里吗？

能否将它分成多个 yml 文件，然后把其他 YML 文件导入到入口 YML 文件(.gitlab-ci.yml)中呢。

gitlab-ci 提供的 include 关键字便可实现这个功能, 它可以用来导入外部的 YML 文件。

例如我们有如下的 YML 结构

```
 ├── .gitlab-ci.h5.yml'
 ├── .gitlab-ci.bd.yml'
 ├── .gitlab-ci.wx.yml
 └── .gitlab-ci.yml
```

**那么在.gitlab-ci.yml 中这么写，就可以对它们做合并**

```
include:
  - '/.gitlab-ci.wx.yml'
  - '/.gitlab-ci.bd.yml'
  - '/.gitlab-ci.h5.yml'
```

gitlab-ci 还提供了 extend 关键字，它的功能和前面提到的 YML 的片段导入的功能是一样的，

不过可读性更好一些。

```
.common-config:
  only: # 表示仅在develop/release分支上执行
    refs:
      - develop
      - release

install-job:
  # 其他配置 ....
  extends: .common-config

build-job:
  # 其他配置 ....
  extends: .common-config
```

## 5.2 Gitlab-ci 的其他配置项

## **cache 关键字**

cache 关键字是需要特别着重讲的一个关键字。顾名思义，它是用来做缓存的。

为什么要做缓存呢？当然是为了**重复运行 pipeline 的时候不会重复安装全部 node_modules 的包**，从而减少 pipeline 的时间，提高 pipeline 的性能。

**但是，这并不是 cache 关键字唯一的功能！**

在介绍 cache 的另外一个功能之前，我要先说一下 gitlab-ci 的一个优点“恶心人”的特点：

它在运行下一个 Job 的时候，会默认把前一个 Job 新增的资源删除得干干静静

![图片](https://mmbiz.qpic.cn/mmbiz_jpg/bwG40XYiaOKnhD9PdakPWKG25O8MDWhx8ibDka7kX0WpokHqrLBGh5YxgkaMp2UPftqeoWp5qxam6AyTRTMib32BQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

没错，也就是说，我们上面 bulid 阶段编译生成的包，会在 deploy 阶段运行前被默认删除！（我生产包都没了我怎么部署 emmmmmmm）

而 cache 的作用就在这里体现出来了：如果我们把 bulid 生产的包的路径添加到 cache 里面，虽然 gitlab 还是会删除 bulid 目录，但是因为在删除前我们已经重新上传了 cache，并且在下个 Job 运行时又把 cache 给 pull 下来，那么这个时候就可以实现在下一个 Job 里面使用前一个 Job 的资源了

总而言之，cache 的功能体现在两点：

- **在不同 pipeline 之间重用资源**
- **在同一 pipeline 的不同 Job 之间重用资源**

虽然 cache 会缓存旧的包，但我们并不用担心使用到旧的资源，因为 npm install 还是会如期运行，并检查 package.json 是否有更新，npm build 的时候，生成的 build 资源包也会覆盖 cache,并且在当前 Job 运行结束时，作为**"新的 cache"**上传

## artifacts 关键字

这个关键字的作用是：将生成的资源作为 pipeline 运行成功的附件上传，并在 gitlab 交互界面上提供下载

例如我们新增以下 YML

```
Build-job:
  stage: build
  script:
  - 'npm run build'
  artifacts:
    name: 'bundle'
    paths:
      - build/
```

pipeline 跑完后是这样的：

![图片](data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVQImWNgYGBgAAAABQABh6FO1AAAAABJRU5ErkJggg==)

## image/services

这两个关键字可使用 Docker 的镜像和服务运行 Job，具体可参考 Docker 的相关资料,这里暂不多加叙述

## only/except

这两个关键字后面跟的值是 tag 或者分支名的列表。

故名思义

- only 的作用是指定当前 Job 仅仅只在某些 tag 或者 branch 上触发
- 而 except 的作用是当前 Job 不在某些 tag 或者 branch 上触发

```
job:
  # use regexp
  only:
    - /^issue-.*$/
    - develop
    - release
```

## allow_failure

值为 true/false, 表示当前 Job 是否允许允许失败。

- 默认是 false,也就是如果当前 Job 因为报错而失败，则当前 pipeline 停止
- 如果是 true，则即使当前 Job 失败，pipeline 也会继续运行下去。

```
job1:
  stage: test
  script:
    - execute_script_that_will_fail
  allow_failure: true
```

## retry

顾名思义，指明的是当前 Job 的失败重试次数的上限。

但是这个值只能在 0 ～ 2 之间，也就是重试次数最多为 2 次，包括第一次运行在内，Job 最多自动运行 3 次

## timeout

配置超时时间，超过时间判定为失败

```
Job:
  script: rspec
  timeout: 3h 30m
```

## When

表示当前 Job 在何种状态下运行，它可设置为 3 个值

- **on_success**: 仅当先前 pipeline 中的所有 Job 都成功（或因为已标记，被视为成功 allow_failure）时才执行当前 Job 。这是默认值。
- **on_failure**: 仅当至少一个先前阶段的 Job 失败时才执行当前 Job。
- **always**: 执行当前 Job，而不管先前 pipeline 的 Job 状态如何。

关于本文

来自：广发证券 澎湖湾

https://zhuanlan.zhihu.com/p/184936276
