---
title: "自托管Utterances教程：基于Github Issues的轻量级博客评论系统"
categories:
  - Java
tags:
  - Utterances
  - 评论
  - 博客评论
  - 自托管
  - Serverless
---

本文主要介绍基于Github Issues的轻量级博客评论系统Utterances。文章前半部分主要介绍Utterances的配置与使用，后半部分主要介绍如何利用近年来流行的Serverless化平台，自托管Utterances。

## Utterances简介

Utterances是一个基于Github Issues的轻量级评论系统，可用于博客、Wiki等。它具有以下优点：

- 开源
- 不追踪用户，没有广告，始终免费
- 所有的数据都存储在Github Issues
- 样式基于[Primer](https://primer.style)设计语言
- 夜间模式
- 轻量级，使用Vanilla TypeScript，不使用网络字体，使用JavaScript框架或者Polyfill兼容旧浏览器。

## 快速上手

- 在GitHub上新建一个公开仓库，安装[Utterances GitHub App](https://github.com/apps/utterances)至该仓库。
- 在你的网页需要插入Utterances评论系统的位置，粘贴以下代码（username，reponame分别修改为你的GitHub用户名，仓库名）。

  ```js
  <script src="https://utteranc.es/client.js"
          repo="username/reponame"
          issue-term="pathname"
          theme="github-light"
          crossorigin="anonymous"
          async>
  </script>
  ```

完成以上操作后刷新网页就可以使用Utterances了。

如果想进一步配置或者自托管Utterances，可以继续下面的内容。

## 配置

下文将介绍以下代码中`repo`，`issue-term`，`label`，`theme`几个选项的配置。

```js
<script src="https://utteranc.es/client.js"
        repo="username/reponame"
        issue-term="pathname"
        label="💬comment"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>
```

### `repo`：设置存放评论的仓库

Utterances 使用Github Issues存储评论，所以需要一个仓库。你可以新建一个公开仓库专门用来放评论，也可以使用原有的仓库。要设置存放评论的仓库只需要将`repo="username/reponame"`这一行中的`username`改为你的GitHub用户名，`reponame`改为你的仓库名，其它不变。

仓库需满足以下条件：

- 仓库必须为公开仓库，私有仓库访客无法查看对应Issues上的评论。
- 确保在仓库中安装了[Utterances GitHub App](https://github.com/apps/utterances)，否则用户将无法发表评论。
- 如果你的仓库是派生(fork)出的，请在仓库的`Settings`选项确认`Features`区`Issues`已勾选。

### `issue-term`：博客文章和Issue映射

Utterances使用以下几种规则建立博客文章和GitHub Issues的映射：

- **Issue标题包含页面路径名（`issue-term="pathname"`）**  
Utterances将查询issue标题是否包含博客文章的路径名（pathname）。如果未找到匹配的issue，则当有人首次对你的博客文章发表评论时，Utterances会自动创建一个以此文章路径名为标题的issue。
- **Issue标题包含页面URL（`issue-term="url"`）**  
Utterances将查询issue标题是否包含博客文章的`URL`。如果未找到匹配的issue，则当有人首次对你的博客文章发表评论时，Utterances会自动创建一个以此文章`URL`为标题的issue。
- **Issue标题包含页面标题（`issue-term="title"`）**  
Utterances将查询issue标题是否包含博客文章的标题。如果未找到匹配的issue，则当有人首次对你的博客文章发表评论时，Utterances会自动创建一个以此文章标题为标题的issue。
- **Issue标题包含页面og:title（`issue-term="og:title"`）**  
Utterances将查询issue标题是否包含博客文章的[Open Graph](https://ogp.me)标题（og:title）。如果未找到匹配的issue，则当有人首次对你的博客文章发表评论时，Utterances会自动创建一个以此文章`og:title`为标题的issue。
- **特定的issue编号（`issue-number="具体数字"`）**  
Utterances按issue编号加载特定的issue。Utterances不会自动创建issue。
- **Issue标题包含特定项（`issue-term="你设置的特定内容"`）**  
Utterances将查询issue标题是否包含你设置的特定项。如果未找到匹配的issue，则当有人首次对你的博客文章发表评论时，Utterances会自动创建一个以你设置的特定项为标题的issue。

### `label`：Issue标签

如果你使用原有的仓库，但是担心Issues页面评论和问题混杂在一起，Utterances支持设置标签（Label）来区分它们。设置`label="你的标签内容"`，Utterances将在创建issue时使用你设置的标签。

- 标签名区分大小写。
- 标签必须存在于你的仓库中（须提前在GitHub Issues页面创建好，不能使用不存在的标签）。
- 标签名支持Emoji。例如：

    ```js
    label="💬"
    ```

### `theme`：主题

Utterances有多种主题，其中包括多款夜间模式主题。

- GitHub Light：`theme="github-light"`
- GitHub Dark：`theme="github-dark"`
- GitHub Dark Orange：`theme="github-dark-orange"`
- Icy Dark：`theme="icy-dark"`
- Dark Blue：`theme="dark-blue"`
- Photon Dark：`theme="photon-dark"`

你可以在文章末尾处的下拉框中选择主题查看效果，[点击此处跳转](#参考)。

## 自托管

自托管Utterances主要包含两个项目的部署：

- [Utterances](https://github.com/utterance/utterances)：前端静态网站，评论系统的界面显示，部署在GitHub Pages
- [utterances-oauth](https://github.com/utterance/utterances-oauth)：后端API，主要功能是授权，鉴权和创建Issues，部署在Cloudflare Workers

### 准备环境

- Debian、Ubuntu或是其他Linux发行版（教程里用的是Debian 10）
- GitHub账号(用于申请GitHub App及仓库)
- Cloudflare账号(用于申请Cloudflare Workers)
- 域名（可选，用于解决第三方Cookie的问题，详情见FAQ：[如何解决第三方Cookie的问题](#如何解决第三方cookie的问题)）

### 在Cloudflare Workers上托管utterances-oauth

#### 注册GitHub App

打开<https://github.com/settings/apps/new>注册自己的GitHub App，仅需要填写以下内容，其它默认:

| 项目 | 值 |
|--|--|
| GitHub App name | 你的博客的名字 |
| Description | 你的博客的描述 |
| Homepage URL | 你的博客的网址 |
| User authorization callback URL | 你的Cloudflare Worker子域以`/authorized`结尾。例如：`https://utterances_oauth.your_subdomain.workers.dev/authorized` |
| Webhook URL | 必填项，但是Utterances不使用此项。随意填，例如你的博客网址。 |
| Repository permissions | 仅`Issues: Read & Write`，不需要其它权限。 |
| Where can this GitHub App be installed | 仅此账号（Only on this account） |

注册成功后，系统会提示你生成私钥（Generate a private key），点击生成（这是必须的步骤，否则无法使用GitHub App）。但是Utterances不使用私钥，无需记住其值。

记下`Client ID`和`Client secret`的值，后面会用到它。

#### 构建utterances-oauth

1. 安装[Yarn](https://yarnpkg.com/lang/en/)和[Git](https://git-scm.com)

1. 克隆（`clone`）utterances-oauth

    ```sh
    git clone https://github.com/utterance/utterances-oauth
    cd utterances-oauth
    ```

1. 安装依赖

    ```sh
    yarn install
    ```

1. 配置环境变量

    在utterances-oauth根目录下新建一个名为`.env`的环境变量配置文件，配置如下变量：

    - **BOT_TOKEN**：创建GitHub Issues时将使用的GitHub个人访问令牌（Scopes仅需勾选`public_repo`，[点此生成](https://github.com/settings/tokens/new?scopes=public_repo)）。
    - **CLIENT_ID**：[GitHub OAuth web application flow](https://developer.github.com/v3/oauth/#web-application-flow)中要使用的`Client ID`，即注册GitHub App时记住的`Client ID`的值。
    - **CLIENT_SECRET**：OAuth web application flow中要使用的`Client secret`，即注册GitHub App时记住的`Client secret`的值。
    - **STATE_PASSWORD**：32位密码，用于加密request headers/cookies中的`state`，[点此生成](https://www.lastpass.com/password-generator)。
    - **ORIGINS**：来源域（origin）列表。多个来源域以逗号分隔，用于跨域资源共享Cross-Origin Resource Sharing（CORS）。

    示例如下（替换`*`号内容为你自己的值，ORIGINS网址改为你的博客网址）：

    ```properties
    BOT_TOKEN=****************************************
    CLIENT_ID=********************
    CLIENT_SECRET=****************************************
    STATE_PASSWORD=********************************
    ORIGINS=https://example.com,http://localhost:4000
    ```

1. 执行命令`yarn run build`构建utterances-oauth

#### 部署utterances-oauth

将生成的index.js文件复制到Cloudflare Workers，并配置好路由即可。

utterances-oauth还提供了直接部署的方法，需要你在`.env`文件中添加以下变量

- **CLOUDFLARE_EMAIL**：你注册Cloudflare时的邮箱
- **CLOUDFLARE_API_KEY**：部署Cloudflare Workers需要的API Token（API令牌模板（API tokens templates）选择`Edit Cloudflare Workers`，[点此生成](https://dash.cloudflare.com/profile/api-tokens)）
- **CLOUDFLARE_ZONE_ID**：你的Cloudflare Zone ID，在你的域名的预览页（Overview）
- **CLOUDFLARE_ACCOUNT_ID**：你的Cloudflare Account ID，在你的域名的预览页（Overview）
- **CLOUDFLARE_WORKERS_DEV_PROJECT**：你的Cloudflare Worker项目名，例如`utterances-oauth`。项目名必须满足以下要求:
  - 以字母开头
  - 以字母或数字结尾
  - 仅包含字母，数字，下划线和连字符
  - 不超过63个字符

```properties
CLOUDFLARE_EMAIL=****************
CLOUDFLARE_API_KEY=*************************************
CLOUDFLARE_ZONE_ID=********************************
CLOUDFLARE_ACCOUNT_ID=********************************
CLOUDFLARE_WORKERS_DEV_PROJECT=******
```

在utterances-oauth目录下的`package.json`文件中找到此行代码  
`"deploy": "cfworker deploy --name utteranc-es --route 'api.utteranc.es/*' src/index.ts"`  
修改`api.utteranc.es`为你的Cloudflare Workers域名，注意域名后的`/*`要保留，执行命令：

```sh
yarn run deploy
```

成功后用浏览器打开你的Cloudflare Workers域名，你会看到页面显示`alive`字样。

如果你想本地测试的话执行命令`yarn run start`。

### 在GitHub Pages上托管Utterances

这里仅介绍托管在GitHub上，其它平台类似。

1. 派生(fork)此仓库<https://github.com/utterance/utterances>

1. 克隆（clone）你刚刚派生的仓库(修改`username`为你的GitHub用户名)：`git clone https://github.com/username/utterances`

1. 从`package.json`文件中找到此行代码  
`"predeploy": "yarn run build && touch dist/.nojekyll && echo 'utteranc.es' > dist/CNAME",`  
修改为`"predeploy": "yarn run build && touch dist/.nojekyll",`

1. 修改`src/utterances-api.ts`文件中的网址为你的Cloudflare Workers网址

    ```ts
    export const UTTERANCES_API = 'https://utterances_oauth.your_subdomain.workers.dev';
    ```

1. 修改`src/index.html`文件中的`src`，`repo`，`issue-term`等配置项。

    ```js
    <if condition="NODE_ENV === 'production'">
      <script src="https://utteranc.es/client.js"
              repo="utterance/utterances"
              issue-term="homepage"
              crossorigin="anonymous"
              async>
      </script>
    </if>
    <else>
      <script src="http://localhost:4000/client.js"
              repo="jdanyow/utterances-demo"
              issue-term="pathname"
              crossorigin="anonymous"
              async>
      </script>
    </else>
    ```

    例如修改`client.js`为自己的链接：  
    `src="https://your_github_username.github.io/utterances/client.js"`

1. 执行`yarn run deploy`部署到GitHub Pages。

打开`https://your_github_username.github.io/utterances/`，登录后即可发表评论。

## FAQ

### 博主为什么选择 Utterances

之前一直想为博客添加评论系统，刚开始考虑的是国外的Disqus，因为在国内无法访问，所以查找过一些使用Disqus API的方法，如[DisqusJS](https://github.com/SukkaW/DisqusJS)和[Disqus PHP API](https://github.com/fooleap/disqus-php-api)，但是它们都需要后端程序，功能上也有一些局限性。之后偶然发现有人用Github Issues写博客，有人用Github Issues存储博客评论，如：[Gitment](https://github.com/imsun/gitment)、[Gitalk](https://github.com/gitalk/gitalk)以及本文介绍的[Utterances](https://github.com/utterance/utterances)都是基于Github Issues开发的评论系统。

目前Gitment已经很久没更新了，Gitalk和Utterances都在持续更新。其中Utterances使用[Primer](https://primer.style)设计语言，这是Github官方使用并开源的设计指南，所以Utterances有和Github Issues类似的漂亮样式。Utterances使用了更精细化的权限管理，这也是笔者最后选择Utterances的原因之一。

### Utterances的原理

Gitment、Gitalk和Utterances的原理都是基于GitHub Issues自带的评论功能，在访问博客时通过GitHub API查询对应博文下Issues的评论，显示在自己的网页上，评论时访客登录Github账号，授权应用，在对应GitHub Issues下发布评论。

### Utterances需要哪些权限

Utterances是一个GitHub App，仅需要Issues的读写权限。不同于OAuth Apps，GitHub App提供了精细化的权限管理，且GitHub App仅能作用于安装了它的仓库，[点此查看区别](https://developer.github.com/apps/differences-between-apps/#what-can-github-apps-and-oauth-apps-access)。

当你点击登录（Sign in to commnet）时，GitHub会有一个界面显示你需要授权GitHub App哪些权限。其中“确定你和GitHub App都可以访问的那些资源（Determine what resources both you and GitHub App can access）”这句可能写的让人很疑惑，这里“资源”指的是你GitHub账号权限和GitHub App权限的交集，即**Utterances只能在安装了它的仓库上读写Issues**。

### 如何解决第三方Cookie的问题

[Chrome浏览器自80版本开始默认设置`SameSite=Lax`](https://www.chromestatus.com/feature/5088147346030592)。越来越多的用户也开始打开“阻止第三方Cookie”的选项。

Utterances仅使用Cookie保存登录信息以避免重复登录的问题，如果你使用[Utterances GitHub App](https://github.com/apps/utterances)的话会存在Cookie跨域问题，这也是很多人选择自托管的原因之一。

要解决这个问题，你需要将博客和Utterances-outh部署在同一域名下。目前Chrome、Firefox、Microsoft Edge Chromium 把同一域名下的子域Cookie视为第一方Cookie。

假如你的博客网址是`https://blog.example.com`，那么Utterances-outh可以部署在`https://api.example.com`或是`https://api.utterances.example.com`下。

## 参考

- [Self hosting instructions](https://github.com/utterance/utterances/issues/42)
- [Cloudflare Workers Configure](https://developers.cloudflare.com/workers/quickstart#configure)
- [utterances requires too many permissions](https://github.com/utterance/utterances/issues/23)
- [Why does MicrosoftDocs require GitHub "Resources" permissions?](https://github.com/MicrosoftDocs/visualstudio-docs/issues/619#issuecomment-368942234)
- [Authenticating with GitHub Apps](https://developer.github.com/apps/building-github-apps/authenticating-with-github-apps/)
- [What can GitHub Apps and OAuth Apps access?](https://developer.github.com/apps/differences-between-apps/#what-can-github-apps-and-oauth-apps-access)

[点击此处](#theme主题)返回`theme：主题`章节。

{::nomarkdown}
<select id="theme" class="form-select" value="github-light" aria-label="Theme">
  <option value="github-light">GitHub Light</option>
  <option value="github-dark">GitHub Dark</option>
  <option value="github-dark-orange">GitHub Dark Orange</option>
  <option value="icy-dark">Icy Dark</option>
  <option value="dark-blue">Dark Blue</option>
  <option value="photon-dark">Photon Dark</option>
</select>
<script>
  var theme = document.querySelector("#theme");
  theme.addEventListener("change", function () {
    var message = {
      type: "set-theme",
      theme: theme.value
    };
    var utterances = document.querySelector('iframe');
    utterances.contentWindow.postMessage(message, new URL(utterances.src).origin);
  });
</script>
{:/nomarkdown}
