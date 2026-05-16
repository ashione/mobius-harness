# Skills Hub

可复用的 Skill Suite 仓库，统一维护技能提示词，软链接到任意 Agent 平台（Claude / Codex / OpenClaw）。

## Mobius Harness

`mobius-harness` 是本仓库的端到端交付主入口。用户可以用它驱动同一个 agent 完成完整软件交付流程：

1. 需求分析
2. 交付计划
3. 本地 worktree 开发
4. 实现与本地验证
5. PR/MR 创建与 CI/CD 跟踪
6. 交付报告

其他 skills 是 Mobius Harness 可按需调用的专业能力，例如：

- `local-repo-development`：本地仓库、worktree、提交前 review、敏感信息扫描、CI/CD 跟踪
- `refactor-planner`：低风险重构计划
- `api-design-review`：API 设计审查
- `test-case-generator`：测试场景生成
- `frontend-ux-polish`：前端体验打磨
- `bug-triage`：bug 分级与复现计划
- `commit-message-writer`：提交信息生成

长任务默认可在本地维护 Delivery Episode Package：

```text
.delivery/runs/<run-id>/
  requirements.md
  plan.md
  verification.md
  delivery-report.md
```

`.delivery/runs/` 默认不提交到 git。

交付过程必须遵循 Mobius Harness 的阶段门禁。任务开始时选择 `Lightweight`、`Standard` 或 `Strict` 模式；大任务可拆成子阶段。每个阶段和子阶段都必须记录 Goal、Checklist、Todo List、Failure List 和 Change List。任何 complete 状态都必须有 evidence。交付产物必须遵循 [docs/HARNESS.md](docs/HARNESS.md) 中的 artifact 标准。

如果交付被中断，后续 agent 应先读取 `.delivery/runs/<run-id>/`，找到最早未完成的阶段或子阶段，再基于 Todo List、Failure List、Change List 和当前 git 状态继续执行。

## 仓库结构

```text
.
├── docs/
│   ├── HARNESS.md
│   └── SKILL_SPEC.md
├── scripts/
│   ├── create-skill.sh
│   ├── link_skills.sh
│   └── validate-skills.sh
├── skills/
│   ├── mobius-harness/
│   │   ├── SKILL.md
│   │   ├── skill.json
│   │   └── references/
│   │       ├── delivery-process.md
│   │       ├── artifact-interface.md
│   │       ├── artifact-templates.md
│   │       └── governance-and-reporting.md
│   ├── local-repo-development/
│   │   ├── SKILL.md
│   │   └── skill.json
│   └── ...
└── README.md
```

## 使用

软链接到目标平台的 skill 目录：

```bash
bash scripts/link_skills.sh /path/to/claude/skills /path/to/codex/skills /path/to/openclaw/skills
```

脚本会把 `skills/` 下每个 skill 目录软链接过去，平台直接读取 `SKILL.md` 和 `skill.json`。

## Codex 兼容性

如果目标平台是本地 Codex，`SKILL.md` 顶部需要带 YAML frontmatter，至少包含：

```md
---
name: my-skill
description: One-line description.
---
```

- `name` 建议与 `skill.json.id` 和目录名保持一致。
- `description` 建议与 `skill.json.description` 保持一致。
- 仅有 `skill.json` 不足以保证 Codex 发现该 skill；Codex 本地发现逻辑会读取 `SKILL.md` 元数据。

本仓库内置的 `create-skill.sh` 已默认生成兼容 frontmatter。

## 新增 Skill

```bash
bash scripts/create-skill.sh my-new-skill
```

脚本会创建 `skills/my-new-skill/` 并生成 SKILL.md 和 skill.json 模板，填写 TODO 即可。

手动新增也可以：

1. 在 `skills/` 下新增目录，例如 `my-new-skill/`。
2. 新增 `SKILL.md`（人类可读标准文档）。
3. 新增 `skill.json`（机器可读结构化数据）。
4. 参考 [docs/SKILL_SPEC.md](docs/SKILL_SPEC.md) 填写字段。

## CI 验证

每次 push 和 PR 会自动检查：

- 每个 skill 目录包含 SKILL.md 和 skill.json。
- skill.json 语法正确且包含全部必需字段。
- skill.json 的 id 与目录名一致。
- `SKILL.md` frontmatter 中的 `name` / `description` 与 skill 元数据一致。

本地也可以先跑：

```bash
bash scripts/validate-skills.sh
```
