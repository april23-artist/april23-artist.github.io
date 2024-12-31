---
title: Kubernetes Cluster 安裝 Helm
slug: k8s-install-helm
date: "2024-12-27"
categories:
  - "Kubernetes"
tags:
  - "Helm"
weight: 1
---

## 前言

Helm 可以將 Kubernetes 資源文件 (Deployment、Service、ConfigMap 等) 應用程式打包為 Chart。
可以輕鬆安裝和管理 Chart，並且能夠執行升級、回滾、卸載等操作。Helm 減少了手動編寫和管理 Kubernetes 資源的負擔。

## 步驟

### 安裝 Helm

下載 Helm 後目錄會有 helm-v3.16.3-linux-amd64.tar.gz 文件。

```bash
wget https://get.helm.sh/helm-v3.16.3-linux-amd64.tar.gz
```

解壓縮 helm-v3.16.3-linux-amd64.tar.gz 文件。

```bash
tar -zxvf helm-v3.16.3-linux-amd64.tar.gz
```

進入 linux-amd64 目錄，將 linux-amd64 目錄底下的 helm 複製到 /usr/local/bin/。

```bash
cd linux-amd64/
sudo cp helm /usr/local/bin/
```

檢查安裝是否完成。

```bash
helm version
```

### 使用 helm create \<chart-name\> 新增 chart

新增 chart 會建立一個 \<chart-name\> 的資料夾。 </br>

```bash
helm create <chart_name>

<chart_name>/
├── Chart.yaml   # Helm Chart 的元數據文件，包含了有關 Chart 的基本信息，例如名稱、版本、描述等。
├── charts/      # 用來存放其他 Chart 的依賴包。當你在 Chart.yaml 中指定了依賴關係時，這些依賴會被下載並放到這個目錄中。
├── templates/   # Kubernetes 資源的模板文件。
└── values.yaml  # 定義了 Helm Chart 中的默認值，它是 Chart 的配置文件。
```

### 常用 helm 指令

```bash
helm install <chart_name>            # 安裝
helm uninstall <chart_name>          # 卸載
helm delete <chart_name>             # 刪除
helm upgrade <chart_name>            # 升級
helm history <chart_name>            # 歷史紀錄
helm rollback <chart_name> <version> # 版本回滾
helm upgrade --install <chart_name>  # 安裝或升級

# -n <namespace> 命名空間配置
# --values <values>.yaml 該文件包含 Helm Chart 配置的默認值
# -f <overwrite-values>.yaml 讓你覆蓋默認的配置
helm upgrade --install <chart_name> -n <namespace> --values <values>.yaml -f <overwrite-values>.yaml
```

## 參考

- [Helm](https://helm.sh/)
