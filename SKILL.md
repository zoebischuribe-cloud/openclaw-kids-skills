---
name: openclaw-kids-daily-push
description: |
  Use this skill whenever the user mentions daily education push, kids learning content,
  automatic push to parent groups, WeChat Work bot, Feishu bot, DingTalk bot,
  Lark webhook, classical Chinese education cards, science education text,
  Windows scheduled tasks for education content, local image assets for kids,
  openclaw kids, morning/evening push schedule, or base64 image upload for bots.
  Also trigger when the user wants to set up, configure, troubleshoot, or modify
  an automated daily education push system for children via IM platforms.
---

# OpenClaw Kids Daily Education Push

Automate daily educational content pushes to parent groups via Feishu, WeChat Work, or DingTalk webhooks.

- **Morning (06:30)**: Science education text
- **Evening (21:00)**: Classical Chinese text with local images (WeChat Work base64 only)

## What This Skill Does

- Guides setup of automated daily education pushes for kids
- Configures Windows Scheduled Tasks for local execution
- Integrates with Feishu / WeChat Work / DingTalk webhooks
- Handles local image assets for classical Chinese education cards
- Troubleshoots common issues (encoding, line endings, webhook invalid, etc.)

## When to Use

- Setting up a new daily education push system
- Adding or switching IM platforms (Feishu, WeChat Work, DingTalk)
- Configuring Windows scheduled tasks
- Troubleshooting failed pushes
- Managing local image assets for classical Chinese cards
- Adjusting push schedules or content
- Changing the start date for sequential day counting

## Prerequisites

- Windows 10/11 machine
- Python 3.9+ installed and on PATH
- Webhook URL from at least one IM platform (Feishu / WeChat Work / DingTalk)
- Git clone of `https://github.com/zoebischuribe-cloud/openclaw-kids-skills`
- For evening classical Chinese push with images: local image files on disk (e.g., `D:\...`)

## File Structure

```
openclaw-kids-skills/
|-- scripts/
|   |-- local_push.py            # Entry point for all pushes
|   |-- setup_windows_task.ps1   # Registers Windows scheduled tasks
|   |-- push.py                  # Generic webhook text pusher
|-- daily-science-kids/
|   |-- scripts/
|   |   |-- generate.py          # Science content generator
|   |-- knowledge_base.yaml      # Science knowledge database
|-- daily-guguwen/
|   |-- scripts/
|   |   |-- generate.py          # Classical Chinese content generator
|   |   |-- indexer.py           # Builds index.json from audio/images
|   |-- index.json               # Index of all 282 classical Chinese entries
|-- .github/workflows/
|   |-- daily-push.yml           # GitHub Actions fallback (cloud, no images)
|-- .env.example                 # Template for webhook configuration
|-- setup_now.ps1                # One-click setup wrapper
|-- test_morning.cmd             # Manual test batch file
```

## Step-by-Step Setup

### 1. Clone the Repository

```powershell
git clone https://github.com/zoebischuribe-cloud/openclaw-kids-skills.git
cd openclaw-kids-skills
```

### 2. Install Python Dependencies

```powershell
pip install pyyaml requests
```

Only `pyyaml` and `requests` are required.

### 3. Configure Webhook

Get your webhook URL from your IM platform group bot settings (see Platform Guides below).

### 4. Set Up Windows Scheduled Tasks

Open **PowerShell as Administrator**, then run:

```powershell
.\setup_now.ps1
```

Or directly with parameters:

```powershell
.\scripts\setup_windows_task.ps1 `
    -WebhookUrl "https://open.feishu.cn/open-apis/bot/v2/hook/xxxx" `
    -WebhookType "feishu"
```

Supported `-WebhookType` values: `wechat`, `feishu`, `dingtalk`.

This creates two daily tasks:
- `OpenClawKids_MorningPush` at 06:30
- `OpenClawKids_EveningPush` at 21:00

### 5. Test a Manual Push

Morning science push:

```powershell
python scripts/local_push.py morning --webhook "YOUR_URL" --type feishu
```

Evening classical Chinese push:

```powershell
python scripts/local_push.py evening --webhook "YOUR_URL" --type feishu
```

If the URL is already embedded in the scheduled task, you can omit `--webhook` and `--type` for task execution.

### 6. Adjust Start Date (Optional)

By default, `evening` mode counts from `2026-05-15` as day 1. To change:

Edit `scripts/local_push.py` and modify the `--base-date` argument in `generate_guguwen()`,
or pass `--base-date YYYY-MM-DD` when running `daily-guguwen/scripts/generate.py` directly.

## Platform Guides

### WeChat Work (Recommended)

- Supports text + image pushes
- Evening classical Chinese push includes base64-encoded local images
- Images are read directly from local disk paths (no cloud upload needed)
- Webhook key required; get it from group robot settings
- Image size limit: 2MB per image (auto-compression built in)

### Feishu

- Text-only push via interactive card
- Does not support local image upload via simple webhook
- Use for morning science text pushes and evening text-only classical Chinese
- Verify webhook signature if enabled in bot settings

### DingTalk

- Text-only push via markdown
- Does not support local image upload via simple webhook
- Keyword or signature verification may be required in bot settings

## Command Reference

### local_push.py

```
python scripts/local_push.py {morning|evening} [--webhook URL] [--type {wechat|feishu|dingtalk}]
```

| Argument | Description |
|----------|-------------|
| `morning` | Push science education text |
| `evening` | Push classical Chinese text + images (WeChat Work only) |
| `--webhook` | Webhook URL (env var `WEBHOOK_URL` also works) |
| `--type` | Bot type: `wechat` / `feishu` / `dingtalk` (default: `wechat`) |

### setup_windows_task.ps1

```powershell
.\scripts\setup_windows_task.ps1 -WebhookUrl "URL" [-WebhookType "feishu"] [-PythonPath "C:\...\python.exe"]
```

| Parameter | Description |
|-----------|-------------|
| `-WebhookUrl` | **Required.** Bot webhook URL |
| `-WebhookType` | Optional. `wechat` (default), `feishu`, or `dingtalk` |
| `-PythonPath` | Optional. Full path to python.exe if not on PATH |

## Troubleshooting

### Webhook returns "token invalid"

- The webhook URL was likely broken across multiple lines during copy-paste.
- Copy the entire URL as one continuous string.
- Use the provided `test_morning.cmd` batch file to avoid manual copy-paste errors.

### Scheduled task runs but push fails

- The task needs the PC to be powered on (lock screen is fine).
- Check that the webhook URL is embedded in the task arguments:
  `Task Scheduler` → `OpenClawKids_MorningPush` → `Actions` → `Edit` → verify Arguments field.

### Images not sending

- Only **WeChat Work** supports direct local image upload.
- Feishu and DingTalk will skip images and send text only.
- Ensure local image paths in `index.json` point to existing files.

### PowerShell script fails with "MissingEndCurlyBrace"

- The `.ps1` file had LF line endings instead of CRLF.
- Run `setup_now.ps1` which includes the corrected version, or convert line endings manually.

### "missing WEBHOOK_URL" error

- In early versions, the scheduled task did not forward the webhook URL to the script.
- Re-run `setup_now.ps1` to recreate tasks with the fix.

### UTF-8 / Emoji display issues in terminal

- Windows PowerShell defaults to GBK encoding.
- The script itself is UTF-8 safe; terminal display issues do not affect actual push content.
- Run `chcp 65001` before manual tests if you want proper emoji display.

## Maintenance

- **Change webhook**: Re-run `setup_now.ps1` with the new URL.
- **Change schedule**: Edit tasks in `Task Scheduler` (taskschd.msc).
- **Disable temporarily**: Disable tasks in `Task Scheduler` without deleting them.
- **View history**: Check `daily-science-kids/scripts/.history.json` and `.history.json`.
