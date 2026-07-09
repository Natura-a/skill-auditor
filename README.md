# skill-auditor v3.1

> **AI Agent Skill 的安全 + 质量审计层。**
>
> 你不会不用 `npm audit` 就直接 `npm install`，那为什么让 AI Agent 加载第三方 Skill 时不审计？

---

## 问题

AI Agent 是新的运行时。Skill、插件、自定义指令——无论平台怎么称呼——就是新的依赖。而目前，**没有审计**。

你安装的每个 Skill 都获得与你亲手写的代码同等的信任。它读取你的提示、访问你的工具、在你机器上运行脚本。一个恶意 Skill 就能泄露 API Key、窃取对话历史、向 Agent 工作流注入隐藏指令。

**skill-auditor 是应该在任何 Skill 接触你的 Agent 之前存在的审计步骤。**

---

## 核心功能

skill-auditor 通过 **五个阶段** 评估每个 Skill：

| 阶段 | 关注点 | 核心能力 |
|---|---|---|
| **0. 来源与信任** | 从哪里来？ | 作者声誉、Star 数、更新时效、信任分级 |
| **1. 意图验证** | 说的和做的一样吗？ | 交叉比对 description 与实际行为，发现未声明的能力 |
| **2. 工程化质量 (0-60)** | 能稳定工作吗？ | 10 维度质量评分（描述质量、Token 效率、位置架构、范围边界、结构信号、自由度、渐进披露、契约接口、范例、容错） |
| **3. 安全审查 (0-40)** | 安全吗？ | OWASP Agentic Top 10 + MITRE ATLAS + CWE 映射，12 项安全检查 |
| **4. 评分与进化** | 结论是什么？ | A-F 等级、SAFE/SUSPICIOUS/MALICIOUS 裁决、触发测试集、知识进化 |

综合 **0–100** 分 + 等级 + 裁决。

---

## v3.0 新特性（跨生态整合）

v3.0 整合了来自 **9 个独立 skill-auditor 实现** 的最佳实践：

| 来源仓库 | 核心贡献 |
|---|---|
| `LeriusLei/skill-auditor` | 来源核查 + 信任分级 + 红旗清单 |
| `maltose21/skill-auditor` | 自审模式 + gotchas/evolution 进化机制 + when_to_use 支持 |
| `awch-D/claude-skill-auditor` | 21+ YAML 规则库（prompt 注入、命令注入、权限滥用） |
| `skatiyar/skill-auditor` | 10 维度质量评分 + 位置架构 + Token 预算分析 |
| `xiaoshi-11111111/codex-skill-auditor` | 结构化审查清单 + 分级报告框架 |
| `wrsmith108/claude-skill-security-auditor` | TypeScript 安全审计模式 |
| `mtoby8326/skill-security-auditor` | A-F 等级制 + 6 维度加权评分 |
| `LLMSecurity/skillguard` | OWASP Agentic Top 10 + MITRE ATLAS 映射 + 混淆检测 |
| `Zgdfsgd/skill-security-auditor-skill` | 供应链审计（SBOM/依赖混淆/签名）+ CWE 映射 |

---

## 安全审查 — 12 项扫描

| # | 检查项 | OWASP | 检测内容 |
|---|---|---|---|
| 1 | **Prompt 注入** | AG01 | 隐藏指令、角色操纵、编码绕过、越狱模式 |
| 2 | **不安全工具使用** | AG02 | `eval/exec/subprocess`、`curl \| sh`、破坏性命令、提权 |
| 3 | **过度代理** | AG03 | 敏感目录访问、sudo、后台进程、禁用安全特性 |
| 4 | **数据外泄** | AG05 | 未声明出站 HTTP、凭证收割、DNS 外泄 |
| 5 | **供应链风险** | AG06 | 未锁定依赖、依赖混淆、非官方源、签名缺失 |
| 6 | **内存投毒** | AG08 | 写入 MEMORY.md、跨 Skill 修改、持久化注入 |
| 7 | **资源滥用** | AG10 | 无界循环、Fork Bomb、Token 浪费、无关计算 |
| 8 | **硬编码凭证** | — | OpenAI/AWS/GitHub/Slack Key、JWT、SSH 私钥、bcrypt |
| 9 | **环境变量泄露** | — | `print(os.environ['KEY'])` — 现实世界最高频漏洞（73.5% 发生率） |
| 10 | **代码混淆** | — | Base64 荷载、字符码构造、多层编码、minified 代码 |
| 11 | **危险工具组合** | — | Bash+WebFetch=外泄、Write+WebFetch=恶意下载 |
| 12 | **权限滥用** | — | 通配符工具、过多工具数(>10)、敏感 MCP 工具 |

---

## 工程化审查 — 10 维度质量门

| # | 维度 | 权重 | 评估内容 |
|---|---|---|---|
| 1 | **描述质量** | 7 | 是否说清 WHAT 和 WHEN？第三人称？近误排除？ |
| 2 | **Token 效率** | 7 | 只写 Claude 不知道的内容，< 500 行 |
| 3 | **位置架构** | 6 | 关键约束在前 20 行，利用首因/近因效应 |
| 4 | **范围边界** | 6 | 显式 IN/OUT 声明，防止相邻领域混淆 |
| 5 | **结构信号** | 6 | 一致标题、代码块模板、无文本墙(>15行) |
| 6 | **自由度标记** | 6 | 硬性约束与灵活指导清晰区分 |
| 7 | **渐进披露** | 6 | 三层加载（metadata → SKILL.md → references）合理利用 |
| 8 | **契约接口** | 6 | JSON Schema 输入、结构化输出、错误码 |
| 9 | **Few-shot 范例** | 5 | 完整调用链、错误场景覆盖 |
| 10 | **容错处理** | 5 | 超时阈值、重试策略、可解析错误码 |

---

## A-F 等级表

| 分数 | 等级 | 裁决 | 操作 |
|---|---|---|---|
| 90-100 | **A** | SAFE | 直接安装 |
| 75-89 | **B** | SAFE | 安装，关注改进项 |
| 60-74 | **C** | SUSPICIOUS | 审查后安装 |
| 40-59 | **D** | SUSPICIOUS | 修复重大问题后使用 |
| 0-39 | **F** | MALICIOUS | **拒绝安装** |

---

## 智能缓存 — 只为增量付费

skill-auditor 维护审计缓存（`audit_cache.json`），记录每个文件的指纹。再次审计时：

- **无变更** → 即时返回缓存评分，零额外开销
- **契约变更** → 全量重审
- **脚本变更** → 安全重扫
- **`force: true`** → 跳过缓存，全量执行

---

## 支持的输入方式

| 输入类型 | 加载方式 |
|---|---|
| 本地 Skill 名称 | 读取 `.codebuddy/skills/<name>/SKILL.md` |
| GitHub URL | 通过 raw.githubusercontent.com 获取 |
| 粘贴的 SKILL.md | 已在对话上下文中 |

---

## 输出模式

| 模式 | 适用场景 |
|---|---|
| `json`（默认） | 程序化消费、CI 流水线 |
| `markdown` | 人工阅读报告 |

---

## 自审

skill-auditor 可以审计自己。同样的 10+12 维度，同样的标准，无一例外。

### 最新自审结果（v3.1, 2026-07-09）

```
══════════════════════════════════════════════
  自审: skill-auditor v3.1
  裁决: SAFE  等级: A  评分: 91/100
  工程化: 52/60  安全: 39/40
══════════════════════════════════════════════
```

| 维度 | v3.0 | v3.1 | 改进 |
|---|---|---|---|
| 描述质量 | 5/7 | **6/7** | 添加近误排除 |
| Token 效率 | 3/7 | **5/7** | 673→481 行，消除 body-reference 重复 |
| 位置架构 | 5/6 | 5/6 | — |
| 范围边界 | 3/6 | **6/6** | 添加显式 IN/OUT 表 |
| 结构信号 | 5/6 | 5/6 | — |
| 自由度标记 | 4/6 | **5/6** | 添加 [硬约束]/[建议] 标签 |
| 渐进披露 | 4/6 | **6/6** | 提取 examples.md，refs 加 TOC |
| 契约接口 | 6/6 | 6/6 | — |
| Few-shot 范例 | 4/5 | **5/5** | 提取到 references/examples.md，新增对比示例 |
| 容错处理 | 5/5 | 5/5 | — |
| **安全 (12 项)** | **39/40** | **39/40** | 新增 allowed-tools 声明 |

---

## v3.1 变更日志

| 改进项 | 来源 | 描述 |
|---|---|---|
| Description 近误排除 | 自审 #1 | 添加 `不用于创建新 Skill / 通用代码审查 / MCP 配置` |
| IN/OUT 范围边界 | 自审 #2 | 新增表格：IN 5 项 + OUT 5 项 |
| Body 精简 | 自审 #3 | 673→481 行（安全正则→regex-patterns.md、示例→examples.md） |
| 消除重复 | 自审 #4 | 阶段 4 安全检测仅保留要点表 + 引用 |
| Reference TOC | 自审 #5 | owasp-agentic-top10.md + regex-patterns.md 加目录 |
| 约束标签 | 自审优化 #6 | 关键位置标注 [硬约束] / [建议] |
| allowed-tools | 自审优化 #7 | `[Read, Write, Bash, WebFetch, Glob, Grep]` |
| Few-shot 提取 | 自审优化 #8 | 108 行示例→references/examples.md，新增缓存命中示例 |

---

## 集成

skill-auditor 是零外部依赖的独立 Skill 定义文件（`SKILL.md`）。集成步骤：

1. 将 `SKILL.md` 复制到 Agent 的 skills 目录
2. 确保 Agent 可以在同目录读写 `audit_cache.json`
3. 安装第三方 Skill 前调用 `{ "skill_name": "目标技能" }`
4. （推荐）配置为每次 `git clone` 或 Skill 下载后自动触发

---

## 座右铭

> **"接口不变，结论不改；脚本有变，安全重审；凭证一现，立即叫停；意图不匹，拒绝安装。"**

skill-auditor 不猜测。它比对契约、扫描源文件、映射到行业标准（OWASP、MITRE ATLAS、CWE），每次都给出量化的、可重现的裁决。

**自信安装。审计为先。**
