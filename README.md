# Personal Agent Skills Library

Portable AI-agent skills for repeatable writing, review, planning, prompting, fact-checking, and cover-design workflows.

## Skill Set

- `article-review-en` / `article-review-ru`
- `article-writing-en` / `article-writing-ru`
- `bitrix24-development-en` / `bitrix24-development-ru`
- `cover-design-en` / `cover-design-ru`
- `fact-checking-en` / `fact-checking-ru`
- `habr-style-en` / `habr-style-ru`
- `prompt-engineering-en` / `prompt-engineering-ru`
- `software-project-planning-en` / `software-project-planning-ru`
- `technical-explainer-en` / `technical-explainer-ru`

Exact source dialogue is preserved in `transcripts/`.

## Structure

```text
skills/
├── <skill-name-en>/
│   └── SKILL.md
├── <skill-name-ru>/
│   └── SKILL.md
├── cover-design-*/
│   ├── SKILL.md
│   ├── references/
│   └── examples/
└── bitrix24-development-*/
    ├── SKILL.md
    └── references/
```

## Setup

Codex:

```bash
mkdir -p ~/.agents
ln -s /PATH/TO/REPOSITORY/skills ~/.agents/skills
```

Claude Code:

```bash
mkdir -p ~/.claude
ln -s /PATH/TO/REPOSITORY/skills ~/.claude/skills
```

OpenCode and other agents can read the same `SKILL.md` files directly or through their own skills directory.

## Repository Description

Short:

Portable AI-agent skills for reusable writing, planning, prompting, and cover-design workflows.

Longer:

A portable library of AI-agent skills for repeatable creative and technical workflows: article writing, editorial review, Habr-style publishing, prompt engineering, software project planning, technical explanations, fact-checking, Bitrix24 application development, and cover design in English and Russian.
