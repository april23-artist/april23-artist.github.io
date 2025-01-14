---
title: Jenkins 與 GitHub 連結
slug: jenkins-connect-github
date: "2025-01-14"
categories:
  - "DevOps"
tags:
  - "Jenkins"
weight: 1
---

## 前言

透過 GitHub Personal access tokens 讓 Jenkins 可以透過連結取得 Repository。

## 步驟

### 在 Jenkins 安裝 GitHub Plugins

進入 Plugins 安裝 GitHub。

<img src="1736827544563.jpg" alt="image" width="800"> </br>

<img src="1736827576773.jpg" alt="image" width="800"> </br>

### 新增 Pipeline 並設定 GitHub URL

新增 Pipeline 作業。

<img src="1736827742235.jpg" alt="image" width="800"> </br>

設定 Build Triggers，勾選 GitHub hook trigger for GITScm polling。

<img src="1736827769989.jpg" alt="image" width="800"> </br>

設定 Repository URL，連結格式為 https://\<personal_access_token\>@github.com/\<repository_name\>.git。</br>
\<personal_access_token\> 需在 GitHub 產生並取得。

<img src="1736827904717.jpg" alt="image" width="800"> </br>

或是新增 Credentials 將 Token 資訊儲存在 Jenkins 中。Username 為 GitHub 用戶名稱，Password 為 GitHub Personal access tokens。

<img src="1736841634587.jpg" alt="image" width="800"> </br>

<img src="1736841423821.jpg" alt="image" width="800"> </br>

指定 Repository 的分支。

<img src="1736827953356.jpg" alt="image" width="800"> </br>

### (補充) 產生新的 GitHub Personal access tokens

進入使用者選單下的 Settings > Developer settings。

<img src="1736828059382.jpg" alt="image" width="400"> </br>

產生 Personal access tokens。

<img src="1736828088274.jpg" alt="image" width="800"> </br>

設定 Personal access tokens 可使用的範圍。

<img src="1736828258534.jpg" alt="image" width="800"> </br>

## 參考

- [建立新的Jenkins任務並與Github連結](https://ithelp.ithome.com.tw/articles/10267686)
