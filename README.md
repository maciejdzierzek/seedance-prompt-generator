# Seedance Prompt Generator - Claude Skill

> A Claude skill for writing prompts for **ByteDance Seedance 2.0** video generation.
> Works with Claude.ai, Claude Desktop, and Claude Code.

[![Live Generator](https://img.shields.io/badge/Live_Generator-maciejdzierzek.com-blue)](https://maciejdzierzek.com/narzedzia/generator-promptow/?model=seedance)
[![Claude Skill](https://img.shields.io/badge/Claude-Skill-orange)](https://claude.ai)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

**Prefer a form over a conversation?** The same four models have a web generator, no install and no account: [maciejdzierzek.com/narzedzia/generator-promptow](https://maciejdzierzek.com/narzedzia/generator-promptow/?model=seedance)

## What is this?

Seedance 2.0 generates **audio and video jointly and natively**, takes text, images, video and audio as simultaneous input, and does multi-shot storytelling in a single generation. It is also unusually opinionated about how you write prompts - and it fails in specific, documented ways when you don't.

This skill teaches Claude the model as ByteDance documents it: the four task types and their distinct sentence patterns, the storyboard format, how to bind subjects to reference assets so the face stops drifting, the symbol conventions that make dialogue drive lip-sync, and the fixes for the three failures everyone hits.

## The mental model

Seedance 2.0 decomposes your input into a **spatial layer** (what is in the frame) and a **temporal layer** (how things change over time). So a good prompt is, in ByteDance's words, *"not simply a copywriting-style description, but an engineering-style instruction"*.

Which is why this fails:

> A man runs nervously down the street, and the scene feels very cinematic.

And this works:

> Shot 1: Side shot of a street alley; the man slowly starts running, with a sense of rapid breathing.
> Shot 2: The man knocks over a fruit stand; the camera shakes quickly and gives a close-up of the man's frightened face.
> Shot 3: The man climbs over a low wall and disappears; the camera slowly pulls back and freezes on the empty street.

Both examples are from the official prompt guide.

## What the skill covers

- **Model lineup** - Seedance 2.0 / Fast / Mini with model IDs, resolutions, ratios, the 4-15s duration range, and the fact that only Seedance 2.0 does 4K (10-bit H.265, concurrency 1)
- **Four task types with their own sentence patterns** - multimodal reference, video editing, video extension, combined. Including the gotcha that saying "reference `<Video 1>`" on an *edit* job silently turns it into a reference job
- **The official advanced formula** and the per-shot ordering: camera move → action and expression → position → audio
- **Subject definition and binding** - `Define [features] in <Image 1> as <Subject 1>`, `Subject@Image 1`, and why an Asset ID must never appear in the prompt
- **Action writing** - refine to body parts, quantify range and speed, prefer slow continuous small movements, bridge between actions, and the official table for externalising emotion into physical detail
- **Camera language** - the documented vocabulary, and the one-movement-per-shot rule
- **Symbol conventions** - `{ }` dialogue (this is what drives lip-sync), `< >` sound effects, `（ ）` music, `【 】` subtitles, plus the rule for marking non-Chinese/English dialogue
- **Asset strategy** - the hard limits (0-9 images, 0-3 videos, 0-3 audio) versus the vendor's own advice to use 4-5 assets in four functional roles, ordered by how precisely they must be matched
- **Extension vs. segmented stitching** - which one each kind of content wants
- **The official FAQ, in full** - character ID drift (and why multi-view character sheets make it *worse*), unwanted subtitles (including: landscape generates fewer than portrait), logos and watermarks, style drift, and the exact 6-frame / 1-frame trim recipe for extension seams
- **`references/prompt-templates.md`** - fill-in templates for every task type, a reusable stability tail, a dialogue/audio cheat sheet, an asset plan, and a cost-controlled iteration ladder

## Two things worth knowing before you generate

**No per-shot durations.** Precise timing such as "0-3 seconds" is documented as unstable and may produce abnormal results. Order the shots; let pacing follow the plot.

**Constraint words are not polish.** The docs call them *"necessary configurations for ensuring stable and compliant final output"*. Every template in this skill ends with a stability paragraph for that reason.

## Installation

### Claude Code

```bash
/plugin marketplace add maciejdzierzek/claude-plugins
/plugin install seedance-prompt-generator@maciejdzierzek
```

### Claude.ai and Claude Desktop

Download the ZIP from [Releases](../../releases), then **Customize > Skills** → **"+"** → **Upload a skill**. Requires code execution enabled in **Settings > Capabilities**.

### Manual

```bash
mkdir -p ~/.claude/skills/seedance
cp -r skills/seedance/* ~/.claude/skills/seedance/
```

## Usage

```
Write me a Seedance 2.0 prompt for a three-shot dialogue scene between two colleagues.
```
```
My character's face keeps changing halfway through the video. How do I fix it?
```
```
Seedance keeps burning subtitles into my vertical videos and I never asked for them.
```
```
How do I edit an existing clip with Seedance instead of generating a new one?
```

## Cost

Deliberately no price table. The charge depends on variant, resolution, duration, aspect ratio and whether audio is generated - all set per request - and ModelArk and third-party aggregators bill differently. The skill teaches the discipline instead: iterate in draft mode or at 480p/720p, read the cost the provider actually returns after a real call, and only then extrapolate a budget. A headline per-clip figure is a base rate for one configuration, and a model with a cheaper rate can end up costing more once resolution and duration are set.

## How this skill is maintained

Every factual claim is verified against ByteDance/BytePlus-controlled documentation - the Seedance 2.0 prompt guide, the series tutorial, and the Lens language reference on `docs.byteplus.com` - with the verification date stated in the skill. Those pages are JavaScript-rendered, so a plain fetch returns an empty shell; they were read with a headless browser. The skill carries an explicit list of things it deliberately does **not** claim, including per-second pricing, a lip-sync language count, and the availability of the announced-but-not-yet-shipped Seedance 2.5.

## About

Built by [Maciej Dzierżek](https://maciejdzierzek.com). All AI tools: [maciejdzierzek.com/narzedzia](https://maciejdzierzek.com/narzedzia/)

Companion skills: [kling-ai](https://github.com/maciejdzierzek/kling-ai-prompt-generator) (AI video) · [nano-banana](https://github.com/maciejdzierzek/nano-banana-prompt-generator) and [z-image](https://github.com/maciejdzierzek/z-image-prompt-generator) (images)

## License

MIT for this skill. Seedance model access and terms are ByteDance's, separately.
