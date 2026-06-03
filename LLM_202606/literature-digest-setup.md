# 文献自动追踪系统 · 搭建记录
> 搭建日期：2026-06-03  
> 功能：每两天自动搜索 PubMed，Claude 总结，发送到 Gmail

---

## 系统架构

```
macOS launchd（定时器）
    ↓ 每两天早上 8 点触发
digest.py（Python 脚本）
    ├── 搜索 PubMed API → 获取最新文献
    ├── 调用 claude -p → Claude Code 总结
    └── Gmail SMTP → 发送 HTML 邮件到收件箱
```

---

## 一、环境信息

| 项目 | 值 |
|------|-----|
| Python 路径 | `/usr/bin/python3` |
| Claude 路径 | `/Users/phoebel/.local/bin/claude` |
| 脚本位置 | `~/claude/literature-digest/digest.py` |
| 日志位置 | `~/claude/literature-digest/digest.log` |
| 报错日志 | `~/claude/literature-digest/digest.err` |
| plist 位置 | `~/Library/LaunchAgents/com.literature.digest.plist` |

---

## 二、搭建步骤

### 1. Gmail 应用专用密码

- 开启两步验证：[myaccount.google.com/security](https://myaccount.google.com/security)
- 创建应用密码：[myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords)
- App name：`literature-digest`
- 生成的密码填入脚本时去掉空格

### 2. 安装依赖

```bash
pip3 install requests --break-system-packages
```

### 3. 脚本配置

文件路径：`~/claude/literature-digest/digest.py`

顶部配置区：

```python
GMAIL_USER     = "phoebeljianggu@gmail.com"
GMAIL_PASSWORD = "iczoymokaiezrfdc"       # 应用专用密码
TO_EMAIL       = "phoebeljianggu@gmail.com"
CLAUDE_PATH    = "/Users/phoebel/.local/bin/claude"

QUERIES = [
    "GWAS colocalization",
    "large language model bioinformatics",
    "LLM genomics variant",
    "model context protocol biology",
    "foundation model single cell",
    "LLM protein structure",
]
DAYS_BACK   = 2
MAX_RESULTS = 8
```

### 4. 注册定时任务

```bash
launchctl load ~/Library/LaunchAgents/com.literature.digest.plist
launchctl list | grep literature
# 输出：-  0  com.literature.digest  ← 注册成功
```

---

## 三、踩过的坑

### 坑 1：Claude 总结失败
**原因**：`~/.zshrc` 里有一个失效的 `ANTHROPIC_API_KEY` 环境变量，导致 Claude 尝试走 API 而不是订阅。

```bash
# 排查
grep "ANTHROPIC_API_KEY" ~/.zshrc
# 找到：# export ANTHROPIC_API_KEY=sk-a

# 临时解决
unset ANTHROPIC_API_KEY

# 永久解决：打开 ~/.zshrc 删掉那一行
```

**教训**：launchd 后台任务会继承 shell 环境变量，所以 `.zshrc` 里的失效 key 会干扰脚本。plist 里的 `EnvironmentVariables.PATH` 也必须包含 claude 所在目录。

### 坑 2：settings.json 配置 MCP 无效
**原因**：Claude Code v2.1 不再从 `settings.json` 读取 `mcpServers`，需要用命令注册：

```bash
claude mcp add <名字> node /路径/server.js
# 配置写入 ~/.claude.json，不是 settings.json
```

### 坑 3：pdf-parse ES module 报错
**原因**：`pdf-parse` 不支持 ES module 默认导入。

```javascript
// ❌ 错误
import pdfParse from "pdf-parse";

// ✅ 正确
import { createRequire } from "module";
const require = createRequire(import.meta.url);
const pdfParse = require("pdf-parse");
```

---

## 四、搜索关键词

| 关键词 | 覆盖方向 |
|--------|---------|
| `GWAS colocalization` | 核心研究方向 |
| `large language model bioinformatics` | LLM 生物信息应用 |
| `LLM genomics variant` | LLM + 基因组变异 |
| `model context protocol biology` | MCP 生物学应用 |
| `foundation model single cell` | 单细胞基础模型 |
| `LLM genomics application` | LLM 应用 |

---

## 五、常用管理命令

```bash
# 手动触发
launchctl start com.literature.digest

# 查看日志
cat ~/claude/literature-digest/digest.log

# 查看报错
cat ~/claude/literature-digest/digest.err

# 停止任务
launchctl unload ~/Library/LaunchAgents/com.literature.digest.plist

# 修改配置后重新加载
launchctl unload ~/Library/LaunchAgents/com.literature.digest.plist
launchctl load ~/Library/LaunchAgents/com.literature.digest.plist
```

---

## 六、后续可以扩展的方向

- [ ] 增加 bioRxiv 预印本搜索
- [ ] 按相关性评分给文献排序
- [ ] 添加关键词高亮到邮件里
- [ ] 支持发送给多个收件人
- [ ] 如果没有新文献则跳过发送
