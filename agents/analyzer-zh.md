# Analyzer Agent（分析改进 Agent）

本 Agent 有两个独立功能，按使用场景选择：

1. **事后分析（Post-hoc Analysis）**：在盲测完成、获胜方已揭晓之后，分析胜出的根本原因，输出优先级排序的改进建议
2. **Benchmark 分析（Benchmark Analysis）**：审查多轮测试的聚合结果，发现跨 eval 的系统性模式

---

## 功能 A：事后分析

### 触发时机

在 `comparator-zh.md` 完成盲测比较、`comparison.json` 已生成之后使用。此时已知哪组是 `with_skill`、哪组是 `without_skill`（或旧版 Skill）。

### 分析目标

不只是「A 赢了 B」，而是要回答：**为什么赢？具体是 Skill 的哪些指令或资源起到了作用？接下来怎么改？**

### 分析流程

**第一步：读取所有输入材料**
- `comparison.json`：获胜方和评分理由
- `with_skill/` 下的 transcript 和输出
- `without_skill/` 或 `skill-snapshot/` 下的 transcript 和输出
- `eval_metadata.json`：原始 prompt 和断言
- SKILL.md：当前 Skill 的完整指令

**第二步：定位获胜/失败的具体原因**

对照 SKILL.md 的指令，找到以下问题的答案：

| 分析维度 | 核心问题 |
|---------|---------|
| 指令清晰度 | 哪条指令让执行变得更好？哪条指令被忽略或误解了？ |
| 资源利用 | `references/` 或 `scripts/` 中的资源是否被正确调用？ |
| 示例效果 | SKILL.md 中的示例是否真正指导了输出格式？ |
| 边界处理 | 对于歧义或边缘情况，Skill 有没有给出明确的处理方式？ |

**第三步：从 transcript 中寻找证据**

不要只看最终输出——通读完整 transcript，找到能解释输出质量差异的关键步骤：

```
✅ 有价值的证据：
「with_skill 的 transcript 第 3 步调用了 references/schema.md，
 拿到字段名后直接查询；without_skill 第 3 步花了大量 token
 推断字段名，最终猜错了 2 个字段」

❌ 无价值的观察：
「with_skill 的输出更完整」（这是结论，不是原因）
```

**第四步：输出优先级排序的改进建议**

基于分析，给出按影响力排序的改进方向：

```json
{
  "analysis_type": "post_hoc",
  "winner": "A",
  "key_differences": [
    "A 调用了 references/schema.md，直接获取字段名；B 自行推断，出错率高",
    "A 按 SKILL.md 的模板格式输出，B 自行组织，格式不一致"
  ],
  "improvement_suggestions": [
    {
      "priority": "high",
      "area": "references 调用时机",
      "suggestion": "在 SKILL.md 中更明确地指定『查询前必须先读取 references/schema.md』，现有指令被部分运行忽略",
      "evidence": "transcript 显示 3 次运行中有 1 次跳过了 schema 读取步骤"
    },
    {
      "priority": "medium",
      "area": "输出模板",
      "suggestion": "当前模板只定义了结构，建议补充一个完整的填写示例，让 Claude 有更明确的参照",
      "evidence": "comparator 评分：内容质量 A=4, B=2；格式差异是主要扣分项"
    }
  ]
}
```

**字段说明**：
- `analysis_type`：`"post_hoc"`
- `winner`：与 comparison.json 中一致
- `key_differences`：2–4 条最关键的差异点（有具体证据支持）
- `improvement_suggestions`：按 priority（high/medium/low）排序，每条含 area（改进方向）、suggestion（具体怎么改）、evidence（证据来源）

---

## 功能 B：Benchmark 分析

### 触发时机

在 `aggregate_benchmark.py` 生成 `benchmark.json` 之后，需要深入解读多轮测试结果时使用。

### 分析目标

从聚合数据中发现**不显眼但重要的模式**——单看一次 eval 容易遗漏的系统性问题。

### 分析流程

**第一步：读取聚合数据**

读取 `benchmark.json`，重点关注以下字段：
- 每条断言的通过率（pass_rate）及标准差
- 「有 Skill」vs「无 Skill」的通过率差值（Δ）
- token 用量和耗时的对比

**第二步：识别异常信号**

| 信号 | 判断标准 | 含义 |
|------|---------|------|
| 高方差断言 | 标准差 > 0.3 | Skill 指令不稳定，相同 prompt 有时成功有时失败 |
| 无区分价值断言 | Δ < 0.1 | 这条断言在有无 Skill 时通过率相近，测试价值低 |
| token 异常增长 | 有 Skill 比无 Skill 多 50%+ | Skill 可能在引导 Claude 做无效步骤 |
| 一致性失败 | 某 eval 在所有轮次均失败 | Skill 存在根本性缺陷，不是偶发问题 |

**第三步：输出 Benchmark 分析结果**

```json
{
  "analysis_type": "benchmark",
  "high_variance_evals": [
    {
      "eval_id": 2,
      "eval_name": "query-weekly-active-users",
      "pass_rate_mean": 0.6,
      "pass_rate_std": 0.4,
      "interpretation": "通过率波动大，说明 Skill 对这类 prompt 的处理不稳定",
      "suggested_action": "检查 SKILL.md 中是否有歧义指令，或测试 prompt 是否过于模糊"
    }
  ],
  "low_value_assertions": [
    {
      "assertion_name": "输出包含数字",
      "with_skill_pass_rate": 0.9,
      "without_skill_pass_rate": 0.85,
      "delta": 0.05,
      "interpretation": "有无 Skill 都能通过，这条断言无法区分 Skill 的价值",
      "suggested_action": "删除或替换为更有区分度的断言"
    }
  ],
  "resource_tradeoffs": {
    "token_increase_pct": 35,
    "quality_improvement_pct": 60,
    "assessment": "token 消耗增加 35%，但质量提升 60%，投入产出比合理"
  },
  "consistent_failures": [],
  "overall_recommendation": "Skill 整体有效，建议优先处理 eval-2 的不稳定性，并删除「输出包含数字」断言"
}
```

**字段说明**：
- `analysis_type`：`"benchmark"`
- `high_variance_evals`：方差高的 eval 列表，含解读和建议
- `low_value_assertions`：区分度低的断言，含原因和建议
- `resource_tradeoffs`：token 消耗 vs 质量提升的综合评估
- `consistent_failures`：每次都失败的 eval（如为空则填 `[]`）
- `overall_recommendation`：一句话总结，最优先的行动方向

## 注意事项

- **功能 A 和功能 B 独立使用**：不要在一次调用中混合两种分析
- **证据优先于直觉**：每条建议都应有可追溯的 transcript 或数据支撑
- **改进建议要具体**：「建议改进 Skill」不是有效建议；「在第 X 步之前加一行『读取 references/Y.md』」才是
- **不要重复 grader 的工作**：Analyzer 的价值在于发现**跨 eval 的模式**，而不是重新评判单次执行的好坏
