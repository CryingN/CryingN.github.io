---
title: "Blackspider"
collection: procedure
category: manuscripts
permalink: /publication/blackspider
excerpt: '黑蜘蛛是基于blackarch渗透系统官网使用python编写的一套爬虫工具, 主要用于对渗透工具进行快速筛选, 也可以作为库快速编写自动化工具...'
date: 2023-06-16
venue: 'Jun 16'
slidesurl: 'https://gitee.com/cryingn/blackspider/archive/refs/tags/v1.2.0.zip'
paperurl: 'https://gitee.com/cryingn/blackspider'
citation: '作者: CryingN'
---

**注意:** 为支持中国VY开源社区的许可证, 1.0.3版本开始blackspider从BSD许可证更换为VY通用许可证

# 环境搭建

`Blackspider` 是由python3构建的网站关键词抓取工具, 可以通过获取源码后直接执行, 其中`Blackspider`使用到第三方库如下:

* `urllib3`

* `beautifulsoup`

* `colorama` (主要对windows的色彩环境进行初始化, 可选安装)

* `lxml` (xml文件解析器, 设置`get_tag`函数变量`feature="xml"`时需要该库支持, 可选安装)

以下为windows与linux的环境搭建方式:

## windows搭建

- 请确认电脑上已经配置好[python](https://www.python.org/downloads/)环境与pip工具包
  
  - 在终端(例如:`powershell`)中输入`python --version`以检验python环境是否安装成功;
  
  - 在终端(例如:`powershell`)中输入`pip --version`以检验可以正常使用python的工具包管理工具.

- 安装所需第三方库
  
  - 在终端(例如:`powershell`)输入`pip install urllib3 bs4`以安装`urllib3`库与`beautifulsoup`库
  
  - 在终端(例如:`powershell`)输入`pip list`检查`urllib3`库与`beautifulsoup`库是否正常安装

- 下载代码压缩包并解压

- 在解压的文件夹中找到make.py文件, 在当前目录下运行该文件:`python make.py`, 按脚本进行配置.

- 使用blackspider
  
  - 可以直接运行`blackspider.py`文件
  
  - 或者在终端(例如:`powershell`)输入`blackspider -f blackarch -t tr -a text -k python,xss`(后缀请参见[wiki:网站与参数](https://gitee.com/cryingn/blackspider/wikis/网站与参数))
  
  - `main.py`文件中大量调用了`blackspider.py`与`Manage.py`中的函数, 可仿照方法进行函数调用

## linux搭建

不同linux发行版的包管理器会有部分区别, 以下主要以archlinux与debian两个主流发行版进行举例:

### archlinux

- 使用前保证已经安装以下工具(使用pacman包进行安装):
  
  - git
  
  - python
  
  - python-beautifulsoup4
  
  - python-urllib3
  
  若还未安装可以使用以下命令安装: `sudo pacman -S git python python-beautifulsoup4 python-urllib3`

- `git clone https://gitee.com/cryingn/blackspider.git`

- `python make.py`

- 使用blackspider
  
  - 可以直接运行`blackspider.py`文件
  
  - 或者在终端(例如:`powershell`)输入`blackspider -f blackarch -t tr -a text -k python,xss`(后缀请参见[wiki:网站与参数](https://gitee.com/cryingn/blackspider/wikis/%E7%BD%91%E7%AB%99%E4%B8%8E%E5%8F%82%E6%95%B0))
  
  - `main.py`文件中大量调用了`blackspider.py`与`Manage.py`中的函数, 可仿照方法进行函数调用

### debian

- 使用前保证已经安装了以下工具:
  
  - git
  
  - python3
  
  - beautifulsoup4
  
  - urllib3

- `git clone https://gitee.com/cryingn/blackspider.git`

- `python3 make.py`

- 使用blackspider
  
  - 可以直接运行`blackspider.py`文件
  
  - 或者在终端(例如:`powershell`)输入`blackspider -f blackarch -t tr -a text -k python,xss`(后缀请参见[wiki:网站与参数](https://gitee.com/cryingn/blackspider/wikis/%E7%BD%91%E7%AB%99%E4%B8%8E%E5%8F%82%E6%95%B0))
  
  - `main.py`文件中大量调用了`blackspider.py`与`Manage.py`中的函数, 可仿照方法进行函数调用

```bash
=========================================================================
'||'''|,'||`            '||     .|'''|.            ..     ||`            
 ||   || ||              ||     ||                 ''     ||             
 ||;;;;  ||  '''|. .|'', || //` `|'''|,'||''|,'||''|| .|''|| .|''|,'||''|
 ||   || || .|''|| ||    ||\{   ..   || ||  || ||  || ||  || ||..|| ||
.||...|'.||.`|..||.`|..'.|| \\. '|...|' ||..|'.||..||.`|..||.`|... .||.
                                        ||
                                       .||
地  址：https://github.com/CryingN/blackspider
        https://gitee.com/cryingn/blackspider
版本号：[当前版本号]
邮  箱：CryingNights7v@gmail.com
=========================================================================
```

# 文件描述

| 核心文件                             | 文件描述                                                                      |
| -------------------------------- | ------------------------------------------------------------------------- |
| [blackspider.py](./blackspider)  | 工具的核心部分, 可直接通过该文件直接进行调试运行                                                 |
| [Manage.py](./Manage.py)         | 工具的空间管理部分, 主要用于检查文件是否完整与编辑内部文件                                            |
| [.Manage.ini](./.Manage.ini)     | 储存的文件数据, test储存调试用的测试数据, 易恢复, group储存数据由main.py文件直接管理. url中"[equal]"表示"=" |
| [main.py](./main.py)             | 配置好的工具入口, 一般通过该文件使用blackspider                                            |
| [\_\_pycache\_\_](./__pycache__) | 用于进行python文件之间的调用                                                         |
| [.git](./.git)                   | git管理工具                                                                   |
| [blackarch](./blackarch)         | 个人推荐默认储存的文件, 可直接快速获取需要的渗透工具                                               |
| [LINCENSE](./LINCENSE)           | VY通用许可证引用书                                                                |
| [make.py](./make.py)             | 一键构建足够舒适的blackspider环境                                                    |

## VY许可证说明

在不进行个人补充的情况下VY许可证又称为VY通用许可证, 公开使用时只需标注社(VYCMa.png)标或声明源码来自VYCMa, 便可以免费修改和商用素材.
对于分发问题, 为方便更多人理解, 在VY许可证中有重新定义"版权转移"概念: 他人修改源码后可以闭源, 每个修改过的文件需放置版权说明, 如果要进行公开展示需标注作者个人的标志,若作者无特殊说明需标注社标(VYCMa.png)标或声明源码来自VYCMa.

<img src='/images/VYCMa.png'>
