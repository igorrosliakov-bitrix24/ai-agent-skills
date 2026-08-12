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
3. First suggest up to three visual directions if the direction is not selected yet.
4. After choosing or recommending a direction, always output a full editable `Cover Parameters` block using the fields from the template. Do not compress it into a short list such as "type", "composition", and "palette".
5. Fill every field with a concrete value or short phrase. If a parameter is uncertain, provide 2-3 options inside that field.
6. Only after the parameter block, assemble the final prompt.
7. Find contradictions between parameters and suggest a concise correction.
8. For a 16:9 cover, keep important details away from the edges and leave a quiet area for the title.
9. When using a reference, transfer composition, era, palette, lighting, texture, and atmosphere.
10. By default, create images without text or logos.
11. If an image generator is available and the parameters are agreed, create the image immediately.

## Required Response Format

When the user asks to choose or prepare a cover, the response must contain:

1. A brief understanding of the task.
2. Up to three visual directions, if no direction has been selected yet.
3. A `Cover Parameters` block for the selected or recommended direction.
4. A `Final Prompt` block when the user is ready for generation or asks for a prompt.

Always write the `Cover Parameters` block in this format:

```text
Image type:

Main subject:

Story:

Era:

Genre:

Artistic style:

Reference:

Camera / image source:

Lens:

Angle:

Composition:

Lighting:

Color palette:

Materials / textures:

Detail level:

Atmosphere:

Post-processing:

Format:

Aspect ratio:

Special requirements:

Scene description:
```

Do not replace this block with a brief bullet list. The user needs it as a working form they can edit manually before generation.

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
- Keep example lists short: at most three options by default. When showing a range, choose one or two representative examples joined with "or" and explain the principle in prose.

## Materials

- Template: `references/prompt-template.md`
- Context: `references/dialogue-context.md`
- Example: `examples/90s-office-cover.md`
