
# 🚀 Automation AIO - SAP BTP 部署项目

[Deploy to SAP BTP](https://account.hanatrial.ondemand.com/)

这是一个专为 **SAP Business Technology Platform (BTP)** Cloud Foundry 环境构建的应用程序。

本项目已集成 **CI/CD 自动化部署流程**，利用 GitHub Actions 解决本地网络连接 SAP 服务器不稳定（如 `EOF` 错误、代理配置繁琐）的问题，实现代码推送即自动上线。

---

## 📋 目录

- [项目介绍](#-项目介绍)
- [前置准备](#-前置准备)
- [🚀 快速部署指南 (GitHub Actions)](#-快速部署指南-github-actions)
- [⚙️ 配置详解](#-配置详解)
- [🔧 本地开发与调试](#-本地开发与调试)
- [❓ 常见问题排查](#-常见问题排查)
- [👥 开发者信息](#-开发者信息)

---

## 📖 项目介绍

**Automation AIO** 旨在提供一种稳定、高效的方式将应用托管至 SAP 云平台。通过配置好的自动化工作流，开发者无需在本地安装复杂的 CF CLI 工具或处理代理设置，只需提交代码，GitHub 云端服务器将自动完成构建与发布。

**核心特性：**
*   **自动化部署**：Push 代码到 `main` 分支自动触发部署。
*   **网络无忧**：利用 GitHub 海外节点直连 SAP 数据中心，避开本地防火墙干扰。
*   **安全可靠**：敏感凭据通过 GitHub Secrets 加密存储。

---

## 🛠 前置准备

在开始之前，请确保您拥有：

1.  **SAP BTP 账号**（支持 Trial 试用版或企业版）。
2.  **GitHub 账号**。
3.  已将本项目 Fork 或 Push 到您自己的 GitHub 仓库。

---

## 🚀 快速部署指南 (GitHub Actions)

这是最推荐的部署方式，**无需**在您电脑上进行任何网络配置。

### 第一步：配置安全凭据 (Secrets)

为了让 GitHub 有权限部署到您的 SAP 账户，需要配置环境变量。

1.  进入您的 GitHub 仓库页面。
2.  点击顶部导航栏的 **Settings (设置)**。
3.  在左侧菜单选择 **Secrets and variables** -> **Actions**。
4.  点击 **New repository secret**，依次添加以下 5 个变量：

| 变量名 (Name) | 值 (Value) 示例 | 说明 |
| :--- | :--- | :--- |
| `CF_API` | `https://api.cf.ap21.hana.ondemand.com` | API Endpoint |
| `CF_ORG` | `971b8fc5trial_sg-fjdz0fqx` | Org Name |
| `CF_SPACE` | `dev` | 您的 Space (空间) 名称，通常是 dev |
| `CF_USERNAME` | `your_email@gmail.com` | 您的 SAP 登录邮箱 |
| `CF_PASSWORD` | `******` | **您的 SAP 登录密码** (注意：不是 SSO 验证码) |

### 第二步：检查部署配置

确保项目根目录下存在 `.github/workflows/sap-deploy.yml` 文件。如果没有，请创建该文件并填入以下内容：

<details>
<summary>点击展开查看 workflow 文件内容</summary>

```yaml
name: Deploy to SAP BTP

on:
  push:
    branches: [ "main" ]
  workflow_dispatch:

jobs:
  deploy-to-sap:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Code
        uses: actions/checkout@v4

      - name: Install Cloud Foundry CLI
        run: |
          wget -q -O - https://packages.cloudfoundry.org/debian/cli.cloudfoundry.org.key | sudo apt-key add -
          echo "deb https://packages.cloudfoundry.org/debian stable main" | sudo tee /etc/apt/sources.list.d/cloudfoundry-cli.list
          sudo apt-get update
          sudo apt-get install cf8-cli

      - name: Login to SAP BTP
        env:
          CF_API: ${{ secrets.CF_API }}
          CF_USER: ${{ secrets.CF_USERNAME }}
          CF_PASS: ${{ secrets.CF_PASSWORD }}
          CF_ORG: ${{ secrets.CF_ORG }}
          CF_SPACE: ${{ secrets.CF_SPACE }}
        run: |
          cf api "$CF_API"
          cf auth "$CF_USER" "$CF_PASS"
          cf target -o "$CF_ORG" -s "$CF_SPACE"

      - name: Push App
        run: cf push
```
</details>

### 第三步：触发部署

1.  修改代码或 `manifest.yml` 文件。
2.  执行 `git add .`, `git commit`, `git push` 推送到 GitHub。
3.  点击 GitHub 仓库页面的 **Actions** 标签，您将看到名为 **Deploy to SAP BTP** 的任务开始运行。
4.  等待图标变为绿色 ✅，即表示部署成功！

---

## ⚙️ 配置详解

### manifest.yml

这是 SAP Cloud Foundry 的核心配置文件，控制应用的名称、内存和构建方式。

**示例配置：**

```yaml
---
applications:
  - name: automation-aio          # 应用在 SAP 里的名称
    memory: 1024M                 # 分配内存
    disk_quota: 1024M             # 磁盘空间
    random-route: true            # 自动生成随机访问网址
    # buildpacks:                 # 如果是代码部署，指定构建包
    #   - python_buildpack
    # docker:                     # 如果是 Docker 部署，指定镜像
    #   image: user/repo:tag
```

---

## 🔧 本地开发与调试

如果您需要在本地运行代码进行测试：

1.  **安装依赖**：
    ```bash
    # Python 示例
    pip install -r requirements.txt
    
    # Node.js 示例
    npm install
    ```
2.  **启动应用**：
    ```bash
    python app.py
    # 或
    npm start
    ```

> **注意**：本地直接使用 `cf push` 可能会遇到网络代理问题（EOF Error）。如果在本地部署遇到困难，请优先使用上述的 **GitHub Actions** 方案。

---

## ❓ 常见问题排查

**Q: GitHub Action 报错 "Invalid credentials"？**
*   **A**: 请检查 `CF_PASSWORD` 是否正确。如果您开启了双重验证 (2FA)，对于 Trial 账号，通常仍直接使用主密码。如果是企业 SSO 账号，自动化部署可能需要使用 Client ID/Secret 方式登录（需额外配置）。

**Q: 部署成功但应用无法启动 (Crashed)？**
*   **A**: 在 SAP BTP 控制台查看日志，或在本地使用 CLI 查看：
    ```bash
    cf logs automation-aio --recent
    ```
    常见原因包括：端口未监听 `$PORT` 环境变量、内存不足或依赖缺失。

**Q: 提示 Quota Exceeded (配额不足)？**
*   **A**: SAP 免费试用账号有内存限制（通常总共 4GB）。请先停止或删除旧的应用，腾出空间。

---

## 👥 开发者信息

*   **维护者**: https://github.com/workerspages
*   **联系方式**: https://github.com/workerspages
*   **版权所有**: © 2024-2026

---
*Happy Coding! 如果觉得项目有帮助，请给个 Star ⭐️*
