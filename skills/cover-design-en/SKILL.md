---
name: cover-design-en
description: Helps in English to develop cover concepts, break down visual style into usable parameters, assemble image-generation prompts, and prepare cover images for articles, magazines, or websites. Use for English requests about cover design, visual references, image prompts, and publication-ready composition.
---

# Cover Design

Use this skill when the user asks to design a cover, choose a visual direction, analyze a visual reference, write an image prompt, or adapt an image for an article, magazine, or website.

## Core Principle

An image style is built from separate parameters. Separate image type, era, genre, camera/source, lighting, color, composition, texture, and post-processing.

## Workflow

1. Identify the topic, main visual metaphor, publication surface, aspect ratio, and whether a title area is needed.
2. Use the template in `references/prompt-template.md`.
3. Do not require every field to be filled: keep only the parameters that matter for the task.
4. Find contradictions between parameters and suggest a concise correction.
5. For a 16:9 cover, keep important details away from the edges and leave a quiet area for the title.
6. Assemble the final prompt as one coherent description, not as a questionnaire.
7. When using a reference, transfer composition, era, palette, lighting, texture, and atmosphere.
8. By default, create images without text or logos.
9. If an image generator is available and the parameters are agreed, create the image immediately.

## Recommended Final Prompt Order

1. Format and image type.
2. Scene and main subject.
3. Action.
4. Era and genre.
5. Camera, angle, and composition.
6. Lighting and color.
7. Textures and post-processing.
8. Atmosphere.
9. Technical constraints.

## Constraints

- Do not suggest more than three visual directions at once.
- Do not overload the user with long lists of examples.
- Do not turn the prompt into disconnected trendy keywords.
- Do not add random holograms, robots, or neon just because the topic mentions AI.
- Check that the cover communicates the article's meaning and remains readable at small size.

## Materials

- Template: `references/prompt-template.md`
- Context: `references/dialogue-context.md`
- Example: `examples/90s-office-cover.md`
