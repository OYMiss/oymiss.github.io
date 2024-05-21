---
author: Chongzhuo Yang
title: Tips 合集
pubDatetime: 2018-02-23 19:07:25
tags:
  - tips
  - misc
slug: "tips-tools"
description: 一些有用的命令和工具。
---

## Git 常用命令

### 取消 commit

```bash
git reset --hard HEAD~1
git push --force
```

### 配置 Git 代理

```bash
git config --global http.proxy 'socks5://127.0.0.1:1080'
git config --global https.proxy 'socks5://127.0.0.1:1080'
```

## 在 vscode 中配置 LaTex

<!--more-->

```json
  "latex-workshop.latex.recipes": [
    {
      "name": "xelatex",
      "tools": [
        "xelatex"
      ]
    },
  ],
  "latex-workshop.latex.tools": [
    {
      "name": "xelatex",
      "command": "xelatex",
      "args": [
        "-synctex=1",
        "-interaction=nonstopmode",
        "-file-line-error",
        "%DOC%"
      ]
    }
  ],
  "latex-workshop.view.pdf.viewer": "tab",
  "latex-workshop.latex.autoClean.run": "onBuilt",
```

测试文档

```tex
\documentclass{article}
% \documentclass[a4paper]{report}
% \usepackage[margin=2cm]{geometry}
% \renewcommand{\thesection}{\arabic{section}}

\usepackage{ctex}

\author{OYMiss}
\title{HelloWorld}
\begin{document}
\maketitle

\section{Test}
$$
\int^{\infty}_{0}{x^ne^{-x}} \mathrm{d}x = n!
$$

\end{document}

```

## macOS 工具

### 在 macOS 上读写 NTFS 硬盘

在 `/etc/fstab` 中添加下面内容，没有的话新建一个。

```
LABEL=硬盘名称 none ntfs rw,auto,nobrowse
```

重新加载，然后打开 `/Volumes` 就可以看到，也可以用下面命令创建一个快捷方式。

```bash
sudo ln -s /Volumes ~/Desktop/Volumes
```

### 在 macOS 上将 PNG 图标转为 ICNS 图标

macOS 上有些应用的图标太丑，如果想要更换 macOS 上应用的图标，需要有一个 ICNS 类型的图标，然后打开应用详情把图标放在原来的图标上就可以了。

下面命令将当前文件夹的 `pic.png` 转化为 `icon.icns`。

```bash
mkdir tmp.iconset
sips -z 16 16     pic.png --out tmp.iconset/icon_16x16.png
sips -z 32 32     pic.png --out tmp.iconset/icon_16x16@2x.png
sips -z 32 32     pic.png --out tmp.iconset/icon_32x32.png
sips -z 64 64     pic.png --out tmp.iconset/icon_32x32@2x.png
sips -z 128 128   pic.png --out tmp.iconset/icon_128x128.png
sips -z 256 256   pic.png --out tmp.iconset/icon_128x128@2x.png
sips -z 256 256   pic.png --out tmp.iconset/icon_256x256.png
sips -z 512 512   pic.png --out tmp.iconset/icon_256x256@2x.png
sips -z 512 512   pic.png --out tmp.iconset/icon_512x512.png
sips -z 1024 1024   pic.png --out tmp.iconset/icon_512x512@2x.png
iconutil -c icns tmp.iconset -o icon.icns
rm -rf tmp.iconset
```
