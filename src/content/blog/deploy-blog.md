---
title: "Blog 快速部署"
description: "本文主要介绍如何快速部署一个像本站的个人博客。"
date: 2024-08-07
category: ["Tutorial"]
tags: ["Astro"]
---

## 基本操作

如何快速部署一个像本站的个人博客：
1. 前置要求：Xnix 机器，我个人用的 Macbook
2. 根据 [Astro 官网] 提供的教程 [Install and set up Astro](https://docs.astro.build/en/install-and-setup/) 使用 `npm run dev` 能够在本地运行网站
3. 我个人使用的模版：[Astro-yi](https://github.com/cirry/astro-yi)，下载并替换所有文件
4. 创建 Github 仓库，保存代码，并且根据官方教程 [Deploy your Astro Site to GitHub Pages](https://docs.astro.build/en/guides/deploy/github/) 在 Github Pages 上部署
6. Commit 你的工作！

## 可选

### 图床

搭建基于 Cloudflare 和 PicGo 的（免费）图床系统，具体参考这篇文章：[从零开始搭建你的免费图床系统 （Cloudflare R2 + WebP Cloud + PicGo）](https://sspai.com/post/90170)

### 部署在自定义域名

1. 购买域名：我使用的是[阿里云域名服务](https://wanwang.aliyun.com/domain/tld?spm=5176.8048432.J_6007716750.9.1db669a3fSNiKa#.com.cn).
2. 添加 DNS 解析记录[^1] ![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/202408141143898.png)
    - **记录类型**选择 CNAME
    - **主机记录**建议填写 www
    - **记录值**填写 `<username>.github.io`
3. 返回 Github, 在 Repository 的 Setting 处点击 Pages, Build and deployment 选择 Github Actions, Custom Domain 填写域名
4. 进行 DNS 检测，稍等若干分钟
5. （备选）强制 HTTPS 连接

![](https://pub-f4fb14aad5ef4ee6a83bd71292941254.r2.dev/202408141151157.png)

## Q&A

可能遇到的问题：
- `npm` 网络问题：参考这篇文章修改代理 [npm查看源地址以及更换源地址](https://blog.csdn.net/weixin_37861326/article/details/107064092)
- Astro 官方提供的 Workflow 无法运行，报错：
    ```text
    Error: No pnpm version is specified.
    Please specify it by one of the following ways:
        - in the GitHub Action config with the key "version"
        - in the package.json with the key "packageManager"
    ```
  修改方式：在 workflow YAML 文件中添加：
  ```yaml {4-6}
  - name: Install, build, and upload your site
        uses: withastro/action@v2
        with:
          path: . # The root location of your Astro project inside the repository. (optional)
          node-version: 20 # The specific version of Node that should be used to build your site. Defaults to 20. (optional)
          package-manager: pnpm@latest # The Node package manager that should be used to install dependencies and build your site. Automatically detected based on your lockfile. (optional)
  ```

[^1]: https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#dns-records-for-your-custom-domain