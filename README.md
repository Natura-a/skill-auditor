# skill-auditor

CodeBuddy Code Skill 质量审计工具，用于审查 Skill 的完整性与可用性，严格依据"契约式接口"原则。

## 功能

- **契约指纹比对**：通过比对输入/输出 Schema 指纹，实现增量跳过，节省 Token 消耗
- **六维工程验收**：语义路由、契约接口、调用范例、异步反馈、熔断兜底、结构化输出
- **审计台账管理**：自动维护 `audit_cache.json`，支持历史评分回溯
- **全量/定向审查**：支持审查单个 Skill 或批量审查所有 Skill

## 使用方式

在 CodeBuddy Code 中输入：

```
审查 skill-auditor 这个技能
```

或批量审查：

```
检查所有技能
```

## 输出示例

```json
{
  "status": "audited",
  "skill_name": "web_scraper",
  "data": {
    "score": 75,
    "details": {
      "semantic_routing": { "pass": true },
      "contract_interface": { "pass": false },
      "few_shot_examples": { "pass": true, "count": 2 },
      "async_feedback": { "pass": false },
      "circuit_breaker": { "pass": true },
      "structured_output": { "pass": true }
    }
  }
}
```
