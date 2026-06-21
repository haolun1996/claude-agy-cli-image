---
name: agy-icon-generator
description: >-
  Generate image icons (PNG) from a text description using the Antigravity
  (`agy`) CLI's image-generation feature, saving them to disk with timestamped
  filenames. Use this skill whenever the user wants to create, generate, make,
  or produce an icon, app icon, logo, glyph, button graphic, or any image from
  a description — even if they don't say the word "agy" or "icon" explicitly
  (e.g. "make me a picture of a rocket", "I need a 512px home button graphic",
  "generate an app icon for my notes app"). Also use it when the user invokes
  the documented argument form with optional --out and -n flags followed by an
  image prompt. Do NOT use it for editing existing images, generating non-image assets, or
  answering questions about the agy CLI itself.
---

# agy-icon-generator

Generate one or more image icons from a text prompt using the Antigravity CLI
(`agy`) and save them as timestamped PNG files. No bundled scripts — you run
`agy` directly following the steps below.

## The one command that matters

`agy` is a general-purpose agent, so the *wording* of the prompt decides whether
it generates an image or wanders off into code-assistant / research behavior.
This exact form is the verified trigger — use it verbatim, substituting only the
bracketed parts:

```bash
agy --print "generate an image with this prompt: <DESCRIPTION>. Save the generated image to <DIR> destination with file name <FILENAME>.png After saving, print the file path"
```

Three things are load-bearing here, each learned by testing — get any of them
wrong and you'll get a wall of text instead of an image:

1. **Start the prompt with the literal phrase `generate an image with this
   prompt:`.** Vaguer phrasings ("create an icon", "use your image capability")
   make `agy` explore the filesystem or answer like a research agent instead of
   generating.
2. **Pass a concrete `<FILENAME>`** (e.g. `20260620-175228.png`), never the word
   "timestamp" — `agy` saves whatever literal name it is given.
3. **Add no other flags.** Just `agy --print "<prompt>"`. Extra flags such as
   `--print-timeout` have been observed to derail the agent into describing the
   flag rather than generating. The default 5-minute timeout is ample — a single
   1024×1024 image takes roughly a minute.

## Procedure

1. **Parse the request** into three inputs:
   - `<DESCRIPTION>` — the descriptive part of what the user wants. Pass their
     words through faithfully, keeping stylistic cues (flat, minimalist, colors,
     transparent background, line-art, etc.) — these steer the output.
   - `<DIR>` — where to save. If the user named a destination ("my Desktop",
     "./assets"), use it; otherwise default to the current working directory.
     `mkdir -p` it first so the path exists. Prefer an absolute path so your
     later existence check matches what `agy` wrote.
   - count — how many to generate. Default `1`. Map quantities like "three" or
     "a couple" onto a number.

   If the user used the explicit form `[--out <path>] [-n <count>] <image
   prompt>`, `--out` is `<DIR>`, `-n` is the count, and the rest is
   `<DESCRIPTION>`.

2. **Compute one shared timestamp** for the whole run:
   `date +%Y%m%d-%H%M%S` (e.g. `20260620-175228`).

3. **Derive the filename(s)** from that single timestamp:
   - one image → `<timestamp>.png`
   - N images → `<timestamp>-1.png`, `<timestamp>-2.png`, … `<timestamp>-N.png`

   A batch shares one base timestamp with numbered suffixes, which keeps a set
   of related icons grouped together on disk.

4. **Generate each image** with its own `agy --print` call using the command
   form above. Run them one at a time (each takes ~a minute).

5. **Verify and report.** After each call, confirm the file actually exists at
   `<DIR>/<FILENAME>.png` (e.g. `ls -la` / `file`). Only report a path once
   you've confirmed it on disk. If a call produced no file, say that generation
   failed rather than claiming success — the usual causes are `agy` not being
   on `PATH` or not authenticated.

## Examples

**Example 1 — natural language, single icon**
Input: `make me a flat minimalist icon of a blue rocket on a transparent background`
Do: timestamp → `20260620-175228`; run
`agy --print "generate an image with this prompt: a flat minimalist icon of a blue rocket on a transparent background. Save the generated image to /current/dir destination with file name 20260620-175228.png After saving, print the file path"`;
verify; report `/current/dir/20260620-175228.png`.

**Example 2 — batch into a subdirectory**
Input: `generate 2 home button icons (golden background, black house) into ./assets`
Do: `mkdir -p ./assets`; timestamp → `20260620-180000`; two calls saving
`20260620-180000-1.png` and `20260620-180000-2.png` into `./assets`; verify both;
report both paths.

**Example 3 — explicit argument form**
Input: `--out /tmp/icons -n 1 a coffee cup glyph, line-art style`
Do: `<DIR>=/tmp/icons`, count `1`, `<DESCRIPTION>=a coffee cup glyph, line-art
style`; `mkdir -p /tmp/icons`; one call → `/tmp/icons/<timestamp>.png`; verify;
report.
