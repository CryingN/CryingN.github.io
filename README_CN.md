# 学术网站

![pages-build-deployment](https://github.com/academicpages/academicpages.github.io/actions/workflows/pages/pages-build-deployment/badge.svg)

Academic Pages 是用于学术网站的 Github Pages 模板。

# 开始

1. 如果您没有 GitHub 帐户，请注册一个帐户并确认您的电子邮件（必填！
1. 单击右上角的“使用此模板”按钮。
1. 在“新建仓库”页面上，输入您的仓库名称为“[您的 GitHub 用户名].github.io”，这也是您网站的 URL。
1. 设置站点范围的配置并添加您的内容。
1. 将任何文件（如 PDF、.zip 文件等）上传到“files/”目录。它们将显示在 https://[您的 GitHub 用户名].github.io/files/example.pdf。
1. 通过转到“GitHub 页面”部分的存储库设置来检查状态
1. （可选）使用“markdown_generator”文件夹中的 Jupyter 笔记本或 python 脚本，从 TSV 文件生成用于发布和讨论的 markdown 文件。

欲了解更多信息，请访问： https://academicpages.github.io/

## 本地运行

当您最初使用您的网站时，能够在将更改推送到 GitHub 之前在本地预览更改非常有用。要在本地工作，您需要：

1. 克隆存储库并进行更新，如上所述。
1. 确保已安装 ruby-dev、bundler 和 nodejs
    
    在大多数 Linux 发行版和 [Windows 子系统 Linux](https://learn.microsoft.com/en-us/windows/wsl/about) 的命令为:
    ```bash
    sudo apt install ruby-dev ruby-bundler nodejs
    ```
    在 MacOS 上，命令为：
    ```bash
    brew install ruby
    brew install node
    gem install bundler
    ```
    在 Archlinux 上, 命令为:
    ```bash
    sudo pacman -S ruby-bundler nodejs
    ```
1. 运行 'bundle install' 以安装 ruby 依赖项。如果遇到错误，请删除 Gemfile.lock，然后重试。
1. 运行 'jekyll serve -l -H localhost' 生成 HTML 并从 'localhost：4000' 提供它，本地服务器将在更改时自动重建并刷新页面。

如果您在 Linux 上运行，则可能需要先安装一些额外的依赖项，然后才能在本地运行：
`sudo apt install build-essential gcc make`

# 维护

对模板的 Bug 报告和功能请求应 [通过 GitHub 提交](https://github.com/academicpages/academicpages.github.io/issues/new/choose). 有关如何设置模板样式的问题，请随时在 GitHub 上开始 [新讨论](https://github.com/academicpages/academicpages.github.io/discussions).

这个仓库被 [Stuart Geiger 进行fork](https://github.com/staeiou) 来自 [Minimal Mistakes Jekyll 主题](https://mmistakes.github.io/minimal-mistakes/), which is © 2016 Michael Rose and released under the MIT License (see LICENSE.md). 它目前由 [Robert Zupko](https://github.com/rjzupkoii) 维护, 欢迎更多的维护者.

## Bug 修复和增强功能

如果您有 bug 修复和增强功能想要作为拉取请求提交，您将需要 [fork](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/fork-a-repo) 此存储库，而不是将其用作模板。这也将允许您 [同步您的副本](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-forks/syncing-a-fork) 的模板也添fork加到您的中。

不幸的是，像 Academic Pages 这样的模板主题存在一个后勤问题，这使得获得核心主题的错误修复和更新变得有点棘手。如果您使用此模板并对其进行自定义，则在尝试同步时可能会遇到合并冲突。如果你想保存你的各种.yml配置文件和 markdown 文件，你可以删除仓库并再次复刻它。或者，您可以手动修补。
