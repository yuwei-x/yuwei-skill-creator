---
name: yuwei-skill-creator
description: 创建、更新和优化 Skill 的完整指导手册。当用户想把某个工作流变成可复用的技能、把专业知识打包给 Agent 使用、或者现有 Skill 效果不好需要改进时，使用此 skill。即使用户没有明确说「Skill」，只要他们在描述一个希望 Agent 能反复执行的专业任务，也应触发此 skill——比如「我每次都要手动做这件事」「能不能让 Agent 以后自动帮我做 X」「上次那个流程能不能固定下来」，都是强烈的触发信号。Guide for creating, updating, and packaging Agent skills — use whenever someone wants to make Agent reliably do a specific type of task, even if they don't use the word "skill".
---

# Skill Creator

本 Skill 提供创建高质量 Skill 的完整指导。

## 关于 Skill

Skill 是可复用的专业能力包，通过提供专业知识、工作流程和工具集成，将通用 Agent 转变为特定领域的专家 Agent。可以将其理解为「给另一个 Agent 实例写的入职手册」——包含该实例无法仅凭训练获得的程序性知识。

### Skill 能提供什么

| 类型 | 说明 | 典型场景 |
|------|------|---------|
| 专业工作流 | 特定领域的多步骤操作流程 | 小红书选题→发布全流程 |
| 工具集成 | 特定 API 或文件格式的操作指令 | 飞书多维表格查询、企业微信推送 |
| 领域知识 | 企业特有 Schema、业务规则、内部政策 | 合同模板、数据库结构文档 |
| 打包资源 | 脚本、参考文档、输出模板 | WPS 报告模板、Python 处理脚本 |

### Skill 的文件结构

```
skill-name/
├── SKILL.md（必须）
│   ├── YAML 元数据（必须）
│   │   ├── name:（必须，使用 kebab-case 英文，如 weibo-publisher）
│   │   └── description:（必须，建议中英双语）
│   └── Markdown 正文指令（必须）
└── 附属资源（可选）
    ├── scripts/     可执行脚本（Python / Bash 等）
    ├── references/  参考文档（按需加载到 context）
    └── assets/      输出资产（模板、字体、图标等）
```

**命名规范**：`name` 字段使用 kebab-case 英文（如 `weibo-publisher`），description 和 SKILL.md 正文可全中文。

### 三层渐进式加载原则

1. **元数据（name + description）**：始终在 context，决定何时触发此 Skill
2. **SKILL.md 正文**：触发时加载，建议 2000 字以内
3. **附属资源（scripts / references / assets）**：按需加载；scripts/ 可直接执行，references/ 需在正文中注明读取时机，assets/ 用于产出物

| 目录 | 用途 | 何时添加 |
|------|------|---------|
| `scripts/` | 可执行脚本（Python / Bash）| 同段代码反复出现，或需确定性执行结果 |
| `references/` | 按需加载的参考文档 | Schema 定义、品牌规范、业务规则等大量背景知识 |
| `assets/` | 产出物资源（模板、图标等）| Skill 的最终输出需要携带文件 |

**注意**：Agent 不会主动读取 references/ 文件——必须在 SKILL.md 正文中明确写出「在什么情况下读取哪个文件」。超过 300 行的参考文档，建议在文件开头加目录。同一内容只存放一处，不要在 SKILL.md 和 references 中重复。

---

## 与用户沟通

创建 Skill 时，用户可能对技术概念不熟悉——遇到时，用类比解释，不要假设他们懂：

| 技术概念 | 建议类比 |
|----------|---------|
| evals / 测试用例 | 「模拟真实使用的例子，用来检验 Skill 有没有按预期工作」 |
| assertions / 断言 | 「你认为好的输出应该满足哪些条件？用来打分」 |
| references/ 目录 | 「Skill 的参考手册，需要时才翻阅」 |
| 三层加载 | 「就像手机 app——先加载图标，点进去才加载内容，需要下载才加载资源」 |
| benchmark | 「把多次测试结果汇总成一张成绩单，看有 Skill 和没有 Skill 的差距」 |
| 盲测 | 「让第三方在不知道哪个是新版的情况下打分，避免自己评自己时的偏心」 |

询问问题时，每次只问最重要的一个，避免一次性提问过多。

---

## 搭建思路：动手前先想清楚这六件事

| # | 核心问题 | 对应位置 |
|---|---------|---------|
| ① 设定名称 | Skill 叫什么名称？用户说什么话/在什么场景下触发？| `name` + `description` |
| ② 明确目标 | 用 1-2 句话说清楚要产出什么结果（不是做什么操作）| SKILL.md 开头目的说明 |
| ③ 确定步骤 | 自己手动做时的完整流程、判断点、分支逻辑 | SKILL.md 工作流主体 |
| ④ 备好资源 | 需要哪些资源，比如反复查阅的文档、可复用脚本、用户需提供的素材 | scripts / references / assets |
| ⑤ 想好风险 | Agent 最可能在哪些环节犯错、误解意图、走偏方向 | SKILL.md 边界条件 |
| ⑥ 迭代完善 | Skill 不是一次写好就完事，建立测试和迭代习惯 | 第五步、第六步 |

**与创建流程的对应**：第一步→①②，第二步→④，第四步→②③⑤（description→①），第五六步→⑥。

---

## Skill 创建流程

以下步骤按推荐顺序排列。理解每步的目的后，可根据实际情况灵活调整——比如用户已有明确需求时可跳过探索步骤，已有现成 Skill 时可直接从第四步开始。

### 第一步：理解 Skill 的使用场景

**跳过条件**：使用模式已非常明确时可跳过，但即便更新现有 Skill 也建议执行。

通过具体例子来理解 Skill 需要做什么，可以来自用户直接提供，或自行生成后请用户确认。如果用户已经在对话中演示了某个工作流，先从对话历史中提取答案（用了哪些工具、步骤顺序、用户在哪里做了纠正），再请用户确认或补充。

示例：构建「小红书内容创作」Skill 时，相关问题包括：
- 「这个 Skill 需要支持哪些功能？生成标题、写正文、选话题标签，还是全流程？」
- 「能举几个用户实际会怎么使用的例子吗？」
- 「用户会说哪些话来触发这个 Skill？比如『帮我写一篇小红书』？」

**原则**：每次只问最重要的问题。

**结束条件**：对 Skill 需要支持的功能范围有清晰判断时，进入下一步。

**深挖边缘场景**（可按需询问，特别是复杂工作流 Skill）：
- 「如果用户提供的格式不对，Skill 应该怎么处理？」
- 「有没有用户会提但这个 Skill 不该处理的任务？」
- 「最常见的失败场景是什么？」

### 第二步：规划可复用内容

将具体使用例子转化为可复用资源。对每个例子逐一分析：
1. 从零开始执行这个例子，需要哪些步骤？
2. 哪些内容在每次执行时都会重复出现？

**分析示例：**

场景：`wps-report` Skill，处理「帮我把这份数据做成 WPS 月报」：
1. 每次都需要设置相同的表头、样式和图表格式
2. → 应将 `assets/monthly-report.xlsx` 模板存入 Skill

场景：`feishu-query` Skill，处理「今天有多少新用户注册？」：
1. 每次都需要重新查找飞书多维表格的字段名和关系
2. → 应将 `references/schema.md` 存入 Skill

**产出物**：列出需要包含的可复用资源清单（scripts / references / assets）。

### 第三步：初始化 Skill

**跳过条件**：Skill 已存在，仅需迭代或打包时跳过，直接进入第四步。

新建 Skill 时，手动创建标准目录结构，再按需添加或删除子目录：

```bash
mkdir -p <skill-name>/{scripts,references,assets}
touch <skill-name>/SKILL.md
```

### 第四步：编写 Skill 内容

**核心视角**：这份 Skill 是写给另一个 Agent 实例的。聚焦于：对该实例有价值且非显而易见的程序性知识、领域细节和可复用资源。

#### 4.1 先完成附属资源

从第二步确定的资源清单开始：`scripts/`、`references/`、`assets/` 文件。

注意：部分资源需要用户提供原始素材，例如品牌规范 Skill 需要用户提供 logo 和色号。

删除 `init_skill.py` 生成的、实际用不到的示例文件。

#### 4.2 编写 SKILL.md 正文

##### 核心写作原则：解释「为什么」，而不是列规则

写 SKILL.md 时，优先解释原因，而不是堆砌禁令清单。Agent 足够聪明，理解了背后的道理后能举一反三；只给规则，它只会机械执行，遇到规则没覆盖到的情况就会犯错。

检验方法：如果你发现自己写了「必须」「始终」「禁止」，停下来问——**这条规则背后的道理是什么？** 能讲清楚的，就用原因替换规则；讲不清楚的，说明这条规则本身可能不必要。

##### 意外最小化原则

Skill 应该做用户期望它做的事，不要过度推断或「聪明过头」。遇到歧义，优先选择最直接、最保守的解读，而非最「有创意」的解读。如果某个场景下无法确定该做什么，就停下来问用户，而不是自行猜测。

```
✅ 用户说「整理一下这份数据」→ 询问具体需求，或只做最小化操作（如去重、排序）
❌ 自动假设用户想要可视化图表或完整的数据分析报告
```

##### 安全原则：不出乎意料

Skill 的内容必须与其 description 所描述的意图完全一致，不能包含：
- 恶意代码或利用系统漏洞的操作
- 未经授权的数据外泄或访问行为
- 超出用户预期范围的系统操作

**一句话检验**：「如果用户看到这个 SKILL.md 的全部内容，他会感到意外吗？」若答案是会，就需要修改。

##### 精简优先

SKILL.md 中每一行指令都应该能提升输出质量。无效的指令会分散 Agent 对核心任务的注意力。

建议：写完初稿后，逐行问「删掉这句话，输出质量会变差吗？」不会变差的，就删掉。中文 SKILL.md 建议控制在 **500 行以内**——比「越全面越好」更重要的是「每一字都在发力」。

**中文写作语气**：使用**动词祈使句**，不使用第二人称或敬语。原因是 Skill 是写给 Agent 实例的指令，祈使句更直接，避免「你应该…」「请您…」这类冗余的客套。

##### 输出格式的定义方式

如果 Skill 需要 Agent 产出固定结构的内容，在 SKILL.md 中直接给出模板，不要只用文字描述。Agent 按模板执行，比按描述推断要准确得多。

##### description 字段写法

description 是决定 Agent 何时触发这个 Skill 的唯一信号，值得认真对待。

**写法要比你直觉认为的更「主动」。** 不只列关键词，还要覆盖用户不用「正式名称」、用口语、甚至用英文触发的场景。

建议中英双语写 description，原因是用户输入经常中英混用，纯中文 description 可能在用户用英文触发时失效。**建议总长度 ≤ 500 字符**，过长会增加每次触发判断的 token 消耗。

对比示例：

```
❌ 弱 description（只有关键词，覆盖不足）：
「飞书多维表格查询工具，用于查询用户数据」

✅ 强 description（覆盖口语和隐含需求）：
「查询飞书多维表格中的业务数据。当用户想知道某项数字、
 问『有多少』『有几个』『今天/本周的 X 是多少』，
 或提到飞书、多维表格、数据看板时，都应触发此 Skill——
 即使用户没有明确说「查询」或「飞书」。
 Query Feishu spreadsheet data — trigger whenever the user
 asks about metrics, counts, or mentions Feishu/Bitable.」
```

**编写 SKILL.md 时回答以下三个问题：**
1. 这个 Skill 的目的是什么？（2–3 句话）
2. 什么情况下应触发这个 Skill？（包括用户不一定会明说但明显需要的场景）
3. 实际操作中，Agent 应如何使用此 Skill？（明确引用所有附属资源）

### 第五步：测试与评估

编写完 Skill 后，通过运行测试用例来验证它是否真正有效。核心方法是**对比**——用同一道题分别在「有 Skill」和「无 Skill」两种条件下运行，才能看出 Skill 实际带来了什么改变。

不需要跑完整评估的情况：如果 Skill 的输出高度主观（写作风格、创意质量），或用户说「直接改、不用评估」，可以跳过本步骤，直接进入第六步凭直觉迭代。

#### 5.1 准备测试用例

根据第一步收集的使用场景，写出 2–3 个测试 prompt。

**什么是高质量的测试 prompt？** 写真实用户会说的话——越具体、越有背景越好。抽象的 prompt 测不出真实使用效果。

```
❌ 低质量（抽象）：
「帮我查一下数据」「整理这份文件」

✅ 高质量（具体、有背景、接近真实输入）：
「老板让我看看 Q4 的新用户留存，数据在飞书
 『2024Q4用户行为分析』的表里，我想知道 11 月和 12 月的对比」
「我下载了个 xlsx（应该在下载里，文件名像是 sales_final_v2
 还是 final_FINAL 啥的），需要在 D 列后面加一列利润率，
 收入在 C 列，成本在 E 列」
```

将测试用例保存到 `evals/evals.json`（此阶段暂不写 assertions，完整 Schema 见 `references/schemas.md`）：

```json
{"skill_name": "feishu-query", "evals": [
  {"id": 1, "prompt": "今天有多少新用户注册了？", "expected_output": "返回今日新注册用户数，说明数据来源字段"}
]}
```

#### 5.2 同时启动两组子 Agent 运行

**重要：必须在同一轮操作中同时启动「有 Skill」和「无 Skill（或旧版 Skill）」两组，不要先跑完一组再启动另一组。** 

结果保存到 `<skill-name>-workspace/iteration-1/` 目录，每个测试用例一个子目录，目录名使用描述性名称（如 `eval-query-new-users/`）。

**有 Skill 的运行：**
```
执行以下任务：
- Skill 路径：<当前 skill 目录路径>
- 任务：<eval prompt>
- 输入文件：<eval files，如无填 none>
- 将输出保存到：<workspace>/iteration-1/eval-<ID>/with_skill/outputs/
```

**无 Skill 的基准线（新建 Skill 时）：**
```
执行以下任务（不使用任何 Skill）：
- 任务：<eval prompt>（与上方完全相同）
- 将输出保存到：<workspace>/iteration-1/eval-<ID>/without_skill/outputs/
```

**迭代改进时**，将「无 Skill」替换为「旧版 Skill」作为基准线（运行前先将旧版复制到 `<workspace>/skill-snapshot/`，子 Agent 指向 snapshot）。

同时为每个测试用例创建 `eval_metadata.json`（assertions 先留空）：
`{"eval_id": 1, "eval_name": "query-new-users-today", "prompt": "今天有多少新用户注册了？", "assertions": []}`

#### 5.3 捕获运行耗时数据

每个子 Agent 完成时立即将通知中的 `total_tokens` 和 `duration_ms` 保存到 `timing.json`——**这是唯一的捕获时机**，逐条处理不要批量：
`{"total_tokens": 84852, "duration_ms": 23332, "total_duration_seconds": 23.3}`

#### 5.4 等待运行期间，起草量化断言

不要等子 Agent 完成后再思考如何评估——利用等待时间起草 assertions，并向用户解释每条 assertion 在检验什么。好的 assertion 要可以客观验证，名称要一眼看出它在检验什么：

```json
"assertions": [
  {
    "name": "返回了具体数字而非模糊描述",
    "check": "输出中包含具体的用户数量数字"
  },
  {
    "name": "引用了正确的飞书字段名",
    "check": "输出提到了 references/schema.md 中定义的字段名"
  }
]
```

主观类 Skill（写作风格、创意质量）不适合量化断言，优先依赖人工评审。

将更新后的 assertions 写入 `evals/evals.json` 和各 `eval_metadata.json`。

#### 5.5 评分、聚合、解读、查看结果

子 Agent 全部完成后，按顺序执行：

**1. 让 grader agent 评分**——参考 `agents/grader-zh.md`，对每个 assertion 判断 pass/fail，将结果保存到各 eval 目录的 `grading.json`（字段名用 `text`/`passed`/`evidence`，eval-viewer 依赖这些字段名）。能用脚本验证的断言，优先写脚本检验，比人工查看更快更可靠。

**2. 聚合 benchmark：**
```bash
python -m scripts.aggregate_benchmark <workspace>/iteration-1 --skill-name <name>
```
生成 `benchmark.json` 和 `benchmark.md`，包含 pass rate、耗时、token 用量。

**3. 解读 benchmark 数据**——聚合后先做一次分析，再开评审界面。特别留意以下信号：

| 信号 | 含义 | 应对 |
|------|------|------|
| 某断言在「有Skill」和「无Skill」两组通过率相差不大 | 这条断言没有区分价值 | 删除或改写这条断言 |
| 某 eval 两次运行结果差异很大 | Skill 指令不稳定，或 prompt 太模糊 | 检查 SKILL.md 是否有矛盾指令 |
| 有 Skill 时耗时/token 大幅增加，但质量提升不明显 | Skill 在引导 Agent 做无效步骤 | 读完整 transcript（不只是最终输出）定位问题所在 |

**4. 启动 eval-viewer：**

**本地 Agent Code / Cowork 环境：**
```bash
nohup python eval-viewer/generate_review.py \
  <workspace>/iteration-1 \
  --skill-name "my-skill" \
  --benchmark <workspace>/iteration-1/benchmark.json \
  > /dev/null 2>&1 &
VIEWER_PID=$!
```

迭代第 2 轮起，加 `--previous-workspace <workspace>/iteration-1`，可三列对比：新版 / 旧版 / 无 Skill。

**Claude.ai 环境（无法启动本地服务器）**：加 `--static /tmp/review-my-skill.html` 生成独立 HTML 文件，下载后在本地浏览器打开；用户点击「Submit All Reviews」后下载 `feedback.json`，放回 workspace 目录供下一步读取：
```bash
python eval-viewer/generate_review.py \
  <workspace>/iteration-1 \
  --skill-name "my-skill" \
  --benchmark <workspace>/iteration-1/benchmark.json \
  --static /tmp/review-my-skill.html
```

**5. 告知用户**：「已在浏览器中打开评审界面，有 Outputs（逐条对比两组输出，可写文字反馈）和 Benchmark（量化通过率、耗时、token 对比）两个 Tab。查看完成后点击『Submit All Reviews』，然后回来告诉我。」

**6. 读取反馈**：用户完成后，读取 `feedback.json`。空反馈 = 该项没问题；有文字 = 该项需要改进。关闭 viewer：`kill $VIEWER_PID`（Claude.ai 环境跳过此步）。

---

### 第六步：根据反馈改写 Skill

这是整个循环的核心。有反馈后，按以下原则改写：

**泛化，不要过拟合**

测试用例只是样本，Skill 最终要在无数种真实 prompt 下工作。与其针对每个测试用例加专门的规则，不如找到背后的共同原因，用一个更通用的解释覆盖它们。

**保持精简**

删除没有实际效果的指令。通读运行 transcript（不只是最终输出）——如果 Agent 在反复做无效的中间步骤，说明 SKILL.md 里有什么在误导它，找到那个地方并修改或删除。

**提取重复操作**

如果多个测试用例的 transcript 里都独立完成了相同的操作，这是应该将其提取为 `scripts/` 或 `references/` 的强信号——写一次，存进 Skill，所有后续运行直接复用。

**需要更严格对比时**

如果你或用户想确认「新版是否真的比旧版好」而不只靠自己判断，可以使用盲测：让独立的 Agent 在不知道哪个是新版的情况下对两组输出打分，消除主观偏见。参考 `agents/comparator-zh.md` 和 `agents/analyzer-zh.md`。

#### 迭代循环

改写完成后：

1. 将改写前的 Skill 复制到 `<workspace>/skill-snapshot/` 作为新一轮基准线
2. 在新目录 `iteration-2/` 下重新运行所有测试用例（同时启动新版 vs 旧版两组）
3. 启动 eval-viewer 时加 `--previous-workspace <workspace>/iteration-1`
4. 读取新一轮反馈，继续改写

**何时停止迭代？** 满足以下任一条件即可停止：
- 用户表示满意
- 所有测试用例的反馈均为空
- 连续两轮改动后，benchmark 通过率提升幅度低于 5%
- SKILL.md 的改动越来越小，说明已接近收敛

---

### 第七步：打包 Skill

**发布前自检（7 条）：**
```
□ SKILL.md < 500 行
□ description 中英双语，覆盖口语触发词
□ name 字段为 kebab-case 英文（如 feishu-query）
□ 每个 references/ 文件在 SKILL.md 中有明确的读取时机说明
□ 删除了所有未使用的空目录或示例文件
□ 删除了所有「请您」「你应该」等冗余措辞
□ 用至少 1 个真实 prompt 手动测过
```

```bash
scripts/package_skill.py <skill 目录路径> [./dist]  # 输出目录可选
```

---

## 进阶：触发描述优化

> **何时使用**：当 Skill 频繁被漏触或误触时才需要此循环。正常情况下，第四步的 description 写法已足够；只有在 Skill 基本可用后需要精调触发精度时，才运行以下优化流程。

### 生成触发测试集

创建 20 个测试 query，一半应触发、一半不该触发，保存为 JSON：

```json
[
  {"query": "帮我把这份销售数据整理成飞书多维表格，按月份分组", "should_trigger": true},
  {"query": "帮我写一首关于秋天的诗", "should_trigger": false}
]
```

**写好 should-trigger query 的关键**：用真实用户的口吻——加入具体文件名、业务背景、口语表达、中英混用。「帮我查一下数据」太模糊；「老板让我看看 Q4 的新用户留存，数据在飞书那张表里」才真实。覆盖不同措辞方式：有人说「查一下」，有人说「看看」，有人用英文说「check」——都要能触发。

**写好 should-not-trigger query 的关键**：最有价值的是「近似但不该触发」的 query——它们和 Skill 的领域相关，但实际上需要其他工具。纯粹无关的反例（如「写一首诗」）没有测试意义。例如，一个飞书查询 Skill 的 should-not-trigger 反例应该是「帮我在飞书上创建一个新的多维表格」（创建而非查询）。

在 `assets/eval_review.html` 中审核测试集（用浏览器打开，可直接修改、删除、导出），确认 query 质量后再运行优化。

### 运行优化循环

```bash
python -m scripts.run_loop \
  --eval-set <触发测试集路径> \
  --skill-path <skill 目录路径> \
  --model <当前使用的模型 ID> \
  --max-iterations 5 \
  --verbose
```

脚本自动迭代优化 description，最多运行 5 轮，输出每轮的触发准确率变化。完成后将优化后的 description 更新回 SKILL.md。

---

## 常见错误与解决方案

| 错误 | 原因 | 解决方案 |
|------|------|---------|
| Skill 未被触发 | description 触发词覆盖不足 | 在 description 中增加用户实际使用的中文短语；覆盖口语、英文混用、不说「正式名称」的场景 |
| SKILL.md 过长 | 细节信息堆积在主文件 | 将详细参考资料移至 references/ 目录 |
| 信息重复 | SKILL.md 和 references 均有相同内容 | 同一内容只保留一处 |
| name 字段含中文 | 系统在解析路径和脚本调用时依赖纯 ASCII 字符 | name 使用 kebab-case 英文（如 `feishu-query`） |
| Skill 被频繁误触 | description 过于宽泛，覆盖了不相关场景 | 在 description 中明确「不适用的场景」，或使用进阶触发优化 |
| 输出质量不稳定 | SKILL.md 存在相互矛盾的指令 | 通读全文，删除冲突要求；优先保留「解释原因」的描述 |
| 测试时质量好，真实使用时变差 | Skill 过拟合了测试用例 | 检查是否有针对特定 eval 加的专项规则，将其替换为通用化描述 |
| Skill 该问时直接执行 | 缺少「遇到歧义停下来问」的指令 | 在 SKILL.md 中明确「遇到以下情况，先询问用户而不是自行推断」 |
