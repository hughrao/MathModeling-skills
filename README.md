# MathModeling Skills

[简体中文](README-zh.md) · [Workflow policy](AGENTS.md) · [Getting started prompt](Initial%20Prompt.md)

A collection of **28 mathematical-modeling workflow skills** for Claude Code and Codex, maintained by [hughrao](https://github.com/hughrao). It helps a modeling team move from understanding a contest problem to reproducible experiments, verified figures, and evidence-backed paper sections.

**The AI owns mechanical correctness; the human owns modeling judgment.** The skills organize data, screen method risks, implement approved methods, review code, and check evidence. The modeler chooses trade-offs, approves methods, interprets results, and authorizes final claims.

## What is included

- Problem parsing, task classification, data auditing, shared symbols, and explicit assumptions.
- A role-based method shortlist: one main candidate, one usable baseline, and at most one fallback with a recorded activation trigger.
- Method-specific risk probes for executability, data coverage, assumptions, output degeneracy, perturbation sensitivity, and scale.
- Python and MATLAB implementation and review; MATLAB workflows also address 北太天元 compatibility constraints.
- Result analysis, robustness checks, figure planning and render verification, and a writer handoff package.
- Frozen numerical claims and three final audits: consistency, completeness, and quality assurance.

This repository contains skill instructions, templates, documentation, and installer scripts. The folders under `examples/` are scenario outlines, not completed solutions or benchmark results.

## Installation

Clone the repository and run the Bash installer:

```bash
git clone https://github.com/hughrao/MathModeling-skills.git
cd MathModeling-skills
bash install.sh --dry-run
bash install.sh
```

The default mode registers a local marketplace and installs `mathmodeling-skills` for both Claude Code and Codex. The corresponding host CLI must be installed and support the plugin commands used by the installer. On Windows, run these commands in Git Bash or WSL. Keep the clone available as the local marketplace source and start a new host session after installation.

Install for one host:

```bash
bash install.sh --target claude
bash install.sh --target codex
```

To copy standalone skills and project rules into an existing contest workspace:

```bash
bash install.sh --mode project --target both --project-dir /path/to/contest
```

Project mode does not require host plugin commands. It copies the selected skill trees and `AGENTS.md`; Claude deployment also includes `CLAUDE.md` and `.claude/settings.json`. It stops on conflicting files. Use `--force` to back up conflicting destinations before replacement, and `--dry-run` to preview changes. Claude plugin installation supports `--scope user|project|local`; project and local scopes require an explicit `--project-dir`. See `bash install.sh --help` for all options.

## Start a modeling task

Supply the problem statement, attachments, available runtime, and relevant contest constraints. Then send the [initial prompt](Initial%20Prompt.md), or begin with:

```text
Read AGENTS.md and the relevant skills. Use workflow-orchestrator to inspect
this workspace. For a new problem, begin with problem-parser. Before method
screening, ask me about output form, priorities, unacceptable failures, and
experiment budget. Do not invent my method choice or interpretation.
```

For a new workspace, the default configuration in `planning/session_config.json` is:

```json
{
  "interaction_mode": "learning",
  "rigor_profile": "lean"
}
```

`learning` and `speed` control interaction density. `lean` and `submission` control artifact and audit density. Exploration uses `lean`; writer handoff and final assembly use `submission`. Neither setting transfers modeling judgment to the AI.

## Workflow and gates

| Gate | Required outcome |
| --- | --- |
| G1 — Problem framed | Parsed problem, classification, data inventory, success criteria, and human framing. |
| G2 — Method screened | Main candidate and usable baseline screened with evidence; any fallback has an explicit trigger. |
| G2.5 — Human method choice | A human-authored method choice and rationale cite the risk-probe evidence. |
| G3 — Code and experiment reviewed | Approved methods ran; run summary and review cover syntax, inputs, method alignment, reproducibility, and outputs. |
| G4 — Results judged and frozen | Evidence-linked human verdicts; submission mode additionally requires the final reports, writer package, and current frozen numbers. |
| G5 — Paper section ready | Drafts use the writer package, frozen numerical claims, human-confirmed interpretation, and verified figures. |
| G6 — Final audits passed | In submission mode, consistency, completeness, and quality assurance all pass before final assembly. |

These gates are instruction and evidence contracts evaluated by the agent, not a standalone workflow engine. Project deployment adds the supplied Claude permissions and hooks; plugin mode supplies skills and packaged policy. Read [AGENTS.md](AGENTS.md) for the authoritative contract.

## Skills by stage

| Stage | Skills |
| --- | --- |
| Coordination and framing | `workflow-orchestrator`, `problem-parser`, `problem-classifier`, `related-paper-analyzer`, `data-auditor-cleaner`, `symbol-table-builder`, `model-assumptions-builder` |
| Method screening and human decisions | `method-selector`, `decision-prompt-builder`, `modeler-decision-logger` |
| Implementation and review | `model-code-analyzer`, `python-model-code-generator`, `matlab-model-code-generator`, `code-reviewer`, `python-code-reviewer`, `matlab-code-reviewer` |
| Results and writer handoff | `result-report-generator`, `robustness-checker`, `final-method-explainer`, `figure-table-planner`, `math-figure-generator`, `solution-package-builder` |
| Paper and final audits | `paper-section-writer`, `paper-polisher`, `reference-manager`, `consistency-auditor`, `completeness-auditor`, `quality-assurance-auditor` |

## Repository layout

```text
MathModeling-skills/
├── .codex/skills/                 # Complete standalone Codex skill tree
├── .claude/skills/                # Complete standalone Claude skill tree
├── .claude/settings.json          # Project permissions and startup hook
├── .agents/plugins/marketplace.json
├── .claude-plugin/marketplace.json
├── plugins/mathmodeling-skills/   # Shared native plugin distribution
├── docs/                         # Runtime guidance and assets
├── skills-docs/                  # Supporting workflow documentation
├── templates/                    # Schemas and artifact templates
├── examples/                     # Scenario outlines
├── scripts/sync-plugin.sh        # Distribution synchronization/check
├── AGENTS.md                     # Authoritative workflow policy
├── CLAUDE.md                     # Claude-specific operating rules
└── install.sh                    # Plugin/project installer
```

A contest workspace created while using these skills is separate from the library:

```text
contest/
├── planning/                     # Configuration, manifests, framing, symbols
├── methods/Qx/                   # Method card, decision ledger, risk probes
├── code/Qx/                      # Python code and review evidence
├── code/matlab/Qx/               # MATLAB implementation
├── results/Qx/experiments/roundN/ # Figures, tables, metrics, run_summary.json
├── results/Qx/reports/           # Final reports, writer package, frozen numbers
├── robustness/Qx/                # Robustness evidence
├── paper/                        # Sections, verified figures, references, audits
├── workspace/data_raw/           # Original data: keep untouched
├── workspace/data_clean/         # Cleaned copies
└── scratch/                      # Temporary exploration
```

Human decisions are appended to `methods/Qx/qx_decisions.jsonl`, with pre-subquestion framing in `planning/framing_decisions.jsonl`. Numbers flow **code → results → freeze → paper**. Change a frozen value by logging a thaw, changing its canonical source, rerunning affected work, and freezing again; never edit the frozen snapshot by hand.

## Updates and contributions

```bash
git pull --ff-only
bash install.sh
```

Maintain `.codex/skills/` and `.claude/skills/` as complete, consistent copies. After changing shared skills, synchronize and verify the distribution:

```bash
bash scripts/sync-plugin.sh
bash scripts/sync-plugin.sh --check
```

Synchronization uses Bash, Python 3, `diff`, `cmp`, and `rsync` (`rsync` is needed for refresh). Plugin release versions must agree in both host manifests and the Claude marketplace entry. See [CONTRIBUTING.md](CONTRIBUTING.md). Report issues or propose changes on [GitHub](https://github.com/hughrao/MathModeling-skills/issues).

## Further reading

- [Chinese initial prompt](Initial%20Prompt-zh.md)
- [Implementation targets](docs/implementation-targets.md)
- [MATLAB / 北太天元 guidance](docs/matlab-beita-tianyuan-guidelines.md)
- [Standalone Codex skills](.codex/skills/) and [Claude skills](.claude/skills/)

## Authorship and license

Maintained by **hughrao**. Copyright © 2026 hughrao for modifications and new documentation in this edition.

Distributed under the [MIT License](LICENSE). This edition is based on [Zhijun Zhang's MathModeling-skills](https://github.com/zhnnky329/MathModeling-skills); the original copyright notice is retained. Figure guidance also acknowledges [nature-skills](https://github.com/Yuan1z0825/nature-skills) and [figures4papers](https://github.com/ChenLiu-1996/figures4papers).

Use the workflow within the applicable contest's AI-use and disclosure rules. It does not establish contest eligibility or replace the modeler's responsibility for the submitted work.
