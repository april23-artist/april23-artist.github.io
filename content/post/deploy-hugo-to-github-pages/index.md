---
title: 部署 Hugo 到 GitHub Pages
description: 透過 GitHub Actions 自動化部署 Hugo
slug: deploy-hugo-to-github-pages
date: 2024-12-24 00:00:00+0000
categories:
  - "Hugo"
tags:
  - "GitHub Actoins"
  - "Hugo"
weight: 1
---

## 前言

> 想將開發記錄下來，決定使用 GitHub Pages 提供的網頁來架設 Blog 網站。 </br>
> 此篇將透過 GitHub Actions 自動化部署 Hugo 到 GitHub Pages 。

## 步驟

> ### Windows 安裝 Hugo
>
> 安裝 Chocolatey 。 </br>
> 使用管理員身分開啟 Windows Terminal 。
>
> ```bash
> Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
> ```
>
> 安裝 Hugo 。
>
> ```bash
> choco install hugo
> choco install hugo-extended
> ```
>
> 確認 Hugo 版本。
>
> ```bash
> hugo version
> ```
>
> ### 建立 GitHub Repository
>
> 建立名稱為 \<username\>.github.io 的 Repo 作為 Hugo 的專案。 </br>
> 這裡使用的 hugo theme 為 [Stask](https://stack.jimmycai.com/) 所以直接使用 [hugo-theme-stack-stater](https://github.com/CaiJimmy/hugo-theme-stack-starter) 範例來修改。 </br>
> 將 hugo-theme-stack-stater 內的檔案放入 \<username\>.github.io 中。
>
> ```bash
> git clone https://github.com/CaiJimmy/hugo-theme-stack-starter.git hugo-theme-stack-starter
> ```
>
> 範例專案沒有 theme ，要再下載 [hugo-theme-stack](https://github.com/CaiJimmy/hugo-theme-stack.git)。
>
> ```bash
> git clone https://github.com/CaiJimmy/hugo-theme-stack.git themes/hugo-theme-stack
> ```
>
> 因為未安裝 Go 在執行 hugo build 時會出錯，所以將專案內 go.mod, go.sum 刪除。 </br>
> 並調整 config > \_default > module.toml 。
>
> ```diff
> - path = "github.com/CaiJimmy/hugo-theme-stack/v3"
> + path = "hugo-theme-stack"
> ```
>
> 執行 hugo build 建置網站。 </br>
> 執行 hugo server --disableFastRender 啟動網站。 </br>
> 成功啟動網站即可將專案 push 到 GitHub 。
>
> ### 設定 GitHub Pages
>
> 新增分支 gh-pages 。 </br>
> Settings > (Code and automation) Pages > (Build and deployment) Branch 設定為分支 gh-pages/(root) 。 </br>
> Settings > (Code and automation) Actions > General > Workflow permissions 設定為 Read and write permissions 。
>
> ### 設定 Workflow
>
> 在專案 (main) 新增 .github\workflows\deploy.yaml，並 push 到 GitHub ，將會自動化部署到 gh-pages 分支。
>
> ```yaml
> name: Deploy Hugo site to GitHub Pages
>
> # 設定在 `push` 事件觸發時運行工作流。你可以根據需求修改觸發條件。
> on:
>   push:
>     branches:
>       - main # 當推送到 main 分支時觸發
>
> # 定義工作流程的各個步驟
> jobs:
>   deploy:
>     runs-on: ubuntu-latest # 使用最新版本的 Ubuntu 運行此工作流
>
>     steps:
>       # 1. Checkout repository (將代碼庫檢出到 runner)
>       - name: Checkout code
>         uses: actions/checkout@v3
>
>       # 2. 設置 Hugo 環境
>       - name: Set up Hugo
>         uses: peaceiris/actions-hugo@v2
>         with:
>           hugo-version: latest
>           extended: true
>
>       # 3. 安裝 Hugo 主題
>       - name: Install Hugo theme
>         run: |
>           git clone https://github.com/CaiJimmy/hugo-theme-stack.git themes/hugo-theme-stack
>
>       # 4. 建構 Hugo 網站
>       - name: Build the site
>         working-directory: ./
>         run: hugo --minify --gc --cleanDestinationDir
>
>       # 5. 部署到 GitHub Pages (gh-pages 分支)
>       - name: Deploy to GitHub Pages
>         uses: JamesIves/github-pages-deploy-action@v4
>         with:
>           branch: gh-pages # 部署到 gh-pages 分支
>           folder: public # Hugo 網站的輸出目錄
>           clean: true # 部署之前清理已有的文件
>         env:
>           GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} # GitHub 自動生成的 token，用於授權
> ```

## 結語

> 做法或許不是那麼正規但也總算是把整個流程建立起來，再來網站的訊息、外觀的調整這裡就不多做說明。
>
> - 準備網站：使用 Hugo 構建網站並推送到 GitHub。 </br>
> - 設定 GitHub Actions：創建 .github/workflows/deploy.yml 文件，配置自動化部署工作流。 </br>
> - 啟動部署：每次 push 到 main 分支時，GitHub Actions 會自動構建並將網站部署到 gh-pages 分支。 </br>
> - 配置 GitHub Pages：設定 GitHub Pages 使用 gh-pages 分支來部署網站。

## 參考

> - [Stack](https://stack.jimmycai.com/)
> - [在 Windows 中安裝 Hugo](https://horace-yeh.github.io/article/202212/b1-to-install-hugo-on-windows/)
