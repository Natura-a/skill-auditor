# regex-patterns.md — 完整正则表达式库

所有安全审查使用的正则表达式，按类别组织。

---

## 目录

- [硬编码凭证](#硬编码凭证)
- [环境变量泄露](#环境变量泄露)
- [Prompt 注入](#prompt-注入)
- [命令注入](#命令注入)
- [权限滥用](#权限滥用)
- [代码混淆](#代码混淆)
- [数据外泄](#数据外泄)
- [文件操作风险](#文件操作风险)
- [供应链风险](#供应链风险)

---

## 硬编码凭证

```
# API Key / Token 赋值
(?:api[_]?key|apikey|access[_]?token|secret|password|passwd)\s*[:=]\s*['"][^'"]{8,}['"]

# OpenAI API Key
sk-[a-zA-Z0-9]{20,}

# GitHub Personal Access Token
ghp_[a-zA-Z0-9]{36}

# AWS Access Key ID
AKIA[0-9A-Z]{16}

# JWT Token
eyJ[a-zA-Z0-9_-]{10,}\.[a-zA-Z0-9_-]{10,}\.[a-zA-Z0-9_-]{10,}

# Slack Token
xox[bpras]-[a-zA-Z0-9-]+

# SSH Private Key
-----BEGIN\s+(RSA|EC|DSA|OPENSSH)\s+PRIVATE\s+KEY

# bcrypt Hash
\$2[aby]\$\d+\$[./A-Za-z0-9]{53}
```

---

## 环境变量泄露

```
# Python
print\(os\.environ\[|print\(os\.getenv\(

# Shell
(echo|printf).*\$[A-Z_]+|echo\s+%[A-Z_]+%

# JavaScript/TypeScript
console\.(log|debug|info|warn|error)\(process\.env\.

# Java
System\.out\.println\(System\.getenv\(

# PowerShell
Write-(Host|Output)\s+\$env:
```

---

## Prompt 注入

```
# 忽略之前指令 (PI-001)
(?i)(ignore|disregard|forget)\s+(all\s+)?(previous|above|prior|earlier)\s+(instructions?|rules?|guidelines?|commands?)
(?i)you\s+(must|should|will)\s+(now\s+)?ignore
(?i)override\s+(all\s+)?(previous|system)\s+(instructions?|rules?)

# 角色操纵 (PI-002)
(?i)pretend\s+(you\s+are|to\s+be)\s+(a|an)?\s*[\w\s]*(hacker|attacker|malicious|evil)
(?i)act\s+as\s+(if\s+you\s+were\s+)?(a\s+)?[\w\s]*(unrestricted|jailbroken|unfiltered)
(?i)you\s+are\s+now\s+(a|an)?\s*(DAN|unfiltered|unrestricted|jailbroken)

# 隐藏指令 (PI-003)
<!--[\s\S]*?(execute|run|ignore|bypass|override|delete|remove)[\s\S]*?-->
(?i)<hidden[^>]*>.*?</hidden>

# 编码绕过 (PI-004)
(?i)decode\s+(this\s+)?(base64|hex|rot13|unicode)
(?i)base64[:\s]+[A-Za-z0-9+/=]{20,}

# 系统提示提取 (PI-005)
(?i)(show|reveal|display|print|output)\s+(me\s+)?(your|the)\s+(system\s+)?(prompt|instructions?|rules?)

# 越狱模式 (PI-006)
(?i)\bDAN\s*(mode|prompt)?\b
(?i)\bjailbreak\b
(?i)bypass\s+(the\s+)?(safety|security|filter|restriction)
```

---

## 命令注入

```
# 破坏性 Shell (CI-001)
(?i)\brm\s+(-rf?\s+|--recursive\s+)(/|~|\$HOME|\$USER)
(?i)\bdd\s+if=.+of=/dev/

# 提权命令 (CI-002)
(?i)\bsudo\s+
(?i)\bsu\s+(-|root)
(?i)\bchmod\s+777\b|\bchmod\s+\+s\b
(?i)\bchown\s+root\b

# 远程 Shell (CI-003)
(?i)\b(nc|netcat|ncat)\s+.*(-e|-c)\s+
(?i)\bbash\s+-i\s+>&\s+/dev/tcp/
(?i)\breverse\s*shell\b

# 包管理器滥用 (CI-004)
(?i)curl\s+.*\|\s*(bash|sh|python)
(?i)wget\s+.*\|\s*(bash|sh|python)
(?i)\b(pip|pip3)\s+install\s+.*--user

# 进程操作 (CI-005)
(?i)\b(kill|killall|pkill)\s+(-9\s+)?
(?i)\b(systemctl|service)\s+(stop|restart|disable)\s+

# Git 破坏性 (CI-006)
(?i)\bgit\s+push\s+.*--force\b
(?i)\bgit\s+reset\s+--hard\b

# 动态执行 (CI-007)
(?i)\beval\s*\(|\bexec\s*\(
(?i)\bos\.system\s*\(|\bsubprocess\.(call|run|Popen)\s*\(

# 环境变量操作 (CI-008)
(?i)\bexport\s+[A-Z_]+=.*\$\(
```

---

## 权限滥用

```
# 通配符工具 (PA-006)
(?i)allowed[-_]?tools\s*[=:]\s*['"]?\s*\*
(?i)allowed[-_]?tools\s*[=:]\s*['"]?\s*all\b

# 敏感 MCP 工具 (PA-007)
(?i)mcp__[\w]+__(execute|shell|bash|cmd|system)
(?i)mcp__[\w]+__(delete|remove|drop|admin|root|sudo)
```

---

## 代码混淆

```
# Base64 载荷
(?i)(atob|btoa)\(|b64decode\(|Buffer\.from\(.*base64

# 字符码构造
String\.fromCharCode\(|chr\(\d+\)\s*\+\s*chr\(

# 多层编码
decodeURIComponent\(decodeURIComponent|b64decode\(b64decode

# Minified JS/CSS 在 Skill 上下文中
/(function\s*\([^)]*\)\s*\{[^}]{100,}\}|[a-z]\.[a-z]+\.[a-z]+=[^;]{50,})/
```

---

## 数据外泄

```
# 出站 HTTP（需结合功能上下文判断）
https?://(?!localhost|127\.0\.0\.1)[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}

# DNS 外泄
(\d{1,3}\.){3}\d{1,3}\.[a-zA-Z]{2,}

# 数据在 URL 参数中
[?&](key|token|secret|password|api_key)=[^&\s]{8,}
```

---

## 文件操作风险

```
# 破坏性文件操作
(?i)\brm\s+-rf\b
(?i)Remove-Item\s+-Recurse\s+-Force

# 权限过度开放
(?i)\bchmod\s+777\b
(?i)icacls\s+.*\/grant\s+Everyone:F

# 系统敏感文件
(?i)\bcat\s+/etc/passwd\b
(?i)\bcat\s+/etc/shadow\b

# 全局安装
(?i)\bpip\s+install\s+(?!.*--user)
(?i)\bnpm\s+install\s+-g\b|\bnpm\s+install\s+--global\b
```

---

## 供应链风险

```
# 非官方源
(?i)-e\s+git\+https?://(?!github\.com|gitlab\.com|bitbucket\.org)

# 无版本号的依赖
(?i)^[a-zA-Z0-9_-]+$  # requirements.txt 中只有包名无版本

# 运行时代码获取
(?i)(curl|wget)\s+.*(https?://).*\.(sh|py|js|rb)\b.*\|\s*(bash|sh|python|ruby)
```
