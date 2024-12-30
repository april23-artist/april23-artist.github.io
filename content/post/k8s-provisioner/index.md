---
title: Kubernetes Provisioner 自動化管理持久化存儲資源
slug: k8s-provisioner
date: "2024-12-30"
categories:
  - "Kubernetes"
tags:
  - "Helm"
  - "NFS"  
  - "Provisioner"
weight: 1
---

## 前言

Provisioner 是 Kubernetes 中的一個組件，用於自動化管理持久化存儲資源。它的主要用途包括:

- 自動創建存儲資源: 在 K8es 中創建 PersistentVolumeClaim (PVC) 時，Provisioner 會自動創建對應的 PersistentVolume (PV)。
- 簡化存儲管理：通過使用 Provisioner，無需手動管理存儲資源，減少運維的複雜度。
- 動態配置存儲：Provisioner 支持動態配置存儲資源，根據應用需求自動調整存儲大小和配置。
- 多種存儲後端支持：不同的 Provisioner 可以支持不同的存儲後端，如 NFS、Ceph、AWS EBS 等，提供靈活的存儲選擇。

## 步驟

假設 NFS 服務共享目錄為 /home/nfs/rw/mssql 。

```bash
# 加上共享設定
# /home/nfs/rw/mssql <nfs_server_ip>.0.0/16(rw,sync,no_subtree_check,no_root_squash)
sudo nano /etc/exports
```

### 建立 Provisioner 並設定 StorageClass 的名稱

```bash
# <nfs_path>: 共享的目錄 /home/nfs/rw/mssql
sudo helm install <provisioner_name> nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --version 4.0.18 \
    --namespace nfs \
    --set nfs.server=<nfs_server_ip> \
    --set nfs.path=<nfs_path> \
    --set storageClass.name=<storageClass_name> \
    --kubeconfig /etc/rancher/k3s/k3s.yaml
```

### 在 Statefulset 裡設定使用 \<storageClass_name\>

```yaml
kind: StatefulSet
spec:
  template:
    spec:
      containers:
      - volumeMounts:
        - mountPath: /var/opt/mssql
  volumeClaimTemplates:
  - metadata:
      name: {{ .Release.Name }}
    spec:
      storageClassName: <storageClass_name>
      accessModes:
      - ReadWriteMany
      resources:
        requests:
          storage: 1Gi
```
