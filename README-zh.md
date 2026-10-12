# MathModeling Skills · 数学建模工作流技能库

[English](README.md) · [项目规则](AGENTS.md) · [中文启动提示词](Initial%20Prompt-zh.md)

由 [hughrao](https://github.com/hughrao) 维护的数学建模工作流技能库，包含 **28 个技能**，支持 Claude Code 和 Codex。覆盖题目解析、数据审查、方法筛选、可复现实验、图表验证、论文材料交接和终稿审计。

核心原则：**AI 负责机械正确性，人负责建模判断。** AI 整理数据、检查方法风险、实现已批准的方法、复核代码和证据；使用者决定目标与取舍、选择方法、解释结果并确认最终主张。

## 项目提供什么

- **问题与数据准备**：解析目标、对象、约束和输出，分别分类子问题，维护符号表和显式假设，保留原始数据。
- **方法筛选**：一个主候选、一个能够完成实际任务的基线，最多一个有明确触发条件的备用方法。
- **风险探针**：检查可执行性、数据覆盖、关键假设、输出退化、微扰敏感性和代表性规模下的耗时与内存。
- **代码与验证**：支持 Python 和 MATLAB；MATLAB 路线包含北太天元兼容性约束，实际兼容性仍需在目标运行时验证。
- **结果与论文交接**：分析实验、检查稳健性、规划图表、验证渲染并构建论文材料包。
- **数值与终稿检查**：冻结论文数值，执行一致性、完整性和质量保证三类终稿审计。

仓库本身包含技能指令、模板、说明文档和安装脚本。`examples/` 当前是预测、优化、评价和混合问题的场景提纲，不是完整解题案例，也没有可用于性能比较的实测榜单。

## 安装

先克隆，再预览安装操作：

```bash
git clone https://github.com/hughrao/MathModeling-skills.git
cd MathModeling-skills
bash install.sh --dry-run
bash install.sh
```

默认以原生插件方式安装到 Claude Code 和 Codex，使用当前克隆目录作为本地 marketplace。对应宿主 CLI 需要已安装，并支持脚本使用的插件命令。Windows 下请在 Git Bash 或 WSL 中执行；安装后保留克隆目录，并开启新的宿主会话。

仅安装到一个宿主：

```bash
bash install.sh --target claude
bash install.sh --target codex
```

若希望把完整技能目录和项目规则直接放进已有的竞赛工作区：

```bash
bash install.sh --mode project --target both --project-dir /path/to/contest
```

项目模式不需要调用宿主插件命令，会复制所选技能目录和 `AGENTS.md`；Claude 部署还包含 `CLAUDE.md` 和 `.claude/settings.json`。遇到内容不同的现有文件会停止，使用 `--force` 可先备份再替换，使用 `--dry-run` 可预览。Claude 插件模式支持 `--scope user|project|local`，其中 `project` 和 `local` 必须明确提供 `--project-dir`。全部参数见 `bash install.sh --help`。

## 开始使用

提供题目、附件、可用运行环境和竞赛约束，然后发送 [中文启动提示词](Initial%20Prompt-zh.md)。也可以从以下指令开始：

```text
请阅读 AGENTS.md 和当前阶段相关的 SKILL.md，用 workflow-orchestrator
检查工作区。新题先从 problem-parser 开始。方法筛选前，请让我说明
输出形式、优先级、不可接受的失败和实验预算；不要替我编造方法选择
理由或结果解释。
```

新工作区默认在 `planning/session_config.json` 使用：

```json
{
  "interaction_mode": "learning",
  "rigor_profile": "lean"
}
```

| 配置 | 作用 |
| --- | --- |
| `learning` / `speed` | 调整提问密度和建议呈现时机。 |
| `lean` / `submission` | 调整产物和审计密度；探索用 lean，论文交接和终稿用 submission。 |

两种配置相互独立，都不改变“人负责建模判断”的边界。

## 工作流关卡

| 关卡 | 通过条件概要 |
| --- | --- |
| G1：问题已明确 | 题目解析、分类、数据清单、成功标准和使用者的目标取舍已明确。 |
| G2：方法已筛选 | 主候选与可用基线经过风险探针；备用方法有明确触发条件。 |
| G2.5：使用者已选方法 | 决策账本记录使用者选择与理由，并引用风险探针证据。 |
| G3：代码与实验已审查 | 已批准的方法实际运行，摘要与审查覆盖语法、输入、方法一致性、复现和输出。 |
| G4：结果已判断与冻结 | 使用者对结果、稳定性和主张范围作出证据关联判断；submission 还要求最终报告、材料包和有效冻结数值。 |
| G5：论文分节就绪 | 从材料包和冻结数值写作，解释经人确认，论文图表通过渲染验证。 |
| G6：终稿审计通过 | 仅在 submission 下执行，一致性、完整性、质量保证三类审计全部通过后才能最终组装。 |

关卡由智能体根据指令和文件证据检查，项目不是独立运行的工作流引擎。项目部署会添加仓库提供的 Claude 权限与 hook，插件部署提供技能及打包规则。完整契约以 [AGENTS.md](AGENTS.md) 为准。

## 28 个技能

| 阶段 | 技能 |
| --- | --- |
| 调度、问题与数据准备 | `workflow-orchestrator`、`problem-parser`、`problem-classifier`、`related-paper-analyzer`、`data-auditor-cleaner`、`symbol-table-builder`、`model-assumptions-builder` |
| 方法筛选与人类决策 | `method-selector`、`decision-prompt-builder`、`modeler-decision-logger` |
| 实现与代码审查 | `model-code-analyzer`、`python-model-code-generator`、`matlab-model-code-generator`、`code-reviewer`、`python-code-reviewer`、`matlab-code-reviewer` |
| 结果、图表与材料交接 | `result-report-generator`、`robustness-checker`、`final-method-explainer`、`figure-table-planner`、`math-figure-generator`、`solution-package-builder` |
| 论文与终稿审计 | `paper-section-writer`、`paper-polisher`、`reference-manager`、`consistency-auditor`、`completeness-auditor`、`quality-assurance-auditor` |

## 仓库结构

```text
MathModeling-skills/
├── .codex/skills/                 # 可独立使用的 Codex 技能副本
├── .claude/skills/                # 可独立使用的 Claude 技能副本
├── .claude/settings.json          # 项目权限与启动 hook
├── .agents/plugins/marketplace.json
├── .claude-plugin/marketplace.json
├── plugins/mathmodeling-skills/   # 两个宿主共用的原生插件分发包
├── docs/                         # 运行环境说明与资源
├── skills-docs/                  # 辅助工作流文档
├── templates/                    # Schema 与产物模板
├── examples/                     # 问题场景提纲
├── scripts/sync-plugin.sh        # 分发包同步与检查
├── AGENTS.md                     # 权威工作流规则
├── CLAUDE.md                     # Claude 专属运行约定
└── install.sh                    # 插件/项目安装脚本
```

使用技能进行建模时，竞赛工作区的产物结构如下；它与技能库目录有不同用途：

```text
contest/
├── planning/                     # 配置、manifest、目标决策、符号与假设
├── methods/Qx/                   # 方法卡、决策账本与风险探针
├── code/Qx/                      # Python 实现与审查证据
├── code/matlab/Qx/               # MATLAB 实现
├── results/Qx/experiments/roundN/ # figures、tables、metrics、run_summary.json
├── results/Qx/reports/           # 最终分析、论文材料包、冻结数值
├── robustness/Qx/                # 稳健性证据
├── paper/                        # 分节、已验证图表、参考文献与审计
├── workspace/data_raw/           # 原始数据，保持不变
├── workspace/data_clean/         # 清洗后的副本
└── scratch/                      # 临时探索
```

使用者决策追加记录到 `methods/Qx/qx_decisions.jsonl`，子问题目录建立前的全局目标决策使用 `planning/framing_decisions.jsonl`。数值按 **代码 → 结果 → 冻结 → 论文** 流转。需要修改冻结数值时，记录解冻原因、修改 canonical source、重跑受影响工作、重新冻结，不能手工改冻结快照。

## 更新与维护

```bash
git pull --ff-only
bash install.sh
```

维护共享技能时，同时修改 `.codex/skills/` 和 `.claude/skills/`，保证两份都能独立使用，然后更新并检查分发包：

```bash
bash scripts/sync-plugin.sh
bash scripts/sync-plugin.sh --check
```

同步工具需要 Bash、Python 3、`diff`、`cmp`，刷新操作还需要 `rsync`。发布时两个插件 manifest 和 Claude marketplace 的版本必须一致。贡献要求见 [CONTRIBUTING.md](CONTRIBUTING.md)，问题和建议可提交到 [Issues](https://github.com/hughrao/MathModeling-skills/issues)。

## 相关文档

- [项目规则](AGENTS.md)与 [Claude 运行约定](CLAUDE.md)
- [中文启动提示词](Initial%20Prompt-zh.md)与[英文启动提示词](Initial%20Prompt.md)
- [实现语言与产物约定](docs/implementation-targets.md)
- [MATLAB / 北太天元兼容性说明](docs/matlab-beita-tianyuan-guidelines.md)
- [Codex 技能目录](.codex/skills/)与 [Claude 技能目录](.claude/skills/)

## 维护者、版权与许可证

维护者：**hughrao**。本版本修改及新增文档版权：Copyright © 2026 hughrao。

项目采用 [MIT License](LICENSE)。本版本基于 [Zhijun Zhang 的 MathModeling-skills](https://github.com/zhnnky329/MathModeling-skills)，保留其原始版权声明。图表指导亦致谢 [nature-skills](https://github.com/Yuan1z0825/nature-skills) 和 [figures4papers](https://github.com/ChenLiu-1996/figures4papers)。

请按具体竞赛的 AI 使用与披露规则使用本项目；工作流本身不构成参赛合规证明，使用者仍需对提交内容负责。
