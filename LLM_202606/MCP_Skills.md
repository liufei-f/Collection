# MCP & Skills 学习笔记
> Claude Code 工作原理 · 本地部署指南 · 常见问题解答

---

## 一、核心概念

### Claude Code 是什么？

Claude Code 是运行在终端里的**执行环境**，给 Claude 配了一台电脑。

没有 Claude Code 的 Claude（网页版）：
- 只能聊天，不能碰你的文件
- 不能运行代码

有了 Claude Code：
- 能读写你电脑上的文件
- 能在终端运行命令（bash）
- 能调用 MCP Server
- 能跨多步骤完成复杂任务

---

### Skills 是什么？

Skills 是存放在文件系统中的 Markdown 指令文件，告诉 Claude 如何完成特定任务。

- **本质**：一段文字说明，Claude 读完后「知道」要怎么做
- **形式**：SKILL.md 文件 + 可选的脚本/资源
- **作用**：告诉 Claude 怎么思考、用什么流程、遵循什么规范

**判断标准：**

> 问自己：Claude 不用任何工具，光靠读一段文字说明，能完成这件事吗？
> - 能 → 适合做 Skill（例如：做 PPT 时用 Arial 字体）
> - 不能 → 需要 MCP（例如：读取本地 PDF 文件）

---

### MCP 是什么？

MCP（Model Context Protocol）是 Anthropic 制定的开放协议，让 Claude 能连接外部工具和服务。

- **本质**：真正运行在你电脑上的程序，有 CPU、内存、网络、文件系统访问权
- **形式**：一个 Node.js / Python 服务器（server.js）
- **作用**：给 Claude 手和眼睛，让它能真正执行操作

> MCP 有执行能力，是因为它就是你电脑上的一个程序——**Claude 的大脑租用了你电脑的手脚**。

---

## 二、三者的关系

### 一句话总结

> **Claude 是大脑，Claude Code 是身体，Skill 是经验，MCP 是工具。**

### Skills vs MCP 对比

| 对比维度 | Skills | MCP |
|---------|--------|-----|
| 是什么 | 给 Claude 的文字指南 | 给 Claude 的工具和权限 |
| 形式 | Markdown 文件 | 运行中的程序 |
| 作用 | 告诉 Claude 怎么做 | 让 Claude 能做到 |
| 例子 | 做 PPT 用 Arial 字体 | 真正读取 PDF 文件 |
| 影响对象 | 影响 Claude 的决策 | 被 Claude 调用执行 |

### 完整工作流程

以「帮我做 ColocBoost PPT」为例：

```
你："帮我做 ColocBoost PPT"
        ↓
Claude Code（终端壳，连接一切）
        ↓
Claude（大脑）
  读 Skill → 「做PPT前要先查文献」
        ↓
  调用 MCP
  ├── search_pubmed("ColocBoost") → 返回最新论文
  ├── list_papers()               → 返回本地 PDF 列表
  └── read_pdf("colocboost.pdf")  → 返回 PDF 内容
        ↓
  综合所有信息，生成 PPT 代码
        ↓
Claude Code 执行代码，生成 .pptx 文件
        ↓
你拿到 PPT
```

---

## 三、MCP Server 技术细节

### server.js 的三个部分

#### 第一部分：启动 Server（固定样板）

```javascript
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server(
  { name: "research-assistant", version: "1.0.0" },
  { capabilities: { tools: {} } }
);
```

每个 MCP Server 都一样，只需改 `name`。`StdioServerTransport` 表示 Claude 通过标准输入输出和 server 通信，像两个程序之间传纸条。

#### 第二部分：注册工具（菜单）

给 Claude 一份工具菜单，告诉它有哪些工具、需要什么参数、什么时候用哪个工具。

```javascript
server.setRequestHandler(ListToolsRequestSchema, async () => ({
  tools: [{
    name: "read_pdf",            // 工具名，Claude 调用时用这个
    description: "读取PDF内容",  // Claude 根据此判断何时用
    inputSchema: {               // 需要什么参数
      properties: { filename: { type: "string" } },
      required: ["filename"]
    }
  }]
}));
```

#### 第三部分：实现逻辑（真正干活）

Claude 每次调用工具都进入这个函数，用 `if` 判断调用哪个工具，执行对应代码。

```javascript
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  if (name === "read_pdf") {
    const buffer = fs.readFileSync(filePath);  // 真正读文件
    const data = await pdfParse(buffer);       // 真正解析 PDF
    return { content: [{ type: "text", text: data.text }] };
  }

  if (name === "search_pubmed") {
    const response = await fetch("https://eutils.ncbi.nlm.nih.gov/...");  // 真正发网络请求
    // ...
  }
});
```

### 添加新工具只需两步

1. 在菜单里注册（`name` + `description` + `inputSchema`）
2. 在逻辑里写 `if (name === "工具名") { ... }`

> 本质上，你能用 JavaScript 写什么，MCP 就能给 Claude 提供什么能力。

---

## 四、部署步骤

### 部署 Skill

```bash
# 1. 创建 skill 目录和文件
mkdir -p ~/.claude/skills/<skill名>
nano ~/.claude/skills/<skill名>/SKILL.md

# 2. SKILL.md 必须有 YAML frontmatter
# ---
# name: my-skill
# description: 什么时候触发、做什么事
# ---

# 3. 在全局 CLAUDE.md 里引用
echo '@~/.claude/skills/<skill名>/SKILL.md' >> ~/.claude/CLAUDE.md

# 4. 验证
# 在 Claude Code 里输入：你有哪些 skill
```

### 部署 MCP Server

```bash
# 1. 创建项目，安装依赖
mkdir ~/my-mcp && cd ~/my-mcp
npm init -y
npm install @modelcontextprotocol/sdk
npm pkg set type=module

# 2. 写 server.js（注册工具 + 实现逻辑）

# 3. 测试能否启动（光标停住不动 = 成功）
node server.js

# 4. 注册到 Claude Code（v2.1+ 用此命令）
claude mcp add <名字> node /完整路径/server.js

# 5. 验证
claude mcp list           # 应看到 ✓ Connected
# 重启 claude，输入 /mcp 检查工具数量
```

### 关键注意事项

- Claude Code v2.1 用 `claude mcp add` 命令注册，**不是**写在 `settings.json` 里
- 配置写入的是 `~/.claude.json`（项目级），不是 `settings.json`
- `pdf-parse` 需要用 `createRequire` 导入，不支持 ES module 默认导入
- `CLAUDE.md` 里未引用的 skill 文件，Claude 完全不知道其存在

---

## 五、问答记录

**Q: 为什么要在 CLAUDE.md 里写 `@~/.claude/skills/presentation/SKILL.md`？**

Claude Code 不会自动扫描文件夹，只读取被告知的内容。`@` 是文件引用语法，等价于把 skill 文件内容插入到 CLAUDE.md 里。不写这行，skill 对 Claude 完全不存在，就像写代码不 `import` 模块一样。

---

**Q: 读 PDF、搜 PubMed 为什么不能做成 Skill？**

这些是需要真正执行的动作——必须打开文件、发网络请求。Skill 只是文字说明，Claude 看完还是没有能力执行。MCP 才是真正运行的程序，能做这些操作。

---

**Q: MCP 为什么具有执行任务的能力？**

因为 MCP Server 就是一段真正运行在你电脑上的程序，拥有文件系统、网络等访问权限。`fs.readFileSync()` 真正读文件，`fetch()` 真正发网络请求——这不是描述，是执行。

---

**Q: 「请先用 search_pubmed 搜索...」算是 Skill 吗？**

不算，这只是一条一次性指令。Skill 应该包含这类流程说明，这样以后只说「帮我做PPT」，Claude 就会自动按流程搜文献，不需要每次手动写。

---

**Q: Skills 是告诉 MCP 怎么做吗？**

不是。Skills 是告诉 **Claude（大脑）** 怎么思考和决策的。MCP 本身没有智能，是被动工具箱，等着被 Claude 调用。Skills 影响 Claude 的决策，MCP 被 Claude 调用执行。

---

**Q: server.js 里要写具体代码吗？**

必须写，而且必须是真正能执行的代码。没有代码，工具只是空壳，Claude 调用它什么都得不到。

- 注册了工具但没实现逻辑 → Claude 能调用但返回空
- 有逻辑但没注册 → Claude 不知道工具存在

---

## 六、你的实际配置

### 已部署的 Skill

- **路径**：`~/claude/.claude/skills/presentation/SKILL.md`
- **功能**：制作学术风格 PPT，Arial 字体，黑白灰配色
- **引用**：`~/.claude/CLAUDE.md` 中已配置

### 已部署的 MCP Server

- **路径**：`~/claude/mcp-research/server.js`
- **状态**：✓ Connected，3 个工具
- `list_papers()` — 列出本地文献库所有 PDF
- `read_pdf(filename)` — 读取指定 PDF 内容
- `search_pubmed(query)` — 搜索 PubMed 学术文献

### 推荐 Prompt 写法

在 `presentation` skill 里加入文献流程说明后，只需说：

```
帮我做一个关于 ColocBoost 的学术 PPT
```

Claude 会自动：搜 PubMed → 查本地 PDF → 读取内容 → 生成 PPT。
