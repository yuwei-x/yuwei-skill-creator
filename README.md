# yuwei-skill-creator

> 帮你把任何工作流打包成可复用的 AI Agent Skill 的完整工作台。

---

## 这是什么

`yuwei-skill-creator` 是一个 AI Agent Skill，安装后当你对 Claude Code（或其他支持 Skill 机制的 Agent）说「我想创建一个 Skill」，它就会自动被激活，引导你完成从零到发布的完整流程：

- **理解需求** — 通过具体例子弄清楚这个 Skill 要做什么
- **规划资源** — 识别哪些内容可以复用，应该放进 `scripts/`、`references/` 还是 `assets/`
- **编写内容** — 写出高质量的 `SKILL.md`，包括触发描述、工作流指令、输出模板
- **测试评估** — 用真实 prompt 跑测试，对比「有 Skill / 无 Skill」两种条件下的输出差异
- **迭代优化** — 根据评审反馈改写，支持多轮对比
- **打包发布** — 生成可分发的 `.skill` 文件

---

## 安装方法

```bash
git clone https://github.com/yuwei-x/yuwei-skill-creator ~/.claude/skills/yuwei-skill-creator
````

克隆完成后重启 Claude Code，即可生效。

---

## 触发方式

安装后，以下说法都会触发这个 Skill：

- 「我想创建一个 Skill」
- 「帮我把这个工作流打包成可复用的技能」
- 「上次那个流程能不能固定下来，以后让 Agent 自动做」
- 「我每次都要手动做这件事，能不能让 Agent 记住」
- "I want to build a skill for my agent"

---

## 目录结构

```
yuwei-skill-creator/
├── SKILL.md                      # 主指令文件（触发后加载）
├── agents/
│   ├── grader-zh.md              # 评分 Agent：对每条断言判断 pass/fail
│   ├── comparator-zh.md          # 盲测比较 Agent：不知道哪组是新版的情况下打分
│   └── analyzer-zh.md            # 分析 Agent：找出胜出原因，输出改进建议
├── scripts/
│   ├── run_eval.py               # 运行触发评估（测试 description 的触发准确率）
│   ├── improve_description.py    # 调用 Claude 优化 description
│   ├── run_loop.py               # 自动循环：评估 → 优化 → 再评估
│   ├── aggregate_benchmark.py    # 聚合多轮测试结果，生成 benchmark.json
│   ├── generate_report.py        # 生成可视化 HTML 报告
│   ├── package_skill.py          # 打包 Skill 为 .skill 文件
│   ├── quick_validate.py         # 快速校验 SKILL.md 格式是否合规
│   └── utils.py                  # 公共工具函数
├── references/
│   └── schemas.md                # 所有 JSON 文件的字段定义（evals、grading、benchmark 等）
├── assets/
│   └── eval_review.html          # 触发测试集可视化审核界面
└── eval-viewer/
    ├── generate_review.py        # 启动评审服务器 / 生成静态 HTML
    └── viewer.html               # 评审界面模板（含 Outputs + Benchmark 两个 Tab）
```

---

## 运行环境要求

`scripts/` 下的脚本依赖：

```bash
pip install anthropic pyyaml
```

`run_eval.py` 和 `run_loop.py` 依赖 `claude` CLI（即 Claude Code）。如使用其他 Agent，需将脚本中的 `claude -p` 替换为对应命令。

---

## 其他平台安装路径

| AI Agent | Skill 目录 |
|----------|-----------|
| Claude Code（全局）| `~/.claude/skills/` |
| Claude Code（项目级）| `<项目根目录>/.claude/skills/` |
| CodeX | `~/.codex/skills/`（以官方文档为准）|
| OpenClaw | `~/.openclaw/skills/`（以官方文档为准）|

核心文件（`SKILL.md`、`scripts/`、`agents/` 等）在所有平台上完全相同，仅安装目录不同。

---

## 创建出来的 Skill 结构

用这个 Skill 创建出的新 Skill，标准结构如下：

```
your-skill-name/
├── SKILL.md          # 必须，含 name + description 元数据和正文指令
├── scripts/          # 可选，可执行脚本
├── references/       # 可选，按需加载的参考文档
└── assets/           # 可选，模板、图标等产出物资源
```

---

## 许可证

本项目采用 [CC BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/) 许可证。

个人使用、学习、研究、非商业项目：不需要署名，不需要申请
公开发布衍生作品（文章、工具、课程等）：请注明来源
商业用途：需要单独授权，请联系作者

---

## 作者

由 [@yuwei-x](https://github.com/yuwei-x) 创建。
如有问题或建议，欢迎提 Issue 或 PR。
