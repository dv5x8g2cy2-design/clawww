---
name: podcast-listener-notes
description: 帮主人制作播客笔记的完整技能。触发场景：主人发来播客链接，说"做播客笔记"、"记一下这个播客"、"帮我记录"等。功能：抓取播客信息→生成详细笔记→询问听后感→整理录入→可选上传GitHub。主人称呼我为"在逃虾丸"，我称呼主人为"主人"。
---

# 🎧 播客笔记制作技能（听众版）

## 触发条件

主人发送播客链接，并表达"做播客笔记"、"帮我记录一下这个播客"、"记笔记"等意图时，激活本技能。

---

## 工作流程（完整版）

### Step 1: 抓取播客信息

使用 `web_fetch` 抓取播客页面内容：

```
URL: 主人发来的播客链接
maxChars: 8000
```

**重要：** 检查页面是否有：
- 节目介绍和章节时间戳
- **评论区听友的详细总结**（这是最宝贵的详细资料来源！）
- 听友整理的核心内容

> ⚠️ 注意：不要把"听友评论区"的内容当成"主播原话"。要区分清楚，只引用正文内容。

### Step 2: 生成详细笔记

根据抓取到的内容，生成结构化笔记到文件：

```
路径: podcast-notes/[播客名]_[期数]_[主题简称]_详细笔记.md
```

**笔记结构必须包含：**

```markdown
## 📋 基本信息
- 播客名
- 期数/标题
- 时长
- 主播/嘉宾
- 等...

## 🎯 核心主题

## 📖 完整内容梳理
（详细的章节/段落梳理，越详细越好）

## 🏗️ 核心观点结构
（树状图或分层列表）

## ✨ 金句整理
（摘录节目中的金句原话）

## 💭 听后感受（主人原创）
（留空，等主人提供后填入）
```

### Step 3: 回复主人摘要

发送格式：

```
🦐 收到主人！已生成 [播客名] 的详细笔记。

📋 基本信息：
- 标题：...
- 时长：...
- 主播：...

🎯 核心主题：...

📖 内容亮点：
...

✨ 金句：...

---
主人听完感受是什么？我帮你录入笔记 🎧
```

### Step 4: 录入听后感

主人提供听后感受后，将感受内容追加到笔记文件的 `## 💭 听后感受（主人原创）` 区域。

---

### Step 5: 询问是否上传 GitHub

笔记完成后，询问主人：

```
主人，这期笔记要上传到 GitHub 吗？
需要的话虾丸会：
1. 把笔记 .md 文件上传到 `podcast-notes/` 目录
2. 更新 GitHub Pages 的 index.html（包含所有播客卡片）
3. 先发给你预览，确认后再上传 🦐
```

---

### Step 6: 上传 GitHub（仅当主人确认后）

#### 6.1 上传笔记 .md 文件

使用 Python 脚本通过 GitHub REST API 上传：

```python
import urllib.request, json, base64, urllib.parse

token = "ghp_XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"
repo = "dv5x8g2cy2-design/clawww"

filepath = "podcast-notes/你的笔记文件名.md"
encoded_path = urllib.parse.quote(filepath)

# Get current SHA (if file exists)
get_url = f"https://api.github.com/repos/{repo}/contents/{encoded_path}"
req = urllib.request.Request(get_url, headers={"Authorization": f"token {token}"})
sha = None
try:
    with urllib.request.urlopen(req, timeout=10) as resp:
        sha = json.loads(resp.read())['sha']
except urllib.error.HTTPError as e:
    if e.code != 404:
        raise

# Read and upload
with open("笔记本地路径", "rb") as f:
    content = base64.b64encode(f.read()).decode()

payload = json.dumps({
    "message": "Add [播客名] 笔记",
    "content": content,
    "sha": sha
})

put_url = f"https://api.github.com/repos/{repo}/contents/{encoded_path}"
req = urllib.request.Request(put_url, data=payload.encode(), method="PUT")
req.add_header("Authorization", f"token {token}")
with urllib.request.urlopen(req, timeout=15) as resp:
    print("✅ Uploaded")
```

#### 6.2 更新 index.html（GitHub Pages）

GitHub Pages 根目录的 `index.html` 是播客笔记展示页，包含多张卡片。

**添加新卡片到 index.html 的步骤：**

1. 在 `</div>` 包裹的最后一个卡片后面、`</div>`（cards-container 的闭合标签）之前插入：

```html
<!-- [播客名] [期数] -->
<div class="card">
    <div class="card-header">
        <span class="card-icon">[emoji]</span>
        <div class="card-title-area">
            <h2 class="card-title">[播客名] [期数]</h2>
            <p class="card-meta">[核心主题] | [时长]</p>
        </div>
    </div>
    <div class="tags">
        <span class="tag">[标签1]</span>
        <span class="tag">[标签2]</span>
    </div>
    <div class="quote">
        "[核心金句]"
    </div>
    <div class="summary-section" id="summary-[N]">
        <div class="summary-header" onclick="toggleSummary('summary-[N]')">
            <span class="summary-label">播客总结</span>
            <div class="toggle-btn">▼</div>
        </div>
        <div class="summary-content">
            <p><strong>核心主题：</strong>...</p>
            <p><strong>主要内容：</strong></p>
            <p>① ...</p>
            <p>② ...</p>
            <p><strong>主人听后感：</strong>...</p>
        </div>
    </div>
    <div class="actions">
        <a href="[小宇宙episode ID链接]" target="_blank" class="btn btn-primary">🎧 收听播客</a>
        <a href="[GitHub笔记文件链接]" target="_blank" class="btn btn-secondary">📄 查看笔记</a>
    </div>
</div>
```

2. 上传 index.html 到 GitHub：

```python
# 先获取当前SHA
get_url = f"https://api.github.com/repos/{repo}/contents/index.html"
req = urllib.request.Request(get_url, headers={"Authorization": f"token {token}"})
with urllib.request.urlopen(req, timeout=10) as resp:
    sha = json.loads(resp.read())['sha']

# 上传
with open("index_new.html路径", "rb") as f:
    content = base64.b64encode(f.read()).decode()
payload = json.dumps({
    "message": "Update index.html with [播客名] card",
    "content": content,
    "sha": sha
})
# ... PUT request same as above
```

**GitHub 文稿链接的 URL 编码注意点：**
- 中文文件名需要 URL 编码
- 用 `urllib.parse.quote(filename)` 获取正确编码
- 例如：少楠 → `%E5%B0%91%E6%A5%A0`（不是 `%E5%B0%91%E6%A5%A6`）

---

## 准确性规则 ⚠️

1. **正文 vs 评论区**：只引用播客正文内容，不把听友评论当成主播原话
2. **无法核实的内容**：如果网页抓不到详细正文，要如实告诉主人，不能编造
3. **episode ID**：必须从用户发送的链接或会话记录中提取，不能编造 episode ID
4. **GitHub 链接编码**：中文文件名用 URL encode 检查，防止 404

---

## 输出文件位置

本地文件：
```
/home/openclaw/.openclaw/workspace-bot-q318/podcast-notes/
```

GitHub 仓库：
```
https://github.com/dv5x8g2cy2-design/clawww
```

GitHub Pages 网站：
```
https://dv5x8g2cy2-design.github.io/clawww/
```

---

## 记忆更新

每次完成一期笔记后，更新 memory/YYYY-MM-DD.md：

```markdown
## 🎧 播客笔记 - [播客名]

**时间：** YYYY-MM-DD HH:MM

### 完成情况
- ✅ 笔记已生成
- 路径：podcast-notes/...

### 基本信息
- 标题：...
- 时长：约 N 分钟

### 听后感
- 待主人提供 / 已录入主人感受

### GitHub
- 笔记文件：已上传 / 待上传
- index.html：已更新 / 待更新
```

---

## 关键参数速查

| 项目 | 值 |
|------|-----|
| GitHub Token | **需要用户提供经典 PAT**（需有 `repo` scope） |
| GitHub 仓库 | `dv5x8g2cy2-design/clawww` |
| GitHub Pages | `https://dv5x8g2cy2-design.github.io/clawww/` |
| index.html 路径 | 仓库根目录 `/index.html` |
| 笔记目录 | `podcast-notes/` |
| 本地工作目录 | `/home/openclaw/.openclaw/workspace-bot-q318/podcast-notes/` |

> ⚠️ GitHub Token 不能写入 SKILL.md，会被 GitHub  secret scanning 拦截。上传时在代码里填入 token。