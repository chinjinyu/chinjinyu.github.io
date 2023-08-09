# 博客园设置Silence主题

博客园的提供了一百多个默认主题，但我还是找不到一个干净、简洁、美观的。了解到博客园可以自定义主题，于是在网上搜寻相关内容，终于被我发现了Silence——一个专注于阅读的博客园主题。

## Silence介绍

[Silence](https://github.com/esofar/cnblogs-theme-silence)是一个专注于阅读的博客园主题

- 界面简洁优雅，响应式网页设计
- 轻量配置，文档给力，非常容易使用
- 提供暗黑模式和多种色彩主题，可随时切换
- 支持自定义导航栏菜单项、悬浮标题目录等
- 提供文章版权签名、赞赏功能等
- 项目结构清晰，代码简单，可实现高度定制化开发

## 部署主题

在进行下面的操作之前，需要申请博客园的JS权限，参考[博客园申请JS权限教程](https://www.cnblogs.com/maczhen/p/14372738.html)，这里不做介绍

成功开通JS权限后，进入博客园后台“设置”页面

在“基本设置”中设置博客标题，并选择 `Custom` 作为博客皮肤，注意博客子标题不用设置，因为Silence不支持

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230807111013.png)

在“代码高亮”中设置渲染引擎为 `highlight.js`，取消勾选“显示行号”，并将系统主题设为 `cnblogs`

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230807110903.png)

在“页面定制CSS代码”中勾选“禁用模板默认CSS”，并在输入框中填入如下内容

```css
@import url(https://cdn.jsdelivr.net/gh/esofar/cnblogs-theme-silence@latest/dist/silence.min.css)
```

可以通过 `url` 中的 `@` 指定主题的版本，这里的 `@latest` 表示使用最新的版本，也可以指定其他版本，比如 `@3.0.0-rc2` 、 `@3.0.0-beta4` 等

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230807113323.png)

在“页脚HTML代码”中填入如下代码

```html
<script>
  window.$silence = {
    avatar: 'https://files.cnblogs.com/files/blogs/611458/avatar.ico',
    favicon: 'https://files.cnblogs.com/files/blogs/611458/favicon.ico',
    github: 'https://github.com/chinjinyu',
    defaultMode: 'auto',
    defaultTheme: 'a',
    navbars: [{
      title: '标签',
      url: 'https://www.cnblogs.com/chinjinyu/tag/'
    }, {
      title: '归档',
      url: 'https://www.cnblogs.com/chinjinyu/p/'
    }],
    showNavAdmin: true,
    catalog: {
      enable: true,
      index: true,
      active: false,
      levels: ['h2', 'h3', 'h4'],
    },
    signature: {
      enable: true,
      author: 'chinjinyu',
      license: ['署名-非商业性使用-相同方式共享 4.0 国际', 'https://creativecommons.org/licenses/by-nc-sa/4.0/'],
    },
    sponsor: {
      enable: true,
      wechat: 'https://files.cnblogs.com/files/blogs/611458/wechat.ico',
      alipay: 'https://files.cnblogs.com/files/blogs/611458/alipay.ico'
    }
  };
</script>
<script src="https://cdn.jsdelivr.net/gh/esofar/cnblogs-theme-silence@latest/dist/silence.min.js"></script>
```

注意这里最后一行 `src` 中 `@` 的版本需要和“页面定制CSS代码”中的保持一致

* `avatar` : 博客主页左侧栏的头像
* `favicon` : 网页标题前的小图标
* `github` : 个人的GitHub主页地址，博客左上角有个GitHub挂件，点击时会进行跳转
* `defaultMode` : 主题模式，有 `light` / `dark` / `auto` 三种选项
* `defaultTheme` : 默认加载的主题色彩，`a` 表示蓝色，其他值参考[官方文档](https://esofar.github.io/cnblogs-theme-silence/#/options?id=defaulttheme)
* `navbars` : 设置导航栏中的自定义菜单项
	* `title` : 导航栏显示的标题
	* `url` : 地址
	* `children` : 设置二级菜单项
		* `title` : 二级菜单标题
		* `url` : 地址
		* `target` : `_blank` 表示在新窗口中打开
* `showNavAdmin` : 是否显示导航栏中的“管理”菜单项
* `catalog` : 设置博文的标题目录
	* `enable` : 是否启用目录
	* `index` : 是否在目录标题前添加索引
	* `active` : 是否直接显示目录
	* `levels` : 设置需要生成目录的标题等级
* `signature` : 设置版权签名
	* `enable` : 是否启用签名
	* `author` : 作者
	* `license` : 知识共享许可协议
	* `remark` : 其他备注
* `sponsor` : 设置打赏
	* `enable` : 是否启用打赏
	* `text` : 打赏的提示信息
	* `paypal` : PayPal收款码
	* `wechat` : 微信收款码
	* `alipay` : 支付宝收款码

在“页首HTML代码”输入框中填入如下代码

```html
<div class="loading">
  <div class="box">
    <h2>Loading</h2>
    <span></span><span></span><span></span><span></span><span></span><span></span><span></span>
  </div>
</div>
```

## 代码高亮及显示行号

Slicence最新版本不能代码高亮和显示行号了，参见GitHub上这个[Issue #191](https://github.com/esofar/cnblogs-theme-silence/issues/191)，代码块的展示效果像这样

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230807114927.png)

这对阅读代码造成了一定的不便，针对这个问题有两种解决方案

一个是退回 `3.0.0-beta4` 以及之前版本的Silience，并在配置中添加 `hljsln: true` 开启显示行号

另一个做法是，在“代码高亮”中设置渲染引擎为 `prismjs` ，勾选“显示行号”，并将系统主题设为 `prism-xonokai`

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230807115302.png)

效果如下

![image.png](https://cdn.jsdelivr.net/gh/chinjinyu/image-hosting-website@main/images/20230807115323.png)

## 参考资料

1. [Silence项目](https://github.com/esofar/cnblogs-theme-silence)
2. [Silence文档](https://esofar.github.io/cnblogs-theme-silence/#/?id=intro)
3. [博客园申请JS权限教程](https://www.cnblogs.com/maczhen/p/14372738.html)
4. [新版本不能代码高亮了吗？](https://github.com/esofar/cnblogs-theme-silence/issues/191)
