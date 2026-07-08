# skill-auditor

> **The missing security layer for AI agent skills.**
>
> You wouldn't run `npm install` without `npm audit`. Why would you let an AI agent load third-party skills without one?

---

## The Problem

AI agents are the new runtime. Skills, plugins, custom instructions — whatever your platform calls them — are the new dependencies. And right now, **there is no audit**.

Every skill you install gets the same trust as code you wrote yourself. It reads your prompts, accesses your tools, and runs scripts on your machine. One malicious skill can leak API keys, exfiltrate conversation history, or inject hidden commands into your agent's workflow.

**skill-auditor is the audit step that should exist before any skill touches your agent.**

---

## What It Does

skill-auditor evaluates every skill on two independent axes:

| Dimension | Score | What it checks |
|---|---|---|
| **Engineering Quality** | 0–80 | Does the skill actually work? Contract completeness, error handling, timeout logic, structured output |
| **Supply Chain Safety** | 0–20 | Can this skill be trusted? Credential leaks, hidden commands, suspicious dependencies, file permission overreach |

A combined score of **0–100** with a risk label: `safe` / `low` / `medium` / `high` / `critical`.

---

## Why It Matters — For Every AI Agent Platform

skill-auditor doesn't care if your agent is built with CodeBuddy, Cursor, Claude Code, Copilot, or a custom framework. It audits the artifact — the skill definition file itself. As long as your skills are defined as Markdown/YAML with a standard contract structure (description, input schema, output schema, tool calls), the auditor works.

**Before you install, you audit. Before you trust, you verify.**

---

## Safety Checks — 7-Point Supply Chain Scan

| # | Check | What it catches |
|---|---|---|
| 1 | **Hardcoded credentials** | API keys, tokens, passwords, JWT secrets baked into skill files |
| 2 | **Environment variable leakage** | `print(os.environ['KEY'])` — the #1 real-world skill vulnerability (73.5% prevalence in academic study) |
| 3 | **Network permission audit** | Does a text formatter really need internet access? |
| 4 | **Hidden command detection** | Ghost shell commands buried in long text, disguised in links, or smuggled in Markdown |
| 5 | **File permission audit** | Is the skill reading `.ssh/`, `.aws/`, `.env` when it shouldn't? |
| 6 | **Dependency source verification** | `requirements.txt` / `package.json` pulling from non-official repos? |
| 7 | **Destructive file operation detection** | `rm -rf`, `chmod 777`, global installs in skill scripts |

---

## Engineering Checks — 6-Dimension Quality Gate

| # | Dimension | What it validates |
|---|---|---|
| 1 | **Semantic routing** | Is the trigger condition clear enough for an LLM to decide when to invoke? |
| 2 | **Contract interface** | Is input declared as JSON Schema? Does output follow `{status, data, error}`? |
| 3 | **Usage examples** | Are there complete end-to-end invocation chains? |
| 4 | **Async feedback** | Long-running operations: does the skill return a `task_id` for polling? |
| 5 | **Circuit breaker** | Timeout thresholds, retry strategies, machine-readable error codes? |
| 6 | **Structured output** | Does the skill enforce JSON output instead of free text? |

---

## Smart Caching — Pay for What's New

skill-auditor maintains an audit cache (`audit_cache.json`) with content fingerprints for every file in a skill. When you re-audit:
- **No changes** → returns cached score instantly, zero additional cost
- **Contract changes** → full re-audit triggered
- **Script changes** → safety re-scan triggered
- **`force: true`** → skip cache, run everything

---

## Risk-Based Decisions — Automate Your Trust Policy

The output risk label isn't just a label. Use it to program decisions:

| Risk | Score Range | Execution Strategy | Recommended Action |
|---|---|---|---|
| `safe` | 20 | `auto_install` | Install without hesitation, no prompt needed |
| `low` | 17–19 | `auto_install_with_advisory` | Install, display a one-line safety note |
| `medium` | 13–16 | `install_after_warning` | Install, show flagged items, let user decide whether to keep |
| `high` | 8–12 | `hold_for_review` | **Pause installation**. Present full risk report. Require explicit user approval before proceeding. |
| `critical` | 0–7 | `block` | **Reject installation**. Do not write skill files to disk. Report findings immediately. |

The `Execution Strategy` column is machine-actionable — wire `overall_risk` into your agent's download handler, CI pipeline, or skill marketplace gateway to enforce policy automatically.

---

## Integration — Drop It Into Any Agent Platform

skill-auditor is a standalone skill definition file (`SKILL.md`) with no external dependencies. To add it to your agent:

1. Copy the `SKILL.md` into your agent's skills directory
2. Ensure the agent can read/write a `audit_cache.json` file in the same directory
3. Call it with `{ "skill_name": "target-skill" }` before installing any third-party skill
4. (Recommended) Wire it to auto-trigger after every `git clone` or skill download

Want audit as a service? Run it as a pre-commit hook, a CI check, or a gateway in your skill marketplace pipeline.

---

## Output — Machine-Readable, Decision-Ready

```json
{
  "status": "audited",
  "skill_name": "third-party-scraper",
  "data": {
    "engineering_score": 68,
    "safety_score": 7,
    "score": 75,
    "overall_risk": "medium",
    "details": {
      "contract_interface": { "pass": false },
      "structured_output": { "pass": true },
      "safety_review": {
        "hardcoded_credentials": { "pass": true },
        "env_leakage": { "pass": false, "count": 1 },
        "hidden_commands": { "pass": true }
      }
    },
    "suggestions": [
      "Remove print(os.environ['API_KEY']) from scripts/scraper.py:45"
    ]
  }
}
```

---

## The Principle

> **"Interface unchanged, verdict unchanged. Script changed, security re-audited. Credential found, stop immediately."**

skill-auditor doesn't guess. It compares contracts, scans source files, and gives you a quantified, reproducible verdict every time.

**Install with confidence. Audit first.**
