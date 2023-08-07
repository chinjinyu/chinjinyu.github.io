# 使用MkDocs搭建个人博客

接触编程已经好几年了，阅读了无数大佬的博客文章，但是从来没有自己写过。这其中最重要的原因当然是懒惰，觉得写博客太费时间了，对自己的帮助也不大。可是如今发现自己的记性越来越捉急了，某样技术在一段时间不接触之后，就几乎完全忘记了。对此我很是苦恼，于是萌生了写博客的想法，毕竟好记性不如烂笔头嘛。曾经听说过GitHub上可以搭建博客，也看到很多人是这样做的，正好借这个机会好好研究下。

## MkDocs介绍

根据[MkDocs](https://www.mkdocs.org/)官网介绍：

> MkDocs is a **fast**, **simple** and **downright gorgeous** static site generator that's geared towards building project documentation. Documentation source files are written in Markdown, and configured with a single YAML configuration file. 

MkDocs是一个**快速**、**简单**、**华丽**的静态站点生成器，适用于构建项目文档。文档源文件使用Markdown编写，并使用单个YAML配置文件进行配置。

MkDocs的特点有：

- 多个漂亮主题
- 易于个性化
- 实时预览
- 随处托管

## 快速搭建

### 新建GitHub仓库

打开[GitHub](https://github.com/)官网并登录，新建一个仓库，名称必须为 `<username>.github.io`

![20230806111851.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230806111851.png)

将这个仓库克隆到本地，这里重命名为 `github-pages`

```bash
git clone https://github.com/<username>/<username>.github.io.git github-pages
```

### 安装MkDocs程序

进入 `github-pages` 目录

```bash
cd github-pages
```

创建python虚拟环境，这里使用 `pipenv`

```bash
pipenv install
```

安装MkDocs程序

```bash
pipenv install mkdocs
```

进入python虚拟环境

```bash
pipenv shell
```

检查是否安装成功

```bash
mkdocs --version
```

如果安装成功，则应该会输出MkDocs的版本信息

![20230806225116.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230806225116.png)

### 创建MkDocs工程

在当前目录( `github-pages` )下新建工程

```bash
mkdocs new .
```

然后编译MkDos文档

```bash
mkdocs build
```

这时会在当前目录下生成 `site` 文件夹，里面保存着网站的静态文件

### 预览和部署

MkDocs支持在部署之前，实时预览网页内容

```bash
mkdocs serve -a 0.0.0.0:8000
```

其中的 `-a` 选项用于指定绑定的 `<IP:PORT>` ，如果不指定，则默认为 `localhost:8000`

如果预览发现问题，直接修改文档源文件，MkDocs检测到文件发生变化会自动更新预览内容

预览如果没有发现问题，接下来就可以部署到GitHub了

```bash
mkdocs gh-deploy --clean
```

执行结束后，查看GitHub仓库，发现多了一个 `gh-pages` 分支，这个分支里存放的就是 `site` 文件夹中的内容

最后一步，配置GitHub Pages的构建和部署分支，将 `Settings` -> `Pages` -> `Build and deployment` -> `Branch` 设置为 `gh-pages/(root)`，点击 `Save` 保存设置

![20230806125526.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230806125526.png)

这时理论上就可以正常访问自己刚刚搭建的博客了，在浏览器地址栏中输入 `https://<username>.github.io/` ，若出现MkDocs的欢迎页则说明部署成功

![20230806125801.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230806125801.png)

## 配置网页

MkDocs工程的所有配置都集成在 `mkdocs.yml` 文件中

### 修改网页名称

网页名称就是显示在浏览器标签以及网页左上位置的内容，默认为 `My Docs`，可以在 `mkdocs.yml` 文件中的设置 `site_name` 来改变

```yaml title='mkdocs.yml'
site_name: <网页名称>
```

### 修改主题

MkDocs支持多个主题，内置主题有 `mkdocs` 和 `readthedocs` ，第三方主题可以在[MkDocs Wiki](https://github.com/mkdocs/mkdocs/wiki/MkDocs-Themes)找到，也可以自定义主题

这里我是用的是比较受欢迎的 `material` 主题

```bash
pipenv install mkdocs-material
```

`material` 主题的详细配置说明可以参阅[Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)，这里我的配置如下

```yaml title='mkdocs.yml'
theme:
name: material
language: zh
hljs_languages:
- yaml
markdown_extensions:
- toc:
  permalink: true
- pymdownx.highlight:
  linenums: true
  anchor_linenums: true
- pymdownx.inlinehilite
- pymdownx.superfences
```

- `name` 指定使用的主题名称
- `language` 指定网页语言
- `hljs_languages` 指定额外需要高亮的语言
- `toc` 用于设置目录，`permalink: true` 表示为网页中的一级/二级/三级/...标题生成链接
- `pymdownx.highlight` 用于设置代码高亮，`linenums: true` 表示显示行号，`anchor_linenums: true` 用于为代码块的行号生成链接
- `pymdownx.inlinehilite` 启用对内联代码高亮的支持
- `pymdownx.superfences` 允许在代码和内容块之间任意嵌套

最终效果如下

![20230807003537.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230807003537.png)

## 发布文章

MkDocs的文章以markdown文件的形式存储在docs文件夹下，其中 `index.md` 为主页

发布新文章时，在docs目录下新建 `<article>.md` 文件并编辑

然后修改 `mkdocs.yml` 文件，将新文章加到导航栏中

```yaml title='mkdocs.yml'
nav:
  - 主页: index.md
  - <标题>: <article>.md
```

标题可以不设置，MkDocs按照 nav配置的标题 > 文章中定义的标题(h1) > 文件名 的顺序进行推断

如果要使用多级导航，则可以这样配置

```yaml title='mkdocs.yml'
nav:
  - 主页: index.md
  - <分类>:
    - <标题>: <aritle>.md
```

然后执行 `mkdocs gh-deploy` 命令部署到GitHub

## 自动部署

一般会将MkDocs源文件也推送到GitHub以防止丢失，这里我就直接推送到 `main` 分支

新建 `.gitignore` 文件，忽略掉除 `docs` 、 `mkdocs.yml` 以及 `git` 相关文件之外的文件

```title='.gitignore'
/*
!/docs
!/mkdocs.yml
!/.gitignore
!/.github
```

然后我们在编写完文章后，一般至少都要执行这些命令

```bash
git add .
git commit -m 'new article'
git push    # 第一次push时执行git push -u origin main
mkdocs gh-deploy
```

本着省一点是一点的原则，前面几个 `git` 相关的命令无法省略，但 `mkdocs gh-deploy` 可以通过GitHub Actions来帮助我们自动完成

在仓库根目录下新建 `.github/workflows` 文件夹

```bash
mkdir -p .github/workflows
```

在 `.github/workflows` 文件夹下新建 `gh-deploy.yml` 文件，其他文件名也可以，内容如下，具体含义可以参考[Deploy MkDocs](https://github.com/marketplace/actions/deploy-mkdocs)

```yaml title='gh-deploy.yml'
name: Publish docs via GitHub Pages
on:
  push:
    branches:
      - main
jobs:
  build:
    name: Deploy docs
    runs-on: ubuntu-latest
    steps:
      - name: Checkout main
        uses: actions/checkout@v2
      - name: Deploy docs
        uses: mhausenblas/mkdocs-deploy-gh-pages@master
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          EXTRA_PACKAGES: build-base
```

在GitHub仓库页面中，将 `Settings` -> `Actions` -> `General` -> `Workflow Permissions` 设置为 `Read an write permissions`，点击 `Save` 保存设置

![20230806212318.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230806212318.png)

这样每次编写完文章，只需成功执行 `git push` ，GitHub就会自动帮助我们部署

## 参考资料

1. [MkDocs官网](https://www.mkdocs.org/)
2. [MkDocs中文文档](https://hellowac.github.io/mkdocs-docs-zh/)
3. [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)
4. [Mkdocs 配置和使用](https://zhuanlan.zhihu.com/p/383582472)
5. [Deploy MkDocs](https://github.com/marketplace/actions/deploy-mkdocs)
