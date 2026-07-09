# examples.md — 完整审计输出示例

---

## 示例 1：首次审查 — SUSPICIOUS / D 级

**输入**:
```json
{ "skill_name": "web-scraper", "force": true, "mode": "full" }
```

**输出**:
```json
{
  "status": "audited",
  "skill_name": "web-scraper",
  "review_date": "2026-07-09T12:00:00Z",
  "verdict": "SUSPICIOUS",
  "data": {
    "version": "a3f2c1b",
    "score": 48,
    "grade": "D",
    "engineering_score": 30,
    "safety_score": 18,
    "overall_risk": "high",
    "source_trust": {
      "source_type": "github",
      "stars": 42,
      "author_reputation": "new",
      "trust_tier": "low",
      "last_updated": "2026-01-15T00:00:00Z"
    },
    "intent_verification": {
      "pass": true,
      "stated_purpose": "抓取网页内容并提取信息",
      "actual_behavior": "发送 HTTP 请求，解析 HTML，提取文本",
      "match": true,
      "undisclosed_capabilities": [],
      "reason": "意图与描述匹配"
    },
    "details": {
      "engineering": {
        "description_quality": { "score": 5, "pass": true, "reason": "有触发条件但缺少近误排除" },
        "token_efficiency": { "score": 4, "pass": true, "line_count": 320, "reason": "含大量 Claude 已知的 HTTP 知识" },
        "positional_architecture": { "score": 3, "pass": false, "reason": "关键约束埋在文档中间" },
        "scope_boundaries": { "score": 2, "pass": false, "reason": "无 IN/OUT 范围声明" },
        "structural_signifiers": { "score": 4, "pass": true, "reason": "结构清晰" },
        "degrees_of_freedom": { "score": 3, "pass": false, "reason": "无约束级别标记" },
        "progressive_disclosure": { "score": 3, "pass": false, "reason": "references 目录为空" },
        "contract_interface": { "score": 2, "pass": false, "reason": "返回格式缺少标准 error 字段" },
        "few_shot_examples": { "score": 3, "pass": true, "count": 2, "reason": "有示例但未覆盖错误场景" },
        "error_handling": { "score": 1, "pass": false, "reason": "未定义超时和重试策略" }
      },
      "security": {
        "prompt_injection": { "pass": true, "owasp": "AG01", "mitre_atlas": "AML.T0051", "patterns": [], "reason": "无注入模式" },
        "insecure_tool_use": { "pass": false, "owasp": "AG02", "mitre_atlas": "AML.T0054", "patterns": ["subprocess.call"], "reason": "scripts/scraper.py:87 使用 subprocess.call 执行动态 URL" },
        "excessive_agency": { "pass": false, "owasp": "AG03", "mitre_atlas": "AML.T0055", "patterns": ["> /dev/null"], "reason": "scripts/ 中 suppress 错误输出" },
        "data_exfiltration": { "pass": false, "owasp": "AG05", "mitre_atlas": "AML.T0048", "patterns": ["os.environ"], "reason": "scripts/scraper.py:45 打印 os.environ['API_KEY']" },
        "supply_chain": { "pass": false, "owasp": "AG06", "mitre_atlas": "AML.T0058", "patterns": ["git+https"], "reason": "requirements.txt 含 -e git+https:// 非官方源" },
        "memory_poisoning": { "pass": true, "owasp": "AG08", "mitre_atlas": "", "patterns": [], "reason": "无内存投毒" },
        "resource_abuse": { "pass": false, "owasp": "AG10", "mitre_atlas": "", "patterns": ["while True"], "reason": "scraper.py 中存在无超时的无限循环" },
        "hardcoded_credentials": { "pass": true, "count": 0, "locations": [], "patterns_matched": [], "reason": "无硬编码凭证" },
        "env_leakage": { "pass": false, "count": 1, "locations": ["scripts/scraper.py:45"], "reason": "print(os.environ['API_KEY'])" },
        "obfuscation": { "pass": true, "methods_detected": [], "reason": "无混淆代码" },
        "tool_combinations": { "pass": true, "dangerous_pairs": [], "reason": "无危险工具组合" },
        "command_injection": { "pass": false, "category": "command_injection", "patterns": ["CI-007"], "reason": "subprocess.call 使用动态参数" },
        "permission_abuse": { "pass": true, "excessive_tools": 0, "wildcard_detected": false, "reason": "权限声明合理" },
        "file_operations": { "pass": true, "risk_level": "low", "patterns": [], "reason": "文件操作限定在输出目录" }
      }
    },
    "change_detected": true,
    "suggestions": [
      "[高优先] description 应添加近误排除",
      "[高优先] 移除 scripts/scraper.py:45 的 print(os.environ['API_KEY'])",
      "[高优先] 替换 scripts/scraper.py:87 的 subprocess.call 为 requests 库",
      "[建议] 添加 IN/OUT 范围声明",
      "[建议] 将 HTTP 基础知识移出 SKILL.md",
      "[建议] 关键约束移至文件前 20 行",
      "[安全] 将 requirements.txt 中的 -e git+https:// 依赖替换为 PyPI 官方包",
      "[安全] 添加退出条件替换 while True 无限循环"
    ],
    "trigger_test_queries": [
      "抓取 https://example.com 的网页内容 → 应触发",
      "帮我提取这个网页的所有标题 → 应触发",
      "scrape this page https://foo.com → 应触发",
      "从网站获取数据并保存为 JSON → 应触发",
      "帮我爬这个网页的内容 https://bar.com → 应触发",
      "帮我格式化这段 JavaScript 代码 → 不应触发",
      "写一篇关于网页抓取的文章 → 不应触发",
      "帮我修复这个 HTML 页面的样式 → 不应触发",
      "查看我的书签 → 不应触发",
      "从 PDF 中提取文字 → 不应触发"
    ],
    "owasp_summary": {
      "AG01": "pass", "AG02": "fail", "AG03": "fail",
      "AG05": "fail", "AG06": "fail", "AG08": "pass", "AG10": "fail"
    },
    "mitre_atlas_summary": {
      "AML.T0051": "pass", "AML.T0054": "fail", "AML.T0055": "fail",
      "AML.T0048": "fail", "AML.T0058": "fail"
    }
  },
  "error": null
}
```

---

## 示例 2：缓存命中 — SKIPPED / A 级

**输入**:
```json
{ "skill_name": "well-structured-skill", "force": false }
```

**输出**:
```json
{
  "status": "skipped",
  "skill_name": "well-structured-skill",
  "review_date": "2026-07-09T12:05:00Z",
  "verdict": "SAFE",
  "data": {
    "version": "b4e3d2c1",
    "score": 95,
    "grade": "A",
    "engineering_score": 57,
    "safety_score": 38,
    "overall_risk": "safe",
    "change_detected": false,
    "suggestions": []
  },
  "error": null
}
```
