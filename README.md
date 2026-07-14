# Personal Agent Skills

Portable AI-agent skills for cover design.

## Structure

```text
skills/
├── cover-design-en/
│   ├── SKILL.md
│   ├── references/
│   │   ├── prompt-template.md
│   │   └── dialogue-context.md
│   └── examples/
│       └── 90s-office-cover.md
└── cover-design-ru/
    ├── SKILL.md
    ├── references/
    │   ├── prompt-template.md
    │   └── dialogue-context.md
    └── examples/
        └── 90s-office-cover.md
```

## Setup

Codex:

```bash
ln -s /PATH/TO/REPOSITORY/skills/cover-design-en ~/.codex/skills/cover-design-en
ln -s /PATH/TO/REPOSITORY/skills/cover-design-ru ~/.codex/skills/cover-design-ru
```

Claude Code:

```bash
ln -s /PATH/TO/REPOSITORY/skills/cover-design-en ~/.claude/skills/cover-design-en
ln -s /PATH/TO/REPOSITORY/skills/cover-design-ru ~/.claude/skills/cover-design-ru
```

OpenCode and other agents can read the same `SKILL.md` files directly or through their own skills directory.
