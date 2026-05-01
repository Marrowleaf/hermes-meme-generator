---
name: meme-generator
description: Generate memes from templates with custom text overlays using Python Pillow, supporting 20+ popular formats with batch generation for content calendars
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  tags:
    - creative
    - memes
    - images
    - pillow
    - content-calendar
    - telegram
    - obsidian
  related_skills:
    - social-poster
    - obsidian-vault
---

# Meme Generator

Create memes from popular templates with custom text overlays. Supports caption-only memes, template-based memes, batch generation, and direct sharing via Telegram or Obsidian.

## Overview

Meme Generator lets you quickly produce memes using a library of 20+ popular templates. It uses Python Pillow for text overlay with Impact font, white text, and black stroke — the classic meme aesthetic. Memes can be saved to Obsidian attachments, sent via Telegram, or batch-generated for content calendars.

## Commands

### `/meme <template> <top_text> | <bottom_text>`
Generate a meme from a template with top and/or bottom text.

**Arguments:**
- `<template>` — Template name or ID (use `/meme-templates` to browse)
- `<top_text>` — Text for the top panel (use `_` for none)
- `<bottom_text>` — Text for the bottom panel (separated by `|`)

**Examples:**
```
/meme drake "Write tests" | "Ship directly to prod"
/meme distracted-boyfriend "New framework" | "Current framework" | "Old framework"
/meme this-is-fine _ | "When the server crashes on Friday"
```

### `/meme-caption <text>`
Generate a caption-only meme (Impact font, white text, black stroke on solid background).

**Options:**
- `--bg <color>` — Background color (default: `black`; options: `black`, `white`, `red`, `blue`, `custom:#hex`)
- `--font-size <size>` — Font size in pixels (default: auto-sized)
- `--width <px>` — Image width (default: `600`)
- `--height <px>` — Image height (default: auto from text)

**Examples:**
```
/meme-caption "Nobody: ... Developers at 3am:"
/meme-caption "When it works on the first try" --bg white
/meme-caption "404 Motivation Not Found" --bg custom:#1a1a2e
```

### `/meme-custom <image_path> <top_text> | <bottom_text>`
Apply meme text overlay to a custom image.

**Options:**
- `--font <name>` — Font to use (default: `impact`; options: `impact`, `arial`, `comic-sans`, `helvetica`)
- `--stroke-width <px>` — Stroke width (default: `3`)
- `--text-color <color>` — Text fill color (default: `white`)
- `--stroke-color <color>` — Stroke color (default: `black`)

**Examples:**
```
/meme-custom ~/photo.jpg "My code" | "When my boss walks by"
/meme-custom ~/meme-base.png _ | "Bottom text only" --font arial
```

### `/meme-templates [filter]`
List available meme templates. Optionally filter by name or category.

**Examples:**
```
/meme-templates
/meme-templates reactions
```

### `/meme-batch <csv_file>`
Batch generate memes from a CSV file.

**CSV format:**
```csv
template,top_text,bottom_text,output_name
drake,"Write docs","Read the code",meme-001
this-is-fine,,"Server is fine",meme-002
distracted-boyfriend,"New JS framework","Current framework","The old one",meme-003
```

### `/meme-save <meme_id> [destination]`
Save a generated meme to Obsidian attachments or a specified path.

**Destination options:**
- `obsidian` (default) — Save to `~/obsidian-vault/Attachments/memes/`
- `telegram` — Send directly via Telegram
- `both` — Save to Obsidian and send via Telegram
- Custom path — Save to specified directory

## Template Library

### Reaction Memes
| ID | Template | Panels | Description |
|----|----------|--------|-------------|
| `drake` | Drake Hotline | 2 | Preference/disapproval split |
| `distracted-boyfriend` | Distracted Boyfriend | 3 | Temptation triangle |
| `this-is-fine` | This is Fine | 2 | Denial in crisis |
| `gal-brain` | Expanding Brain | 5 | Escalating ideas |
| `change-my-mind` | Change My Mind | 1 | Stubborn opinion |
| `surprised-pikachu` | Surprised Pikachu | 1 | Unexpected consequence |
| `disaster-girl` | Disaster Girl | 1 | Sinister observer |
| `side-eye-chloe` | Side-Eye Chloe | 1 | Skeptical reaction |

### Comparison Memes
| ID | Template | Panels | Description |
|----|----------|--------|-------------|
| `two-buttons` | Two Buttons | 3 | Difficult choice |
| `trade-offer` | Trade Offer | 2 | Unfair exchange |
| `buffet-vs-dessert` | Buffet vs Dessert | 2 | Priority comparison |
| `is-this-a-pigeon` | Is This a Pigeon | 3 | Misidentification |

### Situational Memes
| ID | Template | Panels | Description |
|----|----------|--------|-------------|
| `its-free-real-estate` | It's Free Real Estate | 1 | Dubious offer |
| `this-is-where-id-put-my-x` | Where I'd Put X | 2 | Absent feature |
| `slapping-car-roof` | Slapping Car Roof | 1 | Product pitch |
| `woman-yelling-at-cat` | Woman Yelling at Cat | 2 | Heated disagreement |
| `panik-kalm-panik` | Panik Kalm Panik | 3 | Anxiety cycle |
| `monkey-puppet` | Monkey Puppet | 1 | Awkward avoidance |
| `stonks` | Stonks | 1 | Fake financial success |
| `press-f` | Press F to Pay Respects | 1 | Mock sorrow |

### Tech-Specific
| ID | Template | Panels | Description |
|----|----------|--------|-------------|
| `deployment-pipeline` | Deployment Pipeline | 4 | CI/CD humor |
| `stack-overflow` | Stack Overflow | 2 | Copy-paste coding |
| `works-on-my-machine` | Works on My Machine | 1 | Environment excuses |
| `git-blame` | Git Blame | 2 | Code archaeology |

## Technical Implementation

### Image Generation with Pillow

```python
from PIL import Image, ImageDraw, ImageFont
import textwrap
import os

def create_meme(template_name, top_text, bottom_text):
    """Generate a meme from a template with text overlay."""
    # Load template image
    template_path = get_template_path(template_name)
    img = Image.open(template_path)
    draw = ImageDraw.Draw(img)

    # Load Impact font with fallback
    try:
        font = ImageFont.truetype("/usr/share/fonts/impact.ttf", size=48)
    except OSError:
        font = ImageFont.truetype("/usr/share/fonts/truetype/impact.ttf", size=48)

    # Text styling: white fill, black stroke
    fill_color = (255, 255, 255)    # White
    stroke_color = (0, 0, 0)        # Black
    stroke_width = 3

    # Calculate text wrapping based on image width
    max_chars = int(img.width / 18)  # Approximate chars per line

    # Draw top text
    if top_text and top_text != "_":
        lines = textwrap.wrap(top_text.upper(), width=max_chars)
        y = 10
        for line in lines:
            bbox = draw.textbbox((0, 0), line, font=font)
            x = (img.width - (bbox[2] - bbox[0])) // 2
            draw.text((x, y), line, font=font,
                      fill=fill_color, stroke_width=stroke_width,
                      stroke_fill=stroke_color)
            y += bbox[3] - bbox[1] + 8

    # Draw bottom text
    if bottom_text:
        lines = textwrap.wrap(bottom_text.upper(), width=max_chars)
        y = img.height - 10
        for line in reversed(lines):
            bbox = draw.textbbox((0, 0), line, font=font)
            x = (img.width - (bbox[2] - bbox[0])) // 2
            y -= (bbox[3] - bbox[1] + 8)
            draw.text((x, y), line, font=font,
                      fill=fill_color, stroke_width=stroke_width,
                      stroke_fill=stroke_color)

    return img

def create_caption_meme(text, bg_color="black", width=600):
    """Generate a caption-only meme with colored background."""
    # Create background
    font = ImageFont.truetype("/usr/share/fonts/impact.ttf", size=42)
    wrapped = textwrap.wrap(text.upper(), width=30)
    line_height = 52
    height = (len(wrapped) * line_height) + 40
    img = Image.new("RGB", (width, height), bg_color)
    draw = ImageDraw.Draw(img)

    y = 20
    for line in wrapped:
        bbox = draw.textbbox((0, 0), line, font=font)
        x = (width - (bbox[2] - bbox[0])) // 2
        draw.text((x, y), line, font=font,
                  fill=(255, 255, 255), stroke_width=3,
                  stroke_fill=(0, 0, 0))
        y += line_height

    return img
```

### Multi-Panel Meme Assembly

For templates with multiple panels (e.g., Drake, Expanding Brain):

```python
def create_panel_meme(template_name, texts):
    """Build a multi-panel meme by stacking image sections."""
    template = TEMPLATE_REGISTRY[template_name]
    panels = []

    for i, section in enumerate(template["sections"]):
        panel = load_panel_image(template_name, i)
        if texts[i]:
            panel = overlay_text(panel, texts[i],
                                 position=section["text_position"])
        panels.append(panel)

    return stack_panels(panels, direction=template["layout"])
```

## Batch Generation for Content Calendars

Creating memes for a content calendar:

```csv
# memes-january.csv
template,top_text,bottom_text,output_name,schedule_date
drake,"Write documentation","Just add comments",meme-001,2025-01-06
this-is-fine,,"Prod is fine",meme-002,2025-01-08
surprised-pikachu,,"When the test passes",meme-003,2025-01-10
```

```
/meme-batch ~/memes-january.csv

→ 🎨 Generating batch memes (3 items)...
→ ✅ meme-001.png (drake)
→ ✅ meme-002.png (this-is-fine)
→ ✅ meme-003.png (surprised-pikachu)
→ 💾 All saved to: ~/obsidian-vault/Attachments/memes/batch/

📊 Batch Summary:
  - Generated: 3/3
  - Failed: 0
  - Total size: 1.2 MB
```

Integrate with `social-poster` skill:
```
/post-draft twitter "" --media ~/obsidian-vault/Attachments/memes/batch/meme-001.png --tone casual
```

## Pitfalls

1. **Font availability** — Impact font may not be installed on all systems. The skill falls back to available bold sans-serif fonts, but the classic meme look requires Impact. Install `fonts-freefont-ttf` or equivalent.

2. **Text overflow** — Very long text can overflow panel boundaries. The skill auto-wraps text, but extremely long strings may produce tiny font sizes. Keep top/bottom text under 50 characters for best results.

3. **Image quality** — Template images must be at least 600px wide. Low-resolution templates produce blurry memes. The template library uses only high-quality source images.

4. **Special characters** — Unicode emojis and special characters in Impact font may not render correctly. Stick to ASCII for classic meme text; use `--font arial` for emoji support.

5. **Multi-panel alignment** — Templates with more than 2 panels require careful text placement. The skill uses predefined text zones per template, but custom images may need manual adjustment.

6. **File size** — Meme images with large dimensions can exceed Telegram's 5MB send limit. The skill auto-compresses to stay under limits.

7. **Obsidian attachment path** — The default save path `~/obsidian-vault/Attachments/memes/` must exist. The skill creates it if missing, but verify vault path configuration.

8. **Batch CSV formatting** — The batch CSV must use proper quoting for fields containing commas. A malformed CSV will skip the affected row with a warning.

9. **Copyright concerns** — Some meme images have copyright restrictions. The skill uses widely-shared template images, but commercial use may require additional licenses.

10. **Pillow version** — Requires Pillow >= 9.0 for `stroke_width` parameter in `draw.text()`. Older versions will throw a TypeError.

## Verification Steps

1. **Check Pillow is installed:**
   ```bash
   python3 -c "from PIL import Image, ImageDraw, ImageFont; print('Pillow OK')"
   ```

2. **Check Impact font availability:**
   ```bash
   python3 -c "from PIL import ImageFont; ImageFont.truetype('/usr/share/fonts/impact.ttf', 48); print('Impact font OK')"
   ```
   If missing, install: `sudo apt install fonts-freefont-ttf` or download Impact font.

3. **Test basic meme generation:**
   ```
   /meme drake "Write code" | "Write tests"
   ```
   Should produce a two-panel Drake meme image and display it.

4. **Test caption-only meme:**
   ```
   /meme-caption "Hello World this is a test"
   ```
   Should produce a black-background meme with white Impact text.

5. **Verify template library:**
   ```
   /meme-templates
   ```
   Should list 20+ templates with IDs, names, and panel counts.

6. **Test custom image:**
   Use `/meme-custom` with a local image file and verify text overlay is positioned correctly.

7. **Test Obsidian save:**
   ```
   /meme-save meme-output obsidian
   ```
   Check `~/obsidian-vault/Attachments/memes/` for the saved file.

8. **Test Telegram send:**
   ```
   /meme-save meme-output telegram
   ```
   Verify the meme image is sent as a photo in the current Telegram chat.

9. **Test batch generation:**
   Create a small CSV with 2-3 entries and run `/meme-batch`. Verify all images are generated and saved.

10. **Test error handling:**
    - Try `/meme nonexistent-template "test"` → should show template not found error
    - Try `/meme drake "text that is extremely long and goes on forever"` → should auto-wrap or warn