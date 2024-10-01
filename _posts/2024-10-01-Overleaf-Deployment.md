---
layout: post
title:  使用官方开源项目搭建自有Overleaf服务
categories: [IT建设,科研]
# excerpt: In graphic design, a pull quote (also known as a lift-out pull quote) is a key phrase, quotation, or excerpt that has been pulled from an article and used as a page layout graphic element, serving to entice readers into the article or to highlight a key topic.
---

Overleaf对于使用LaTex编辑论文的科研工作者和研究生来说扮演着重要的角色，可以说贯穿了从论文初稿的撰写到提交的全过程，而Overleaf作为一个由商业公司运作的云服务产品，即便对免费用户施加的限制很少（只有编译时间和频率的限制），也不足以覆盖研究工作者全部常见的应用场景，加上其每月9每月的订阅费用，对于中国的研究工作者而言，还是贵了一些。

不过好在Overleaf对其产品做了开源，这就为广大科研工作者利用研究机构的提供的硬件资源部署自有的Latax排版服务。

本篇将提供使用官方开源项目在服务器上部署Overleaf服务的教程。

# 基础环境要求和配置

硬件资源需要能够流畅运行Ubuntu操作系统，且有足够的内存和硬盘空间。

本篇所使用的软件配置：
1. Ubuntu 20.04 LTS
2. Docker 4.33.0
3. Git

# 配置步骤

1. 下载Overleaf官方源代码

```
git clone https://github.com/overleaf/toolkit.git ./overleaf-toolkit && cd overleaf-toolkit
```

下载完成之后，在overleaf-toolkit文件夹内有很多用来管理相关功能的脚本；

2. 初始化并修改配置
```
bin/init
```
这条命令会在`./config`目录下生成三个文件：
```
overleaf.rc     variables.env     version
```
这些文件中的配置可以根据需要修改，这里介绍几个常用的。

`overleaf.rc `文件中的下面几个配置一般来说需要一开始修改：
```
# 这里的IP地址可以替换成服务器的IP，如果是在自己电脑上本地部署，则可以忽略
OVERLEAF_LISTEN_IP=127.0.0.1
# 这里的端口是Overleaf服务的监听端口，这里用的是9000，只要不冲突即可
OVERLEAF_PORT=9000
```

3. 启动服务
输入下面的命令启动服务：
```
bin/up
```
如果是需要后台启动，请使用`-d`参数。

4. 补全宏包
默认情况下，overleaf使用的texlive中宏包是不全的，因此需要额外下载。

这里先输入
```
bin/shell
```
进入容器内部，执行下面的代码来下载补全宏包

```
# 下载并运行升级脚本
wget http://mirror.ctan.org/systems/texlive/tlnet/update-tlmgr-latest.sh
sh update-tlmgr-latest.sh -- --upgrade
# 更换texlive的下载源
tlmgr option repository https://mirrors.sustech.edu.cn/CTAN/systems/texlive/tlnet/
# 升级tlmgr
tlmgr update --self --all
# 安装完整版texlive（时间比较长，不要让shell断开）
tlmgr install scheme-full
```


至此，在浏览器中输入`IP:端口`即可访问自己部署的Overleaf服务了。
如果提示需要注册，可以访问`IP:端口/launchpad/`来注册。

# 一些其他可能需要的配置

对于很多需要自己部署Overleaf写作的人，往往是因为所需要编译的论文过大，这种过大的情况往往是毕业论文，而毕业论文往往要用到中文，因此，这里额外给出配置中文的一种方式

在文件中引入宏包
```
\usepackage[UTF8]{ctex}
```

在编辑页面的左上角的`menu`菜单中，更改Overleaf编译引擎为`XeLaTex`。

然后，Overleaf可以正常编译中文了。

如果因为一些格式的要求，规定不允许使用这种方式，读者可以自行搜索其他方式。


# 参考资料
https://yangzhang.site/Note/NAS/self-hosted-overleaf/
https://blog.wsine.top/posts/selfhost-overleaf-for-thesis/
https://blog.csdn.net/rolling0707/article/details/86220557
