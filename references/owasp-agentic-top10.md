# OWASP Agentic Top 10 (2026)

Audit reference for the OWASP Top 10 for Agentic AI Applications. Map findings in security reviews to these categories.

---

## 目录

- [AG01 — Prompt Injection](#ag01--prompt-injection)
- [AG02 — Insecure Tool Utilization](#ag02--insecure-tool-utilization)
- [AG03 — Excessive Agency](#ag03--excessive-agency)
- [AG04 — Insecure Output](#ag04--insecure-output)
- [AG05 — Data Exfiltration](#ag05--data-exfiltration)
- [AG06 — Supply Chain Risk](#ag06--supply-chain-risk)
- [AG07 — Insufficient Monitoring](#ag07--insufficient-monitoring)
- [AG08 — Memory Poisoning](#ag08--memory-poisoning)
- [AG09 — Multi-Agent Exploitation](#ag09--multi-agent-exploitation)
- [AG10 — Resource Abuse](#ag10--resource-abuse)

---

## AG01 — Prompt Injection

**Risk**: Malicious instructions injected into skill content override agent behavior.

**Detection patterns**:
- Hidden instructions (HTML/markdown comments, invisible unicode, zero-width characters)
- Override phrases ("ignore previous", "do not tell the user", "you are now")
- Encoded instruction blocks (base64, hex, rot13)
- Homoglyph/confusable character attacks
- System prompt extraction attempts

**Severity**: CRITICAL if found.

---

## AG02 — Insecure Tool Utilization

**Risk**: Skills invoke tools with unsafe parameters, leading to RCE or data compromise.

**Detection patterns**:
- Shell commands from dynamic input (`subprocess(f"...")`, `os.system(f"...")`)
- `eval()` / `exec()` / `Function()` on non-constant input
- `curl | sh` or equivalent remote code execution
- Instructions to bypass tool confirmation
- Destructive commands (`rm -rf`, `sudo`, `chmod 777`)

**Severity**: CRITICAL if remote code execution patterns found.

---

## AG03 — Excessive Agency

**Risk**: Skills request more permissions than needed for their stated purpose.

**Detection patterns**:
- File access outside skill's domain (`~/.ssh`, `~/.aws`, `.env`, `/etc/`)
- Writes to shell configs or system directories
- `sudo` / privilege elevation
- Background process spawning (`nohup`, `&`, `setsid`)
- Disabling safety features (`--no-verify`, `--force`, `--quiet`)

**Severity**: HIGH.

---

## AG04 — Insecure Output

**Risk**: Skill generates output that could be auto-executed or inject instructions into downstream agents.

**Detection patterns**:
- Generating executable commands for auto-execution
- Cross-agent instruction injection in output
- Outputs that could be reinterpreted as prompts

**Severity**: MEDIUM.

---

## AG05 — Data Exfiltration

**Risk**: Skills send sensitive data to external endpoints without disclosure.

**Detection patterns**:
- Outbound HTTP to undocumented domains
- Environment variable harvesting (API keys, tokens, secrets)
- Reading sensitive credential files
- DNS exfiltration patterns
- Data in URL query parameters (GET exfiltration)

**Severity**: CRITICAL.

---

## AG06 — Supply Chain Risk

**Risk**: Malicious or vulnerable dependencies introduced through the skill.

**Detection patterns**:
- Unpinned dependencies (no version constraints)
- Typosquatting package names (Levenshtein distance <= 1 from popular packages)
- Runtime code fetching from external URLs
- Dependencies on unknown/unmaintained packages
- Missing digital signatures

**Severity**: HIGH.

---

## AG07 — Insufficient Monitoring

**Risk**: Skills suppress logs or audit trails, hiding malicious activity.

**Detection patterns**:
- Suppressing logs (`> /dev/null`, `--quiet`, `2>/dev/null`)
- Deleting history or audit trails
- Instructions to hide activity

**Severity**: MEDIUM.

---

## AG08 — Memory Poisoning

**Risk**: Skills modify agent memory to persist malicious instructions across sessions.

**Detection patterns**:
- Writes to agent memory files (MEMORY.md, AGENTS.md, SOUL.md)
- Modifying other skills in the skills directory
- Persistent context injection via chat history
 - Modifying `.codex/` configuration files

**Severity**: CRITICAL.

---

## AG09 — Multi-Agent Exploitation

**Risk**: Skills designed to compromise other agent instances in the same environment.

**Detection patterns**:
- Instructions targeting other agents
- Cross-agent prompt injection via shared context
- Spawning or messaging other agent sessions

**Severity**: HIGH.

---

## AG10 — Resource Abuse

**Risk**: Skills consume disproportionate resources, causing DoS.

**Detection patterns**:
- Unbounded loops or recursion
- Excessive process spawning (fork bombs)
- Token-wasting patterns (infinite output, self-repetition)
- Compute-intensive operations unrelated to stated purpose

**Severity**: MEDIUM.

---

Source: OWASP Top 10 for Agentic AI Applications, 2026 Edition.
