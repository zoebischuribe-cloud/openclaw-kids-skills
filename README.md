# OpenClaw Kids — 每日儿童教育自动推送

每天早上 6:30 科普知识，晚上 9:00 小古文学习，自动推送到家长群。

支持 **企业微信（推荐，支持图片）** / **飞书** / **钉钉**。

---

## 功能

- **每日科普（06:30）**：按星期轮替学科（科技/物理/化学/自然/时政/英语/惊喜）
- **每日小古文（21:00）**：顺序播放 282 集经典，含原文、出处、学习任务
- **本地图片推送**：企业微信可直接发送本地图片（大卡 + 长文稿），无需上传到云端
- **Windows 计划任务**：开机后自动运行，锁屏不影响推送
- **顺序计数**：支持自定义起始日期，从今天开始算第 1 天

---

## 快速开始

### 1. 克隆仓库

```powershell
git clone https://github.com/zoebischuribe-cloud/openclaw-kids-skills.git
cd openclaw-kids-skills
```

### 2. 安装依赖

```powershell
pip install pyyaml requests
```

### 3. 配置群机器人

在飞书/企业微信/钉钉群里添加自定义机器人，复制 Webhook 地址。

### 4. 创建计划任务（管理员 PowerShell）

```powershell
.\scripts\setup_windows_task.ps1 `
    -WebhookUrl "你的Webhook地址" `
    -WebhookType "feishu"
```

`-WebhookType` 可选值：`wechat`（默认，支持图片）、`feishu`、`dingtalk`。

### 5. 手动测试

```powershell
# 早上科普
python scripts/local_push.py morning --webhook "你的地址" --type feishu

# 晚上小古文
python scripts/local_push.py evening --webhook "你的地址" --type feishu
```

---

## 文件结构

```
openclaw-kids-skills/
|-- scripts/
|   |-- local_push.py            # 推送入口（morning / evening）
|   |-- setup_windows_task.ps1   # 创建 Windows 计划任务
|   |-- push.py                  # 通用 Webhook 文字推送
|-- daily-science-kids/
|   |-- scripts/generate.py      # 科普内容生成器
|   |-- knowledge_base.yaml      # 科普知识库
|-- daily-guguwen/
|   |-- scripts/generate.py      # 小古文内容生成器
|   |-- scripts/indexer.py       # 从音频/图片建立索引
|   |-- index.json               # 282 集小古文索引
|-- .github/workflows/
|   |-- daily-push.yml           # GitHub Actions 备用方案（云端，无图片）
|-- docs/2026-05-15/             # 详细构建记录（私有）
|-- SKILL.md                     # Skill 文档（公开）
```

---

## 平台对比

| 平台 | 文字 | 本地图片 | 推荐度 |
|------|------|----------|--------|
| 企业微信 | 支持 | **支持 base64 直传** | 首选 |
| 飞书 | 支持 | 不支持 | 次选 |
| 钉钉 | 支持 | 不支持 | 备选 |

---

## 常见问题

**Q: 电脑关机了还会推送吗？**  
A: 不会。需要保持开机，锁屏可以。

**Q: 怎么换 Webhook 地址？**  
A: 重新运行 `setup_windows_task.ps1` 覆盖旧任务。

**Q: 怎么改推送时间？**  
A: 打开「任务计划程序」(taskschd.msc)，找到 OpenClawKids_* 任务修改触发器。

**Q: 今天是第 1 天怎么设置？**  
A: `scripts/local_push.py` 已默认传入 `--base-date 2026-05-15`，从今天起顺序播放。

---

## 详细文档

- [SKILL.md](SKILL.md) — 公开的 Skill 说明文档
- [docs/2026-05-15/从零到一构建记录.md](docs/2026-05-15/从零到一构建记录.md) — 完整的构建过程、踩坑记录、Debug 笔记
