# skill-auditor

CodeBuddy Code Skill 质量 + 安全双维审计工具，严格依据"契约式接口"原则与"供应链安全"标准。

## 功能

### 工程化审计（80 分）
- **契约指纹比对**：通过比对输入/输出 Schema 指纹，实现增量跳过，节省 Token 消耗
- **六维工程验收**：语义路由、契约接口、调用范例、异步反馈、熔断兜底、结构化输出
- **审计台账管理**：自动维护 `audit_cache.json`，支持历史评分回溯

### 安全审计（20 分）
- **硬编码凭证检测**：扫描 API Key / Token / Password 是否硬编码在代码中
- **环境变量泄露检测**：检查是否存在 `print(os.environ)` 等敏感操作
- **网络权限审查**：网络权限声明是否与 Skill 功能合理匹配
- **文件权限审查**：文件读写权限声明是否限定在合理目录范围内
- **隐藏指令检测**：是否存在隐藏在正常内容中的幽灵指令
- **依赖风险检测**：Python/npm 依赖包是否来自可信源

### 综合评分
- **总分**：engineering_score（0-80）+ safety_score（0-20）
- **风险等级**：safe / low / medium / high / critical
- **安全快速扫描**：支持 `safety_only` 模式，仅执行安全审查

## 使用方式

在 CodeBuddy Code 中输入：

```
审查 skill-auditor 这个技能
```

定向审查指定 Skill：

```
审计 bilibili-analyzer
```

批量审查：

```
检查所有技能
```

仅安全扫描：

```
安全审计 humanizer-zh
```

## 自动触发

以下场景会自动触发审计：
- `mcp-skill-config` 完成新 Skill 配置后
- 从外部仓库下载/安装新 Skill 后（匹配 "配置完成"、"git clone" 等上下文信号）

## 输出示例

```json
{
  "status": "audited",
  "skill_name": "web_scraper",
  "data": {
    "engineering_score": 68,
    "safety_score": 7,
    "score": 75,
    "overall_risk": "medium",
    "details": {
      "semantic_routing": { "pass": true },
      "contract_interface": { "pass": false },
      "few_shot_examples": { "pass": true, "count": 2 },
      "async_feedback": { "pass": false },
      "circuit_breaker": { "pass": true },
      "structured_output": { "pass": true },
      "safety_review": {
        "pass": false,
        "overall_risk": "medium",
        "sub_checks": {
          "hardcoded_credentials": { "pass": true, "count": 0 },
          "env_leakage": { "pass": true, "count": 0 },
          "network_permission": { "pass": true, "risk_level": "low" },
          "hidden_commands": { "pass": false, "count": 1 },
          "file_permission": { "pass": true, "risk_level": "low" },
          "dependency_risk": { "pass": true },
          "file_operations_risk": { "pass": true, "risk_level": "low" }
        }
      }
    }
  }
}
```
