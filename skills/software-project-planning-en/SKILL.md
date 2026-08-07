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
