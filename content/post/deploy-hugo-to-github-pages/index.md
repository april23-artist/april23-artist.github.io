---
title: 透過 GitHub Actions 自動化部署 Hugo 到 GitHub Pages
slug: deploy-hugo-to-github-pages
date: "2024-12-24"
categories:
  - "DevOps"
tags:
  - "Hugo"
  - "GitHub Actions"
  - "GitHub Pages"
weight: 1
---

## 前言

想紀錄開發筆記，決定透過 GitHub Actions 自動化部署 Hugo 到 GitHub Pages。

## 步驟

### Windows 安裝 Hugo

安裝 Chocolatey ，使用管理員身分開啟 Windows Terminal。

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

安裝 Hugo，並確認 Hugo 版本。

```powershell
choco install hugo
choco install hugo-extended
hugo version
```

### 建立 GitHub Repository

建立名稱為 \<user_name\>.github.io 的 Repo 作為 Hugo 的專案。 </br>
這裡使用的 hugo theme 為 [Stask](https://stack.jimmycai.com/)，直接使用 [hugo-theme-stack-stater](https://github.com/CaiJimmy/hugo-theme-stack-starter) 範例來修改。 </br>
將 hugo-theme-stack-stater 內的檔案放入 \<user_name\>.github.io。

```powershell
git clone https://github.com/CaiJimmy/hugo-theme-stack-starter.git hugo-theme-stack-starter
```

因為環境未安裝 Go 在執行 hugo build 時會出錯，所以要稍微調整。 </br>
範例專案沒有 theme，需下載 [hugo-theme-stack](https://github.com/CaiJimmy/hugo-theme-stack.git)。

```powershell
git clone https://github.com/CaiJimmy/hugo-theme-stack.git themes/hugo-theme-stack
```

將專案內 go.mod, go.sum 刪除。 </br>
並調整 config > \_default > module.toml。

```diff
- path = "github.com/CaiJimmy/hugo-theme-stack/v3"
+ path = "hugo-theme-stack"
```

建置並啟動網站，若成功啟動網站即可將專案 push 到 GitHub。

```powershell
hugo build
hugo server --disableFastRender
```

### 設定 GitHub Pages

新增分支 gh-pages。 </br>
Settings > (Code and automation) Pages > (Build and deployment) Branch 設定為分支 gh-pages/(root)。

### 設定 Workflow

在分支 main 將範例原本的 .github\workflows\deploy.yaml 內容替換，並 push 到 GitHub ，將會自動化部署到 gh-pages 分支。

```yaml
name: Deploy Hugo site to GitHub Pages

# 設定在 `push` 事件觸發時運行工作流。你可以根據需求修改觸發條件。
on:
  push:
    branches:
      - main # 當推送到 main 分支時觸發

# 定義工作流程的各個步驟
jobs:
  deploy:
    runs-on: ubuntu-latest # 使用最新版本的 Ubuntu 運行此工作流

    steps:
      # 1. Checkout repository (將代碼庫檢出到 runner)
      - name: Checkout code
        uses: actions/checkout@v3

      # 2. 設置 Hugo 環境
      - name: Set up Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: latest
          extended: true

      # 3. 安裝 Hugo 主題
      - name: Install Hugo theme
        run: |
          git clone https://github.com/CaiJimmy/hugo-theme-stack.git themes/hugo-theme-stack

      # 4. 建構 Hugo 網站
      - name: Build the site
        working-directory: ./
        run: hugo --minify --gc --cleanDestinationDir

      # 5. 部署到 GitHub Pages (gh-pages 分支)
      - name: Deploy to GitHub Pages
        uses: JamesIves/github-pages-deploy-action@v4
        with:
          branch: gh-pages # 部署到 gh-pages 分支
          folder: public # Hugo 網站的輸出目錄
          clean: true # 部署之前清理已有的文件
```

## 參考

- [Stack](https://stack.jimmycai.com/)
- [在 Windows 中安裝 Hugo](https://horace-yeh.github.io/article/202212/b1-to-install-hugo-on-windows/)
