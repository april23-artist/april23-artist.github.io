---
title: Kubernetes Cluster 安裝 Network File System
slug: k8s-install-nfs
date: "2024-12-27"
categories:
  - "Kubernetes"
tags:
  - "NFS"
  - "Helm"
weight: 1
---

## 前言

NFS (Network File System) 是一種共享文件系統，它允許不同的主機或容器之間共享文件或資料夾。 </br>
NFS 是一種網路檔案系統協定，可以讓多個容器和 Pod 在不同的節點上訪問同一份數據，這對於需要跨多個 Pod 或節點持久儲存數據的應用非常有用。

## 步驟

每個 Node 都需要安裝。

### 安裝 NFS 客戶端工具 (nfs-common)

```bash
# 更新包列表
sudo apt update
# 安裝 NFS 客戶端工具
sudo apt install nfs-common
```

### 安裝 NFS 伺服器工具 (nfs-kernel-server)

```bash
sudo apt install nfs-kernel-server
```

### 啟動 NFS 伺服器

```bash
sudo systemctl start nfs-kernel-server
# 檢查 NFS 伺服器狀態
sudo systemctl status nfs-kernel-server
# 檢查 NFS 客戶端，使用 showmount 命令查看 NFS 伺服器共享的目錄
showmount -e <nfs_server_ip>
```

### 選擇適合共享 NFS 目錄的 Node (適合資料存儲的配置的 Node)

進入 NFS Node，這邊選擇 /home/nfs/rw 作為共享目錄。

```bash
cd /home
sudo mkdir nfs
cd nfs/
sudo mkdir rw
```

設置共享目錄。

```bash
# 加上共享設定
# /home/nfs/rw <nfs_server_ip>.0.0/16(rw,sync,no_subtree_check,no_root_squash)
sudo nano /etc/exports
```

重新加載。

```bash
exportfs -f
sudo systemctl reload nfs-server
```

### (補充) 將 NFS Node 共享目錄掛載到其他 Node 的指定目錄裡

進入其他 Node，選擇要同步共享目錄的資料夾。

```bash
sudo mkdir -p /mnt/nfs/rw
sudo mount -t nfs <nfs_server_ip>:/home/nfs/rw /mnt/nfs/rw
# 取消掛載
sudo umount /mnt/nfs/rw
```
