---
title: Multipass 搭配 K3s 建立 Kubernetes Cluster
slug: multipass-k3s
date: "2024-12-25"
categories:
  - "Kubernetes"
tags:
  - "Multipass"
  - "K3s"
  - "Chocolatey"
weight: 1
---

## 前言

Multipass 是輕量化的虛擬機管理工具，選擇搭配 K3s 而不選擇 minikube 是因為 K3s 可以建立多個 Node，非常適合用來學習 Kubernetes 的知識。

## 步驟

### 安裝 Multipass

```powershell
choco install multipass
```

### 建立虛擬機 (Node)

建立 3 個 Node ，1 個 Master，2 個 Worker。

```powershell
multipass launch --name k8s-master --cpus 1 --memory 4G --disk 10G
multipass launch --name k8s-node1 --cpus 1 --memory 4G --disk 10G
multipass launch --name k8s-node2 --cpus 1 --memory 4G --disk 10G
```

常用的 Multipass 指令。

```powershell
multipass ls                 # 虛擬機列表
multipass start <node_name>  # 啟動虛擬機
multipass stop <node_name>   # 關閉虛擬機
multipass shell <node_name>  # 進入虛擬機
multipass delete <node_name> # 刪除虛擬機
multipass purge              # 清除已刪除虛擬機
# 將虛擬機檔案複製到本機
multipass transfer <node_name>:<file_path> .
```

### 在 Master Node 建立 K3s

進入 Master Node。

```powershell
multipass shell k8s-master
```

在 Master Node 安裝 K3s。

```bash
# 安裝 K3s
curl -sfL https://get.k3s.io | sh -
# 查看 K3s 配置文件
sudo cat /etc/rancher/k3s/k3s.yaml
# 安裝後即可使用 kubectl 指令
sudo kubectl get nodes
```

### 將 Worker Node 加入 Master 叢集

輸入 exit 指令離開 Master Node。

```powershell
# 取得 Token
$TOKEN = multipass exec k8s-master -- sudo cat /var/lib/rancher/k3s/server/node-token
# 取得 IP
$MASTER_IP = (multipass info k8s-master | Select-String "IPv4" | ForEach-Object { $_ -replace 'IPv4:\s*', '' }).Trim()
# 將 Token、IP 指定給 Worker Node。
For ($f = 1; $f -le 2; $f++) { multipass exec "k8s-node$f" -- bash -c "curl -sfL https://get.k3s.io | K3S_URL='https://$($MASTER_IP):6443' K3S_TOKEN='$TOKEN' sh -" }
```

### 建立 Master 與 Worker Node 的 SSH

進入 Master Node 產生 SSH 密鑰。

```bash
# 確認 Master Node 是否有 SSH 密鑰
cat ~/.ssh/id_rsa.pub
# 若沒有則產生 SSH 密鑰
ssh-keygen -t rsa -b 4096
# 取得 SSH 密鑰
cat ~/.ssh/id_rsa.pub
```

### Worker Node 調整 SSH 配置

每個 Worker Node 都要執行。

```powershell
multipass shell k8s-node1
```

檢查 SSH 服務是否開啟

```bash
systemctl status ssh
```

調整 SSH 配置。

```bash
# 進入設定檔，確保以下設置未被註解 (即沒有 #)。
# PubkeyAuthentication yes
# AuthorizedKeysFile .ssh/authorized_keys
sudo nano /etc/ssh/sshd_config
# 重新啟動 SSH 服務
sudo systemctl restart ssh
# 將 Master SSH 密鑰寫入授權金鑰
echo "Master SSH 密鑰" >> ~/.ssh/authorized_keys
```

在 Master Node 執行。

```bash
# 測試 SSH 是否有通
ssh <worker_node_name>
```

### 將 Master Node 的 k3s.yaml 配置複製到 Worker Node

進入 Master Node 複製 k3s.yaml 到 Worker Node。

```bash
# SSH 密鑰複製到 root 用戶
sudo cp ~/.ssh/id_rsa /root/.ssh/
sudo cp ~/.ssh/id_rsa.pub /root/.ssh/
# 設定權限
sudo chmod 600 /root/.ssh/id_rsa
sudo chmod 644 /root/.ssh/id_rsa.pub
# 複製 k3s.yaml 到 Worker Node
sudo scp /etc/rancher/k3s/k3s.yaml ubuntu@k8s-node1:/tmp/k3s.yaml
```

進入 Worker Node ，將 k3s.yaml 的 Server 改為 Master Node 的 IP。

```bash
# 檢查目錄是否存在
ls /etc/rancher/k3s/
# 建立 /etc/rancher/k3s/ 目錄
sudo mkdir -p /etc/rancher/k3s/
# 確保 /tmp/k3s.yaml 文件存在後移動文件
sudo mv /tmp/k3s.yaml /etc/rancher/k3s/k3s.yaml
# 修改 Server 變數
# server: https://<master_ip>:6443
sudo nano /etc/rancher/k3s/k3s.yaml
```

### 設定 Worker Node 的 KUBECONFIG

進入 Worker Node 添加 KUBECONFIG 環境變數。

```bash
# 添加環境變數
# export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
nano ~/.bash_profile
# 使變更生效
source ~/.bash_profile
```

### (補充) 若 Master Node IP 改變，則調整 Worker Node 的設定重新加入叢集

進入 Worker Node 編輯環境變數文件。

```bash
# 修改 K3S_URL 變數
# K3S_URL='https://<master_ip>:6443'
sudo nano /etc/systemd/system/k3s-agent.service.env
# 重新載入和重啟服務
sudo systemctl daemon-reload
sudo systemctl restart k3s-agent
```

## 參考

- [Multipass](https://canonical.com/multipass)
- [K3s](https://k3s.io/)
- [Kubernetes 1 小時入門](https://geekhour.net/2023/12/23/kubernetes/)
