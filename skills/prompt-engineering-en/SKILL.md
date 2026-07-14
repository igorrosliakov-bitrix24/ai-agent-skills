---
name: prompt-engineering-en
description: "Helps in English to turn an unstructured task into a short, precise, executable instruction for an AI agent or model. Use for English requests to create, improve, compress, or diagnose prompts."
---

# Prompt Engineering

Use this skill when the user wants to formulate a task for a model, improve a prompt, split a complex request into steps, or make an instruction suitable for an agent.

## Workflow

1. Identify the goal, inputs, expected output, and constraints.
2. Remove ambiguity: roles, response format, completion criteria, prohibitions, and assumptions.
3. For complex tasks, split the prompt into context, task, process, and output format.
4. Add quality checks the model should perform before its final answer.
5. Keep the prompt short enough that it will actually be used.
6. When useful, provide two versions: compact and expanded.

## Constraints

- Do not overload the prompt with obvious instructions.
- Do not add contradictory requirements.
- Do not promise accuracy where sources or external verification are needed.
