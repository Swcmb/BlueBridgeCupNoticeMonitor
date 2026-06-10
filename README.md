# 🏆 蓝桥杯通知监控系统

[![GitHub Stars](https://img.shields.io/github/stars/W1ndys/BlueBridgeCupNoticeMonitor?style=for-the-badge)](https://github.com/W1ndys/BlueBridgeCupNoticeMonitor/stargazers)
[![Python Version](https://img.shields.io/badge/Python-3.8+-blue?style=for-the-badge&logo=python)](https://www.python.org/)
[![License](https://img.shields.io/github/license/W1ndys/BlueBridgeCupNoticeMonitor?style=for-the-badge)](LICENSE)

> 实时监控蓝桥杯大赛官方通知，通过钉钉或飞书机器人即时推送的自动化工具

---

## ✨ 功能特性

- 🕵️‍♂️ **定时监控** — 自动扫描蓝桥杯官网最新通知列表
- 🔔 **多渠道推送** — 支持钉钉机器人 + 飞书机器人双通道通知
- 💾 **状态去重** — 本地存储已通知记录，避免重复推送
- 🧪 **自测模式** — 提供通知通道的测试验证功能
- ☁️ **GitHub Actions** — 内置工作流，无需自备服务器即可运行
- 🏗 **面向对象设计** — 代码结构清晰，易于扩展和维护

---

## 📁 项目结构

```
.
├── main.py                  # 主程序（监控 + 推送核心逻辑）
├── requirements.txt         # Python 依赖
├── lanqiao_data.json        # 已通知记录的本地缓存（自动生成）
├── .github/
│   └── workflows/
│       └── monitor.yml      # GitHub Actions 定时任务配置
└── README.md
```

---

## 🛠 快速开始

### 环境准备

```bash
# 克隆仓库
git clone https://github.com/W1ndys/BlueBridgeCupNoticeMonitor.git
cd BlueBridgeCupNoticeMonitor

# 安装依赖
pip install -r requirements.txt
```

### 配置环境变量

本项目通过**环境变量**读取配置，无需创建 `config.py`。在运行前请设置以下变量：

| 变量名 | 是否必需 | 说明 |
| --- | --- | --- |
| `DINGTALK_TOKEN` | 钉钉启用时必需 | 钉钉机器人 Webhook 的 `access_token` |
| `DINGTALK_SECRET` | 钉钉启用时必需 | 钉钉机器人的加签密钥（加签模式） |
| `FEISHU_BOT_URL` | 飞书启用时必需 | 飞书自定义机器人 Webhook 完整地址 |
| `FEISHU_BOT_SECRET` | 飞书启用时必需 | 飞书机器人的签名密钥 |
| `ENABLE_DINGTALK` | 可选 | 是否启用钉钉推送，默认 `true` |
| `ENABLE_FEISHU` | 可选 | 是否启用飞书推送，默认 `false` |

### 本地运行

```bash
# 示例：Linux / macOS
export DINGTALK_TOKEN="your_token_here"
export DINGTALK_SECRET="your_secret_here"
export ENABLE_DINGTALK="true"
export ENABLE_FEISHU="false"

python main.py
```

---

## ☁️ 部署方式一：GitHub Actions（推荐，免服务器）

仓库已内置工作流 `.github/workflows/monitor.yml`，默认 **每 10 分钟** 运行一次。

### 配置步骤

1. **Fork 本仓库**到你的 GitHub 账号

2. 在仓库 **Settings → Secrets and variables → Actions** 中添加以下 Secrets：

   | Name | Value |
   | --- | --- |
   | `DINGTALK_TOKEN` | 钉钉机器人 `access_token` |
   | `DINGTALK_SECRET` | 钉钉机器人加签密钥 |

3. 在 **Settings → Actions → General** 中，将 **Workflow permissions** 设置为 **Read and write permissions**，以便工作流可以提交并推送 `lanqiao_data.json`

4. 手动触发一次测试：进入 **Actions → 蓝桥杯通知监控 → Run workflow**

5. 完成！GitHub 会自动每 10 分钟检查一次，有新通知即推送

### 自定义运行频率

编辑 [monitor.yml](.github/workflows/monitor.yml) 中的 `cron` 表达式：

```yaml
on:
  schedule:
    - cron: '*/30 * * * *'  # 每 30 分钟一次
```

---

## 🖥 部署方式二：自建服务器（Crontab）

如果你希望在自己的服务器上运行：

```bash
# 编辑 crontab
crontab -e

# 添加以下内容（示例：每 30 分钟检查一次，日志追加到文件）
*/30 * * * * DINGTALK_TOKEN="your_token" DINGTALK_SECRET="your_secret" ENABLE_DINGTALK="true" ENABLE_FEISHU="false" /usr/bin/python3 /path/to/BlueBridgeCupNoticeMonitor/main.py >> /var/log/lanqiao_monitor.log 2>&1
```

---

## 🤖 钉钉机器人配置指南

1. 打开钉钉 → 目标群 → **群设置 → 机器人 → 添加机器人 → 自定义**
2. **安全设置**选择**加签**，复制密钥即为 `DINGTALK_SECRET`
3. 创建完成后，复制 Webhook 地址中 `access_token=` 后的部分作为 `DINGTALK_TOKEN`
4. 参考 [钉钉开放平台文档](https://developers.dingtalk.com/document/robots/custom-robot-access) 了解更多

---

## 🚀 飞书机器人配置指南

1. 在飞书群中添加 **自定义机器人**，选择签名校验
2. 将 Webhook 完整地址填入 `FEISHU_BOT_URL`
3. 将签名密钥填入 `FEISHU_BOT_SECRET`
4. 运行前设置环境变量 `ENABLE_FEISHU="true"`

参考 [飞书开放平台文档](https://open.feishu.cn/document/ukTMukTMukTM/ucTM5YjL3ETO24yNxkjN) 了解更多

---

## 🧪 测试通知通道

在 [main.py](main.py) 中，`LanQiaoMonitor` 类提供了两个测试方法：

```python
monitor.test_dingtalk_notification()  # 发送钉钉测试消息
monitor.test_feishu_notification()    # 发送飞书测试消息
```

可在 `__main__` 区块临时调用它们验证通道配置。

---

## 📌 注意事项

1. **网络可达性** — 运行环境需能访问蓝桥杯官网 API (`www.guoxinlanqiao.com`)
2. **签名校验** — 钉钉 / 飞书机器人必须开启**加签模式**，否则无法推送
3. **首次运行** — 首次运行时没有历史数据，会将当前所有通知视为新通知并推送；可通过删除 `lanqiao_data.json` 重置
4. **权限问题** — GitHub Actions 部署时需确保工作流有写权限，否则无法保存去重记录

---

## 🤝 参与贡献

欢迎提交 Issue 和 Pull Request！

1. Fork 本项目
2. 创建特性分支 (`git checkout -b feature/your-feature`)
3. 提交更改 (`git commit -am 'Add some feature'`)
4. 推送到分支 (`git push origin feature/your-feature`)
5. 创建 Pull Request

---

## 📄 开源协议

本项目采用 [GPL-3.0](LICENSE) 协议开源。

---

⭐ **如果这个项目对你有帮助，请点个 Star 支持一下！**
