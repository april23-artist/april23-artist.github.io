---
title: 使用 Openssl 生成證書
slug: openssl-create-certificate
date: "2025-01-14"
categories:
  - "Certificate"
  - "Kubernetes"
  - "ASP.Net"
tags:
  - "Openssl"
weight: 1
---

## 前言

我們將使用 OpenSSL 生成證書，並將其應用於 Kubernetes 中的服務間通信。這些證書可以用來保障 Kubernetes Cluster 中服務之間的安全通信。

## 步驟

### 使用 OpenSSL 生成證書

生成一個 RSA 私鑰。這個私鑰將用於簽名證書請求 (CSR) 並保護後續生成的證書。

```bash
openssl genpkey -algorithm RSA -out private.key -pkeyopt rsa_keygen_bits:2048
```

使用私鑰生成證書簽名請求 (CSR)，並提供基本的證書資料（如國家、州、省、市、組織等）。

```bash
# -new：創建新的 CSR
# -days 365：設置證書有效期為 365 天
# -nodes：生成沒有密碼保護的私鑰
# -keyout private.key：指定私鑰輸出的文件
# -out certificate.csr：指定 CSR 的輸出文件
# -subj 標誌後的資料應根據實際情況修改，例如國家、組織名稱等
openssl req -new -newkey rsa:2048 -days 365 -nodes -keyout private.key -out certificate.csr -subj "/C=TW/ST=ST/L=L Angeles/O=O/OU=OU/CN=CN"
```

使用私鑰對生成的 CSR 進行簽名，並生成自簽發的證書（.crt 文件）。

```bash
openssl x509 -req -in certificate.csr -signkey private.key -out certificate.crt
```

將證書和私鑰轉換成 PFX 格式

```bash
# -export：表示將私鑰和證書導出為 PFX 格式
# -password pass:：此處未指定密碼 (如果需要，請填寫密碼)
openssl pkcs12 -export -out certificate.pfx -inkey private.key -in certificate.crt -password pass:
```

### 將 Kubernetes Secret 掛載到對應的 Deployment

將證書和私鑰添加到 Kubernetes Cluster 中。

```bash
sudo kubectl create secret generic identity-api-keys \
  --from-file=certificate.pfx=certificate.pfx

sudo kubectl create secret generic client-api-keys \
  --from-file=private.key=private.key
```

將 Secret 掛載到 Deployment，需要將創建的 Secret 掛載到對應的容器中的 /app/keys 目錄。</br>

Identity Server:

```yaml
spec:
  template:
    spec:
      containers:
      - volumeMounts:
        - name: keys
          mountPath: /app/keys
      volumes:
      - name: keys
        secret:
          secretName: identity-api-keys
```

Client Api:

```yaml
spec:
  template:
    spec:
      containers:
      - volumeMounts:
        - name: keys
          mountPath: /app/keys
      volumes:
      - name: keys
        secret:
          secretName: client-api-keys
```

### 在 ASP.NET Core 中設定證書與私鑰

在 Identity Server 中，需要加載 certificate.pfx 來進行簽名和驗證。

```csharp
using Duende.IdentityServer.Services;

var certificatePath = Path.Combine("/app/keys", "certificate.pfx");
if (!File.Exists(certificatePath))
{
    throw new InvalidOperationException("Certificate file not found.");
}

builder.Services.AddIdentityServer(options =>
{
    // IdentityServer 的設置
})
.AddSigningCredential(new X509Certificate2(certificatePath, "password")); // 如果需要密碼的話
```

在 Client API 中，需要將私鑰加載並設置為 JWT Bearer 驗證的簽名密鑰。

```csharp
services.AddAuthentication().AddJwtBearer(options =>
{
  var issuerSigningKeyPath = "/app/keys/private.key";

  // 1. 讀取私鑰
  var privateKey = File.ReadAllText(issuerSigningKeyPath);

  // 2. 創建 RSA 物件並從 PEM 格式的私鑰中讀取
  //    請注意 ImportFromPem 需要在 .NET 5 及以上版本中使用
  var rsa = new RSACryptoServiceProvider();
  rsa.ImportFromPem(privateKey.ToCharArray());

  options.TokenValidationParameters.IssuerSigningKey = new RsaSecurityKey(rsa);
  options.TokenValidationParameters.ValidateIssuerSigningKey = true;
});
```