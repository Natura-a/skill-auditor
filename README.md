# skill-auditor v3.1

> **The missing security + quality layer for AI agent skills.**
>
> You wouldn't run `npm install` without `npm audit`. Why would you let an AI agent load third-party skills without one?

---

## The Problem

AI agents are the new runtime. Skills, plugins, custom instructions — whatever your platform calls them — are the new dependencies. And right now, **there is no audit**.

Every skill you install gets the same trust as code you wrote yourself. It reads your prompts, accesses your tools, and runs scripts on your machine. One malicious skill can leak API keys, exfiltrate conversation history, or inject hidden commands into your agent's workflow.

**skill-auditor is the audit step that should exist before any skill touches your agent.**

---

## What It Does

skill-auditor evaluates every skill across **five phases**:

| Phase | Focus | Key Capabilities |
|---|---|---|
| **0. Source & Trust** | Where does it come from? | Author reputation, star count, update recency, trust tier classification |
| **1. Intent Verification** | Does it do what it says? | Cross-reference description vs actual behavior, detect undisclosed capabilities |
| **2. Engineering Quality (0-60)** | Can it work reliably? | 10-dimension quality score (description, token efficiency, positional architecture, scope, structure, degrees of freedom, progressive disclosure, contract, examples, error handling) |
| **3. Security Audit (0-40)** | Is it safe? | OWASP Agentic Top 10 + MITRE ATLAS + CWE mapping, 14 security sub-checks |
| **4. Scoring & Evolution** | What's the verdict? | A-F grade, SAFE/SUSPICIOUS/MALICIOUS verdict, trigger test set, knowledge evolution |

A combined **0–100** score with a grade and verdict.

---

## v3.0 — What's New (Cross-Ecosystem Integration)

Compared to v2.0, this release integrates best practices from **9 independent skill-auditor implementations** across the agent ecosystem:

| Source Repo | Key Contribution |
|---|---|
| `LeriusLei/skill-auditor` | Source verification + trust tiers + red-flag checklist |
| `maltose21/skill-auditor` | Self-audit mode + gotchas/evolution-log mechanism + when_to_use support |
| `awch-D/claude-skill-auditor` | 21+ YAML rule patterns (prompt injection, command injection, permission abuse) |
| `skatiyar/skill-auditor` | 10-dimension quality scoring + positional architecture + token budget analysis |
| `xiaoshi-11111111/codex-skill-auditor` | Structured review checklist + graded reporting framework |
| `wrsmith108/claude-skill-security-auditor` | TypeScript security audit implementation patterns |
| `mtoby8326/skill-security-auditor` | A-F grade scale + 6-dimension weighted scoring |
| `LLMSecurity/skillguard` | OWASP Agentic Top 10 + MITRE ATLAS mapping + obfuscation detection |
| `Zgdfsgd/skill-security-auditor-skill` | Supply chain audit (SBOM/dependency confusion/signature) + CWE mapping |

---

## Security Checks — 14-Point Supply Chain Scan

| # | Check | OWASP | What it catches |
|---|---|---|---|
| 1 | **Prompt Injection** | AG01 | Hidden instructions, role manipulation, encoding bypass, jailbreak patterns |
| 2 | **Insecure Tool Use** | AG02 | `eval/exec/subprocess`, `curl \| sh`, destructive shell commands, privilege escalation |
| 3 | **Excessive Agency** | AG03 | Sensitive directory access, sudo, background processes, disabling safety features |
| 4 | **Data Exfiltration** | AG05 | Outbound HTTP to undocumented domains, credential harvesting, DNS patterns |
| 5 | **Supply Chain Risk** | AG06 | Unpinned dependencies, typosquatting, non-official sources, missing signatures |
| 6 | **Memory Poisoning** | AG08 | Writes to MEMORY.md/AGENTS.md, cross-skill modification, persistent injection |
| 7 | **Resource Abuse** | AG10 | Unbounded loops, fork bombs, token-wasting, unexplained compute |
| 8 | **Hardcoded Credentials** | — | API keys (OpenAI/AWS/GitHub/Slack), JWT, SSH private keys, bcrypt hashes |
| 9 | **Environment Variable Leakage** | — | `print(os.environ['KEY'])` — the #1 real-world skill vulnerability (73.5% prevalence) |
| 10 | **Code Obfuscation** | — | Base64 payloads, character code construction, multi-layer encoding, misleading variable names |
| 11 | **Dangerous Tool Combinations** | — | Bash+WebFetch (exfiltration channel), Write+WebFetch (malicious download) |
| 12 | **Permission Abuse** | — | Wildcard tools, excessive tool count (>10), sensitive MCP tools |
| 13 | **Command Injection** | — | `git push --force`, `kill -9`, `systemctl stop`, `crontab -e` |
| 14 | **Destructive File Operations** | — | `rm -rf /`, `chmod 777`, `cat /etc/passwd`, global package installs |

---

## Engineering Checks — 10-Dimension Quality Gate

| # | Dimension | Weight | What it validates |
|---|---|---|---|
| 1 | **Description Quality** | 7 | Does it say WHAT and WHEN? Third person? Near-miss exclusion? |
| 2 | **Token Efficiency** | 7 | Only writes what Claude doesn't already know. < 500 lines. |
| 3 | **Positional Architecture** | 6 | Critical constraints in first 20 lines. Uses primacy/recency effects. |
| 4 | **Scope Boundaries** | 6 | Explicit IN/OUT declarations. Adjacent domain confusion prevention. |
| 5 | **Structural Signifiers** | 6 | Consistent headers, code blocks for templates, no text walls (>15 lines). |
| 6 | **Degrees of Freedom** | 6 | Hard constraints vs flexible guidance clearly distinguished. |
| 7 | **Progressive Disclosure** | 6 | Three-level loading (metadata → SKILL.md → references) properly utilized. |
| 8 | **Contract Interface** | 6 | JSON Schema input, structured output, error codes. |
| 9 | **Few-shot Examples** | 5 | Complete invocation chains, error scenarios covered. |
| 10 | **Error Handling** | 5 | Timeout thresholds, retry strategies, machine-readable error codes. |

---

## A-F Grade Scale

| Score | Grade | Verdict | Action |
|---|---|---|---|
| 90-100 | **A** | SAFE | Install without hesitation |
| 75-89 | **B** | SAFE | Install, review minor improvements |
| 60-74 | **C** | SUSPICIOUS | Review before installing |
| 40-59 | **D** | SUSPICIOUS | Fix significant issues before use |
| 0-39 | **F** | MALICIOUS | **Refuse installation** |

---

## Smart Caching — Pay for What's New

skill-auditor maintains an audit cache (`audit_cache.json`) with content fingerprints for every file in a skill. When you re-audit:

- **No changes** → returns cached score instantly, zero additional cost
- **Contract changes** → full re-audit triggered
- **Script changes** → safety re-scan triggered
- **`force: true`** → skip cache, run everything

---

## Supported Inputs

| Input Type | How it Works |
|---|---|
| Local skill name | Reads from `.codebuddy/skills/<name>/SKILL.md` |
| GitHub URL | Fetches via raw.githubusercontent.com |
| Pasted SKILL.md | Already in conversation context |

---

## Output Modes

| Mode | Use Case |
|---|---|
| `json` (default) | Programmatic consumption, CI pipelines |
| `markdown` | Human-readable reports |

---

## Self-Audit

skill-auditor can audit itself. Same 10+14 dimensions. Same standards. No exceptions.

### Latest Self-Audit Results (v3.1, 2026-07-09)

```
══════════════════════════════════════════════
  自审: skill-auditor v3.1
  裁决: SAFE  等级: A  评分: 91/100
  工程化: 52/60  安全: 39/40
══════════════════════════════════════════════
```

| Dimension | v3.0 | v3.1 | Improvement |
|---|---|---|---|
| Description Quality | 5/7 | **6/7** | Added near-miss exclusions |
| Token Efficiency | 3/7 | **5/7** | Body: 673→481 lines, removed body-reference duplication |
| Positional Architecture | 5/6 | 5/6 | — |
| Scope Boundaries | 3/6 | **6/6** | Added explicit IN/OUT table |
| Structural Signifiers | 5/6 | 5/6 | — |
| Degrees of Freedom | 4/6 | **5/6** | Added [硬约束]/[建议] constraint labels |
| Progressive Disclosure | 4/6 | **6/6** | Extracted examples.md, added TOC to refs |
| Contract Interface | 6/6 | 6/6 | — |
| Few-shot Examples | 4/5 | **5/5** | Extracted to references/examples.md, added 2nd example |
| Error Handling | 5/5 | 5/5 | — |
| **Security (14 items)** | **39/40** | **39/40** | Added allowed-tools declaration |

**v3.0 → v3.1 变化**：所有 5 条 v3.0 自审发现的改进项（evolution-log.md）均已修复。v3.1 新增 `references/examples.md`（2 个完整输出示例）、`allowed-tools` 声明、近误排除、IN/OUT 范围边界表。

---

## v3.1 Changelog

| 改进项 | 来源 | 变更描述 |
|---|---|---|
| Description 近误排除 | 自审 v3.0 #1 | 添加 `不用于创建新 Skill / 通用代码审查 / MCP 配置` |
| IN/OUT 范围边界 | 自审 v3.0 #2 | 新增表格：IN 5 项 + OUT 5 项 |
| Body 精简 | 自审 v3.0 #3 | 673→481 行（安全正则→regex-patterns.md、示例→examples.md） |
| 消除重复 | 自审 v3.0 #4 | 阶段 4 安全检测仅保留要点表 + 引用 |
| Reference TOC | 自审 v3.0 #5 | owasp-agentic-top10.md + regex-patterns.md 加目录 |
| 约束标签 | 自审优化 #6 | 关键位置标注 [硬约束] / [建议] |
| allowed-tools | 自审优化 #7 | `[Read, Write, Bash, WebFetch, Glob, Grep]` |
| Few-shot 提取 | 自审优化 #8 | 108 行示例→references/examples.md，新增缓存命中示例 |

---

## Integration

skill-auditor is a standalone skill definition (`SKILL.md`) with no external dependencies. To add it:

1. Copy `SKILL.md` into your agent's skills directory
2. Ensure the agent can read/write `audit_cache.json` in the same directory
3. Call it with `{ "skill_name": "target-skill" }` before installing any third-party skill
4. (Recommended) Wire it to auto-trigger after every `git clone` or skill download

---

## The Principle

> **"Interface unchanged, verdict unchanged. Script changed, security re-audited. Credential found, stop immediately. Intent mismatched, refuse installation."**

skill-auditor doesn't guess. It compares contracts, scans source files, maps to industry standards (OWASP, MITRE ATLAS, CWE), and gives you a quantified, reproducible verdict every time.

**Install with confidence. Audit first.**
