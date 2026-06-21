# claude-agy-cli-icons

A [Claude Code](https://claude.com/claude-code) skill that generates images
from a text description using the [Antigravity (`agy`) CLI](https://antigravity.google/product/antigravity-cli).

Ask Claude to "make me an icon of a blue rocket" and it drives `agy` headlessly to
produce a PNG and save it to your working directory with a timestamped filename.

## Requirements

- The [`agy` CLI](https://antigravity.google/product/antigravity-cli) installed, on
  your `PATH`, and authenticated.
- Claude Code (or any client that loads Claude skills).
- Network access — generation runs through `agy`'s backend.

## Install

Pick one:

- **Folder** — copy the `agy-image-generator/` directory into your Claude skills
  directory (e.g. `~/.claude/skills/`).

Once installed, the skill triggers automatically whenever you ask Claude to create,
generate, or make an icon, logo, glyph, or image from a description.

## Usage

Just describe what you want:

```
make me a flat minimalist icon of a blue rocket on a transparent background
```

```
generate 2 home button icons (golden background, black house) into ./assets
```

You can also use the explicit argument form:

```
[--out <dir>] [-n <count>] <image prompt>
```

| Argument        | Default           | Meaning                          |
| --------------- | ----------------- | -------------------------------- |
| `--out <dir>`   | current directory | Where to save the PNG(s).        |
| `-n <count>`    | `1`               | How many icons to generate.      |
| `<image prompt>`| —                 | Free-text description of the icon. |

Example:

```
--out /tmp/icons -n 1 a coffee cup glyph, line-art style
```

### Filenames

A single timestamp (`YYYYMMDD-HHMMSS`) is computed per run:

- One image → `<timestamp>.png`
- Many → `<timestamp>-1.png`, `<timestamp>-2.png`, … (one shared base, numbered
  suffixes, so a batch stays grouped on disk).

## How it works

`agy` is a general-purpose agent, so the *wording* of the prompt decides whether it
generates an image or drifts into code-assistant behavior. The skill encodes three
things learned by testing, which is what makes generation reliable:

1. The prompt begins with the literal phrase **`generate an image with this prompt:`**
   — the verified trigger for image generation.
2. It passes a **concrete filename** — `agy` saves whatever literal name it's given.
3. It uses **no extra flags** — just `agy --print "<prompt>"`. Flags like
   `--print-timeout` were observed to derail the agent into describing the flag rather
   than generating.

Output is a 1024×1024 PNG; each image takes roughly a minute. The skill verifies the
file exists on disk before reporting its path.

## Notes & limitations

- Image generation is **not deterministic** — the same prompt yields different results
  each run.
- One `agy` call per image; batches run sequentially.
- Requires `agy` to be authenticated and reachable; if it isn't, the skill reports a
  failed generation rather than producing a file.
- This skill **creates** images from text — it does not edit existing images or produce
  non-image assets.
