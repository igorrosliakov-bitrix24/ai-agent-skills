# AGENTS.md

This repository is a personal library of portable AI-agent skills.

- Treat the repository as a growing system of repeatable workflows, not as one large prompt.
- Each skill lives in `skills/<name>/`.
- Each skill has a required `SKILL.md` with YAML frontmatter containing only `name` and `description`.
- Keep every skill available in two variants when practical: English `*-en` and Russian `*-ru`.
- Put large templates or background context in `references/`.
- Put completed examples in `examples/`.
- Preserve exact source dialogues in `transcripts/`; do not rewrite them.
- Prefer small, practical instructions over long general advice.
- When adding a skill, validate the YAML frontmatter before committing.
