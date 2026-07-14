# Dialogue Context: Building a Reusable Cover Creation Process

The user regularly creates cover images for articles, especially for Habr. They need a repeatable process that helps choose a visual direction, analyze references into parameters, fill a template, assemble a coherent prompt, and only then generate the image.

## Main Idea

An image usually does not have one magical style name. It is assembled from separate parameters:

- image type;
- era;
- genre;
- artistic execution method;
- camera or image source;
- lens and angle;
- lighting;
- color palette;
- texture;
- composition;
- atmosphere;
- post-processing;
- publication format.

For example, a reference with a fighter in a dark room can be broken down like this:

- photograph;
- late-1990s to early-2000s aesthetic;
- tech or military thriller;
- surveillance-camera look;
- night lighting;
- cold color grading;
- film grain;
- vignette and VHS artifacts;
- tense atmosphere.

## Universal Template

Image type:
(photograph, editorial illustration, 3D render)

Main subject:
(office employee, AI agent, server room)

Story:
(analyzes a call, collects data, manages tasks)

Era:
(late 1990s, present day, near future)

Genre:
(tech thriller, corporate drama, science fiction)

Artistic style:
(editorial photography, isometric illustration, paper collage)

Visual references:
(early-2000s spy game, technology magazine cover, 1990s office cinema)

Camera or image source:
(film camera, CCTV, digital SLR)

Lens:
(35 mm, 50 mm, wide-angle)

Angle:
(eye level, top-down, over the shoulder)

Composition:
(rule of thirds, symmetrical, centered)

Lighting:
(daylight through a window, harsh office light, neon)

Color palette:
(warm, cold blue, muted monochrome)

Materials and textures:
(film grain, matte metal, paper texture)

Detail level:
(minimal, detailed, photorealistic)

Atmosphere:
(moderately tense, calm, ironic)

Post-processing:
(VHS noise, vignette, chromatic aberration)

Format:
(Habr cover, magazine cover, website hero image)

Aspect ratio:
(16:9, 1:1, 9:16)

Title area:
(left, right, top)

Special requirements:
(no text, quiet center, important objects away from edges)

Exclude:
(logos, watermarks, random text)

Scene description:
[Describe in detail what is happening in the image.]

## Example

Type: photograph.
Main subject: office clerk.
Story: handles several calls at once.
Era: late 1990s.
Genre: tech thriller.
Style: editorial photography.
Visual reference: stealthy, tense aesthetic of early-2000s spy computer games without copying any specific character.
Camera: film camera.
Lighting: daylight.
Color: warm.
Texture: film grain.
Detail level: photorealism.
Atmosphere: moderately tense.
Post-processing: VHS noise and vignette.
Format: magazine cover.
Aspect ratio: 16:9.
Scene description: an office clerk tries to talk on two landline phones at once, holding the receivers with his shoulders and writing call notes in yellow notepads. Several already-filled notepads lie on the desk, showing these are far from the first calls of the day.

## How the Skill Should Work

1. Determine whether a scene idea already exists.
2. Help choose a visual approach.
3. Fill missing parameters by offering at most 2-3 options.
4. Check composition for the format and aspect ratio.
5. Clarify where the title area should be.
6. Assemble a coherent prompt without mechanically listing fields.
7. Remove contradictions between parameters.
8. Do not add text unless explicitly requested.
9. When a reference exists, describe its visual properties.
10. After agreement, create the image immediately if the agent has an image generator.
