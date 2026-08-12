---
name: software-project-planning-en
description: "Helps in English to create step-by-step software project plans with architecture, typing, linters, tests, checks, risks, and completion criteria. Use for English planning of apps, libraries, integrations, and MVPs."
---

# Software Project Planning

Use this skill when the user wants to plan a software project, MVP, integration, library, or technical prototype.

## Workflow

1. Identify the product goal, users, environment, constraints, and integrations.
2. Split work into verifiable stages with a concrete result for each stage.
3. For each stage, list files or modules, checks, tests, and completion criteria.
4. Separately flag risks: unknown APIs, security, data, migrations, and performance.
5. Do not start the next stage if the user asks for a step-by-step confirmation workflow.
6. End with the minimal command set needed to verify the project.

## Engineering Rules

- Choose a strictly typed language. When that is not possible, add explicit schemas or runtime type validators at system boundaries.
- Always include a linter and make linting part of the required checks.
- Require every code change to add or update tests and run all tests relevant to the change. When AI performs implementation work, state this requirement explicitly in every code-change prompt.
- Plan test coverage measurement. Set a threshold of at least 70% and target 80%; raise the threshold for critical logic according to risk.
- Validate all input and external data before use.
- Minimize the number of frameworks and dependencies.
- Plan regular dependency updates, outdated-package checks, and known-vulnerability audits.
- Do not put secrets in source code.
- Test behavior, not only the happy path.
- In explanations and examples, do not overuse lists: use at most three items by default. Longer lists are acceptable for stages, rules, checks, or completion criteria when the full set is genuinely needed.

## Learning Handoff After Practice

After completing a practical development, configuration, validation, or deployment stage, always add a separate Markdown blockquote starting with `> **💡` that explains one idea actually used in the work: a security practice, architecture pattern, testing approach, Git technique, or another useful principle. Preserve the `>` prefix on every line so the interface renders a vertical line on the left.

- Explain what it is, where it is used in the current project, and why it matters.
- Add a short explanation in parentheses on the first use of a technical term.
- Keep the explanation to 5–7 sentences, split it into small paragraphs of 1–3 sentences, and do not turn the work report into a general lesson.
- Explain one concrete mechanism from the current stage in order: where important data originates, how it moves, and where it is used.
- Do not repeat an idea without a new reason; choose the next explanation from the current stage.
