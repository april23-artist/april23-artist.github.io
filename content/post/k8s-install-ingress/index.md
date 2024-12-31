---
title: Kubernetes Cluster 安裝 Ingress
slug: k8s-install-ingress
date: "2024-12-31"
categories:
  - "Kubernetes"
tags:
  - "Ingress"
  - "Nginx"
  - "Helm"
weight: 1
---

## 前言

Kubernetes Ingress 是一種用於管理外部訪問 Kubernetes 叢集中服務的方式。它提供了 HTTP 和 HTTPS 路由功能，允許外部用戶通過單一 IP 地址訪問多個服務。

主要功能:
- 路由：根據 HTTP/HTTPS 請求的 URL 路徑或主機名，將流量路由到不同的服務。
- 負載均衡：在多個後端服務實例之間分配流量，實現負載均衡。
- SSL/TLS 終止：處理 HTTPS 請求，提供 SSL/TLS 終止功能。
- 虛擬主機：支持基於主機名的虛擬主機配置，允許多個域名共享同一個 IP 地址。

Ingress 需要搭配 Ingress Controller 使用，是一個負責處理 Ingress 資源的控制器。常見的 Ingress Controller 有:
- Nginx
- Traefik

**Ingress 是一種抽象，nginx則是實作抽象**
- Ingress 用於定義如何將外部流量路由到 Kubernetes 叢集中的服務。
- Nginx 實作這個抽象的工具，作為 Ingress Controller 來處理和路由流量。

## 步驟

### 檢查 kube-system 命名空間中是否已經有預設的 Traefik Ingress Controller

因為預設的 Traefik 會占用預設的 80 和 443 端口，所以在安裝 Ingress Controller 之前應該先移除它。

```bash
# 檢查 kube-system 命名空間中是否有 Traefik
sudo kubectl get all -n kube-system | grep traefik

# 如果有 Traefik 移除它
sudo kubectl delete deployment traefik -n kube-system
sudo kubectl delete service traefik -n kube-system
```

### 透過 Helm 安裝 Ingress

下載 ingress-nginx。

```bash
sudo helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
sudo helm repo update
```

加上 ingress-nginx 命名空間，以及指定 Node 加上 Label，讓資源建立在上面。

```bash
sudo kubectl create ns ingress-nginx
sudo kubectl label node k8s-node1 ingress=true
```

安裝 ingress-nginx。

```bash
sudo helm install ingress-nginx ingress-nginx/ingress-nginx \
    --namespace ingress-nginx \
    --set kind=DaemonSet \
    --set nodeSelector."kubernetes\.io/os"=linux \
    --set nodeSelector.ingress=true \
    --kubeconfig /etc/rancher/k3s/k3s.yaml
```

### 範例

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress
spec:
  ingressClassName: "nginx"
  rules:
  - host: test.com # Domain 配置
    http:
      paths:
      - pathType: Prefix
        backend:
          service:
            name: test-svc # 代理到哪個 Service
            port: 
              number: 80
        path: /
```