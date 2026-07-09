# gotchas.md — 审查易错点与进化记录

审查 Skill 时最容易犯的错误。每次发现新模式时追加到下方「进化记录区」。

---

## 规范认知

- **不要删除合法字段**：`when_to_use`、`allowed-tools`、`paths`、`hooks`、`metadata`、`arguments`、`model`、`effort` 都是官方/平台字段。不确定某个字段是否合法时，查 `~/.codebuddy/docs/` 中的 Agent Skills 规范，不要猜。
- **when_to_use 与 description 互补不重复**：description 写 1-2 句话核心触发条件，when_to_use 展开具体场景。两者不重复内容。
- **description 是触发条件，不是摘要**：错误："这是一个帮助生成周报的 Skill"。正确："当用户要求写周报、生成站报时触发"。
- **description 使用第三人称**：官方要求，因为 description 注入 system prompt。

## 触发判断

- **不要误触发"创建新 skill"**：用户说"帮我写一个 skill"时不应触发 skill-auditor，那是 skill-creator 的职责。
- **不要误触发通用代码审查**：用户说"review my code"时不应触发。
- **负例必须是"近误"**：用相邻领域的近误（如"review this code"对 skill-auditor）。"今天天气如何"什么都测不出。

## 评估偏差

- **不要因为短就扣分**：一个 60 行做透窄场景的 Skill 可能是完美的。token 效率高是优点。
- **不要奖励"全面感"**：文档写得多不等于好。真正的质量 = 只说"不说就不会对"的内容。
- **自评必须诚实**：审查 skill-auditor 自己时标准不能放松，不能用"这是我的设计"作为借口。

## 建议质量

- **改进 description 须给具体改写文本**："改进 description"不是一个建议；给出改写后的完整文本才是。
- **逐字引用原文**：让用户能 Cmd+F 直接定位到要改的内容。
- **引官方依据**：引用 Agent Skills 规范中的具体条款，而非"我认为"。

## 安全扫描

- **环境变量打印是最高频漏洞**：73.5% 的 Skill 存在此问题（学术研究数据），审查时优先检查。
- **description 与实际行为不符是最强恶意信号**：skillguard 方法论指出，意图不匹配是恶意 Skill 最可靠的特征。
- **代码混淆本身就是红旗**：Skills 应保持人类可读，任何 minified/obfuscated 代码都应拒绝。
- **工具组合的危险性**：Bash+WebFetch = 数据外泄通道，Write+WebFetch = 恶意下载，Bash+Write+无限制 = 完全自由。

## 自审陷阱

- **your description 也要过自己这关**：审查 skill-auditor 自己时，把自己当陌生 Skill 审一遍。你自己在 when_to_use 中写了那些触发短语，它们真的都能准确命中吗？你给别人提的"description 加近误排除"的建议，自己的 description 做到了吗？
- **token 预算双标**：给别人审计时说"只写不说就不会对的"，自己的 SKILL.md 是否也在教 Claude 已知知识？自审时最容易被自己写的"核心技术栈"列表糊弄过去——Claude 不需要知道用了什么框架，它只需要知道怎么做。

## 渐进披露反模式

- **body 与 reference 重复是最大浪费**：当 SKILL.md body 的安全检测模式与 references/regex-patterns.md 高度重复时，Agent 在触发后加载了双份相同内容，白白浪费 token 预算。安全的做法：body 仅写检查要点 + 引用（见 references/xxx.md），完整正则库留在 reference 中按需加载。
- **超长 few-shot 示例应提取**：当 few-shot 超过 60 行时，它占用的 token 大概率超过它带来的指令信号价值。提取到 references/examples.md，body 中保留精简版。
- **IN/OUT 范围边界几乎人人都忘**：这是审查了 9 个独立 auditor 实现后的最大发现——包括 skill-auditor 自己在内的几乎所有 Skill 都缺少显式的范围边界声明。但这对防止近误触发至关重要。

---

## 进化记录区

> 每次审查后发现新模式时，按以下格式追加：
> `YYYY-MM-DD | 审查 <skill-name> | 新增: <简述> | 状态: pending`

<!-- 进化记录将在使用中自动追加 -->
