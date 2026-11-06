---
name: seedance
description: Write and improve prompts for ByteDance Seedance video generation, and pick the right model across Seedance 2.5 and the 2.0 series. Covers the four task types (multimodal reference, video editing, video extension, combined), the shot-by-shot storyboard format the model expects, subject definition and binding to reference assets, the official symbol conventions for dialogue, sound effects, music and subtitles, camera language, constraint words, asset selection strategy, token pricing, and fixes for character ID drift, unwanted subtitles and style drift. Includes what changes on Seedance 2.5 - timestamps, 30-second clips, up to 50 references, locked edit and extension parameters.
when_to_use: Use for anything involving Seedance, Seedance 2.5, Seedance 2.0, Seedance 2.0 Fast, Seedance 2.0 Mini, Seedance 1.5 Pro, Dreamina, ByteDance video generation, ModelArk video models, or the model IDs dreamina-seedance-2-5-*, dreamina-seedance-2-0-* and bytedance/seedance-*. Trigger phrases include "Seedance", "generate a video with audio", "multi-shot AI video", "30-second AI video", "edit an existing video with AI", "extend this video", "lip sync video", "character keeps changing face in my video", "unwanted subtitles in generated video", "video prompt".
---

# Seedance Prompting Guide (2.0 series, with 2.5 differences)

Seedance is ByteDance's video generation family (branded **Dreamina Seedance** on BytePlus ModelArk). It generates **audio and video jointly and natively** - not audio added afterwards - and accepts text, images, video and audio as simultaneous input.

Facts here are verified against ByteDance's own ModelArk documentation on **2026-11-06**. See "Verification & Sources". The prompt rules in Steps 1-7 are the Seedance 2.0 series rules. **Seedance 2.5 changes several of them** - see "Seedance 2.5: what differs" right after the model table before writing a 2.5 prompt.

**The mental model that makes this work** (from the official guide): Seedance 2.0 is a multimodal director that decomposes your input into a **spatial layer** (what is in the frame) and a **temporal layer** (how things change over time). So a good prompt is *"not simply a copywriting-style description, but an engineering-style instruction"*: who, in what scene, doing what, how the camera moves, and in what order events happen - each delivered to the right layer.

---

## Model lineup

| | **Seedance 2.5** | **Seedance 2.0** | **Seedance 2.0 Fast** | **Seedance 2.0 Mini** |
|---|---|---|---|---|
| ModelArk model ID | `dreamina-seedance-2-5-260628` | `dreamina-seedance-2-0-260128` | `dreamina-seedance-2-0-fast-260128` | `dreamina-seedance-2-0-mini-260615` |
| Status | **API live** | API live | API live | API live |
| Positioning (vendor's words) | next-generation: long-form storytelling, omni reference, editing | highest generation quality in the 2.0 series | balance of cost and speed | best cost performance |
| Resolution | 480p, 720p, 1080p (10-bit) | 480p, 720p, 1080p, **4K** (10-bit) | 480p, 720p | 480p, 720p |
| Aspect ratios | 21:9, 16:9, 4:3, 1:1, 3:4, 9:16, adaptive | same | same | same |
| Duration | **4-30 s**, or -1 (model picks) | 4-15 s, or -1 | 4-15 s, or -1 | 4-15 s, or -1 |
| Frame rate | 24 fps | 24 fps | 24 fps | 24 fps |
| Output format | mp4, **mov** | mp4 | mp4 | mp4 |
| Reference assets, maximum | **50** (30 images + 10 videos + 10 audio) | 15 (9 images + 3 videos + 3 audio) | same | same |
| Text-to-video, first frame, first+last frame | ✅ | ✅ | ✅ | ✅ |
| Omni reference (image / video / combined) | ✅ | ✅ | ✅ | ✅ |
| Audio-only reference | ✅ | ❌ needs an image or video | ❌ | ❌ |
| Edit video, extend video | ✅ | ✅ | ✅ | ✅ |
| Native audio (`generate_audio`), `return_last_frame` | ✅ | ✅ | ✅ | ✅ |
| Draft mode | ✅ | ❌ | ❌ | ❌ |
| Offline inference (`service_tier: flex`) | ❌ | ❌ | ❌ | ❌ |
| Max RPM / concurrency (individual) | 180 / 3 | 180 / 3, **4K: 15 / 1** | 180 / 3 | 180 / 3 |
| Max RPM / concurrency (enterprise) | 600 / 10 | 600 / 10, **4K: 15 / 1** | 600 / 10 | 600 / 10 |

**Which one.** Use **2.5** when you need more than 15 seconds in one generation, timestamp control, more references, an audio-only reference, or `mov` for edit and extension work. Use **2.0** when you need **4K**: 2.5 stops at 1080p. Use **Fast** or **Mini** when cost decides. 2.5 costs about 1.5x the 2.0 token rate; see Cost at the bottom. Fast and Mini share the 2.0 feature set and stop at 720p.

4K on Seedance 2.0 is **10-bit H.265 (HEVC)** - beautiful gradients, but some players and browsers will not play it back directly, so plan a transcode for review. A 4K batch is serialised whether you plan for it or not: concurrency is 1 for every account type.

**Seedance 1.5 Pro** (`seedance-1-5-pro-251215`) is the previous generation and has its own separate prompt guide - if a user is on 1.5, the formulas below still broadly apply but check that guide for specifics.

**Access routes:** BytePlus ModelArk directly, or third-party aggregators which expose their own IDs such as `bytedance/seedance-2.0` and `bytedance/seedance-2.0-fast`. To activate the models on ModelArk you need one of: account balance above USD 30, an AI Savings Plan at the USD 30 tier or above, or a resource pack with available quota. See Cost at the bottom - the two routes bill differently.

---

## Seedance 2.5: what differs

The 2.5 prompt guide is a separate document with partly different rules. Where the steps below conflict with this section, this section wins for 2.5. All of it comes from the [2.5 tutorial](https://docs.byteplus.com/en/docs/modelark/seedance-2-5) and [2.5 prompt guide](https://docs.byteplus.com/en/docs/modelark/seedance-2-5-prompt-guide).

**Prompt shape.** The guide frames you as a visual content producer: map the assets, give a one-sentence summary (subject + location + event + genre/style + camera), write the detailed plot by "Shot N" or by timestamps, then add notes on anything that must hold throughout. The tutorial's basic order: subject + action/event + scene and environment + visual style + camera movement/shot cuts + sound.

**Timestamps work.** The vendor's comparison is explicit: 2.0 does not respond to timestamps, 2.5 supports integer seconds. Supported forms: intervals (`0-3 seconds ... 3-7 seconds`, `[1s-4s]`), time points (`at the 5-second mark`), relative time (`after 3 seconds`). Keep the timeline continuous with no gaps, and balance the load: too little in a range and the model improvises, too much and it cuts hard or drops plot. Do not use timestamps for high-frequency actions.

**Actions.** The advice flips from 2.0: prefer general descriptions and write specifics only for a few memorable actions. Use descriptive sentences for expressions rather than idioms. Avoid intense emotional language: words like "fanatical" can make pupils glow, and the fix is neutral wording plus "normal human eyes; no glowing eyes".

**Locked versus unlocked tasks.** Some tasks take their format from the input, and breaking that returns an error (`InvalidParameter.TaskTypeConstraint`, or `InvalidParameter.TaskTypeMismatch` when the model reads the task differently from what you declared):

| Task | `ratio` | `duration` | Prompt must contain | Notes |
|---|---|---|---|---|
| Edit | `adaptive` | `-1` | edit video, add, insert, remove, delete, modify, replace, change to | Input video 4-30 s, best under 20 s. Output keeps its ratio and length, up to about 0.4 s shorter. Prefer `output_format: mov` |
| Extend | `adaptive` | 4-30 or `-1` | extend forward, extend backward, continue, continue from, extend the story | Use `mov` for both input and output for the best continuity |
| First frame, first+last frame | `adaptive` | 4-30 or `-1` | - | Set `content.role` to `first_frame` / `last_frame`. Use the same ratio for both, or the last frame gets stretched |
| Reference, storyboard, keyframes | free | free | - | The input is a semantic reference only |

Set `omni_reference_task_type` to `edit` or `extend` to validate early instead of failing after the task starts; `auto` lets the model classify from the prompt.

**Flexible ratio.** 2.0 has six fixed output ratios. 2.5 can produce any ratio between 0.4 and 2.5 by controlling the input assets.

**Mapping many assets.** Number assets by upload order and bind each one in the prompt text: "Images 1-2 are Character 1 and correspond to Audio 1". Say what each asset provides and what it should not. Do not rely on a name written inside the image. If characters swap, upload references in the order subjects first appear. Multi-view subject images are supported for 1-5 subjects, unlike 2.0. Vendor guidance: 1-8 subjects from images, 1-5 from audio or video with 5-10 s clips.

**Storyboards versus keyframes.** A multi-panel storyboard is a plot reference, not a frame-accurate one: 15 panels at most, simple line art, no text on it. For strict alignment use keyframes: separate images, opening with "Use Images 1 to 7 in order as keyframes."

**Negatives.** Prefer positive description. Negative control is supported for subtitles and audio: "no subtitles", "no BGM". If music still appears, enumerate the terms (music, BGM, score, instrumental, melody) and repeat the constraint at the start and end of the prompt.

**Other documented fixes.** Subtitles triggered by dialogue: use `Character's line (emotion): content` and do not repeat words after a line. Misspelled on-screen text: spell it letter by letter, or supply the text as an image reference. Fingerprint-like textures on foliage: resize reference images so they do not exceed the output resolution. Mixing 2.0 and 2.5 footage: the aesthetics differ visibly, so use a 2.0 clip as input to a 2.5 extension with `mov` output.

**Languages.** Native generation in 11: Chinese, English, Spanish, Indonesian, Malay, Thai, Arabic, Portuguese, Vietnamese, Japanese, Korean. **Polish is not listed.** The tutorial recommends stating the language before any non-Chinese dialogue.

**Real faces.** Neither 2.5 nor the 2.0 series accepts directly uploaded reference images or video containing real human faces. The vendor's routes are trusted outputs from your own account, preset digital characters, and authorized real-person assets ([guide](https://docs.byteplus.com/en/docs/modelark/seedance-portrait-asset-guide)).

---

## Step 1: pick the task type, because it changes the sentence pattern

Reference-based generation splits into three tasks plus combinations, and **each has its own official sentence pattern**. Using the wrong one is the most common structural mistake.

### Multimodal reference - build a new video from elements of your assets

The docs now call this task **omni reference**, and the API roles are `reference_image`, `reference_video`, `reference_audio`.

Use for action transfer, subject reuse, atmosphere reference.

```
Reference <Subject_N> in <Image_N> to generate ...
Reference <Action / Camera_movement / Style / Sound_effect> in <Video_N> to generate ...
Reference the timbre in <Audio_N> to generate ...
```

### Video editing - modify an existing video, everything unmentioned stays as it is

Use for local replacement, subject removal, attribute changes.

```
Add elements:    <Element_Features> + <Timing> + <Location>
Modify elements: Strictly edit <Video_N>, and modify <Original_Characteristic> in it to <New_Characteristic>
Delete elements: name the elements to remove; for anything that must stay, say so explicitly
```

### Video extension - continue an existing video in time

Use for continuing a plot, extending an action, completing a clip. Audio-video style, subject and narrative must stay consistent.

```
Extend <Video_N> forward/backward to generate ...
Track completion: <Video_1> + <Transition_Description> + followed by <Video_2> + ...
```

### Combined

```
Reference [Reference_Dimension] of <Image/Video_N>, strictly edit <Video_X>, [Specific_Edits]
```

### ⚠️ The gotcha that silently ruins edit and extend jobs

Straight from the official guide: for **edit** and **extend** tasks, refer to the video *directly* as `<Video_N>`. **Do not write "reference `<Video_N>`"** - the word "reference" makes the model classify the job as a *reference* task instead, and you get a new video inspired by yours rather than an edit of it. Note the deliberate wording in the official edit pattern: **"Strictly edit `<Video_N>`"**.

---

## Step 2: the advanced formula

The official formula, verbatim:

> precise subject + action details + scene/environment + lighting & color tone + camera movement + visual style + image quality + constraints

In plain terms: lock in *who* is *doing what*, then *where* and *what atmosphere*, then tell the model *how to shoot it*, then tighten the result with style, quality and constraints.

---

## Step 3: define your subjects explicitly

A reference image usually contains more than one thing. If you do not name what you mean, the model guesses.

```
Define [Core_Subject_Features] in <Image/Video_N> as <Subject_N>
```

- Use **2-3 clear, stable, static features** - clothing, hairstyle, appearance, category. Enough to identify uniquely, no more.
- Examples from the docs: *"Define the woman wearing a red dress and a straw hat in Image 1 as Subject 1"*, or give her a name: *"... as Zhang Hong"*.
- **Every later mention must re-refer explicitly.** Two supported styles:
  - Undefined subjects: bind on every mention with `<Subject_N>@<Image_N>`, e.g. `Zhang San@Image 1`.
  - Pre-defined subjects: reuse the same label consistently. Define the tall man in Video 1 as `police officer` and the short one as `thief`, then always say "police officer" and "thief".
- **Keep it concise and non-contradictory** - do not give one subject conflicting attributes.
- **Prefer showing spatial relationships in a reference image** over describing them in long text.
- If you use an asset library, you still address assets as `<Image/Video_N>` in the prompt. **The model cannot associate an Asset ID with its content** - never put the Asset ID in the prompt.

*(Official examples use both bracket forms - `[Video 1]`, `[Image 1]` in the API tutorial and `@Image 1` in the prompt guide's worked example. Both appear in vendor material; pick one and be consistent within a prompt.)*

---

## Step 4: sequence the shots

Because space and time are modelled separately, **the ideal prompt for anything non-trivial is a timeline storyboard**: split the video into shots and describe each in the order events occur.

Negative example from the docs: *"A man runs nervously down the street, and the scene feels very cinematic."*

Positive example from the docs:

> Shot 1: Side shot of a street alley; the man slowly starts running, with a sense of rapid breathing.
> Shot 2: The man knocks over a fruit stand; the camera shakes quickly and gives a close-up of the man's frightened face.
> Shot 3: The man climbs over a low wall and disappears; the camera slowly pulls back and freezes on the empty street.

Organise **each** shot in this order:

1. **Camera movement or transition** - "slowly push in from a wide shot", "fixed camera position", "cut to ..."
2. **Subject action and expression** - the key movement and change of expression
3. **Position or spatial change** - where the subject is, how that changes
4. **Audio** - sound effects, voices, background music for that shot

**Do not assign durations to shots on the 2.0 series.** The docs are explicit: support for precise timing such as "0-3 seconds" is **unstable, and forcing it may produce abnormal results**. (2.5 supports timestamps.) Let pacing follow the plot. Order shots primary-first.

---

## Step 5: write actions the way the model wants them

This section is where most quality is won or lost.

**Refine to body parts and quantify the degree.** Actions should name hands, legs, head, shoulders, back - plus range, speed and force. "Slowly raise a hand", "quickly turn the head", "push hard off the ground", "slightly lower the head".

**Prefer slow, gentle, continuous, small movements.** The docs say to *avoid* high-burst, large-dynamic actions - sprinting, big jumps, violent rolls. "Walk slowly", "gently raise a hand", "sit down naturally with the motion". This is a genuine model limitation, not timidity: big motion is where video models break.

**Bridge between actions.** State inertia and continuity: "use the inertia of turning around to naturally raise a hand", "naturally transition from a pause into raising a hand".

**Externalise emotion into physical detail** instead of naming the feeling. The official mapping table:

| Abstract emotion | Write instead |
|---|---|
| Sadness | lowering the head, shoulders trembling slightly, eyes reddening, fingers unconsciously clutching the corner of clothing, tears welling but not falling |
| Joy | corners of the mouth rising uncontrollably, brows and eyes relaxing, steps becoming light, unconsciously humming, unable to resist spinning in place |
| Nervousness / anxiety | frequently checking the watch, fingers tapping the tabletop, rapid breathing, eyes darting away, biting fingernails |
| Anger | fists clenched, jawline tense, chest heaving, eyes sharp, squeezing words out through gritted teeth |
| Relief | letting out a long breath, tense shoulders relaxing, a faint long-lost smile, looking up toward the distance |

---

## Step 6: camera

The model has strong understanding of standard camera terminology - use it directly rather than inventing descriptions: *medium shot, close-up, wide shot, slow push-in, smooth lateral tracking, fixed shot*.

**One camera movement per shot.** The docs are explicit: do not ask for push, pull, pan and move simultaneously - it increases image instability. If you need a complex move, give it its own shot.

Vocabulary the vendor documents as reliably understood (from the Lens language reference): **push, pull, pan, move, circle around, follow, rise, zoom**; shot sizes **long / full / medium / close-up**; perspectives **underwater, aerial, high-angle, low-angle, macro, over-the-shoulder, shooting through <object>, "with X as the foreground"**. Combined multi-move long takes are documented as an advanced technique - possible, but expect less stability.

---

## Step 7: quality, style and constraint words

The docs call these *"necessary configurations for ensuring stable and compliant final output"* - not optional polish.

- **Image quality:** HD, rich details, cinematic texture, natural colors, soft lighting
- **Style:** name it concretely - "cyberpunk cool blue-purple tone", "retro film", "fresh Japanese style", "2D Japanese anime style", "3D Chinese-style comic"
- **Constraint words** (described as "very important" - they suppress deformities, breakdowns and stray elements):
  - Subtitles: *"keep it subtitle-free"*, *"avoid generating any text or subtitles"*
  - Logo: *"do not generate a logo"*
  - Watermark: *"do not generate a watermark"*

Standard tail for a clean render, close to the official worked example:

```
The entire video should be high-definition with cinematic texture, natural colors and soft lighting.
The character's face remains stable without deformation; movements are natural and smooth, with no
stutter or flicker. Keep it subtitle-free. Do not generate a logo or watermark.
```

---

## Symbols: the official conventions

These are documented conventions, not stylistic preference - they tell the model what kind of information it is reading.

| Information type | Symbol | Example |
|---|---|---|
| Music | `（ ）` | （fast-paced rock music is playing in the background） |
| Sound effect | `< >` | < dog barking can be heard in the distance > |
| Dialogue | `{ }` | {Hello, world} |
| Subtitles | `【 】` | 【Chapter One: Departure】 |

**Dialogue language rules:**
- Keep dialogue language **consistent**; do not mix Chinese and English (proper nouns excepted).
- For a language other than Chinese or English, **mark the language**: `says in Japanese {こんにちは}`. By the same rule, Polish dialogue should be written `says in Polish {Dzień dobry}`.
- Native lip-sync is a headline feature, so dialogue in `{ }` is the mechanism that drives it - not a caption instruction.

Text rendered **on screen** (ad slogans, subtitles, speech bubbles) is supported; prefer common characters and avoid rare glyphs and special symbols. There is a dedicated "Text generation" page in the docs for that topic.

---

## Asset strategy: fewer than you are allowed

Hard input limits on the 2.0 series: **0-9 images, 0-3 videos, 0-3 audio files**. Also: **"text + audio" and "audio-only" inputs are not supported** - if you pass audio, pass an image or video with it. (2.5 raises these to 30 + 10 + 10 and accepts audio-only references.)

But the vendor's own recommendation is to use far fewer. Assets fall into four functional roles:

| Role | Purpose |
|---|---|
| Character anchoring | lock the character's appearance |
| Scene tone-setting | lock the environment and style |
| Camera movement reference | lock shot language and action rhythm |
| Rhythmic atmosphere | audio controls emotion and timbre |

**Recommended configuration: 4-5 assets total** - 1-2 character images (facial close-up and/or full body) + 1 scene image + 1 camera-movement video + 1 audio clip.

> *"It is not recommended to use the full asset limit. Too many assets will make it difficult for the model to judge feature priorities, which can easily cause issues such as style conflicts, blurry subject identification, and generated results deviating from expectations."*

**Order matters: place the assets that need the most precise reference earliest in the prompt.**

**First/last frame:** you *can* nominate a reference image as first or last frame in the prompt text, but if the frames must strictly match, use the dedicated image-to-video (first and last frame) feature with the `role` parameter set to `first_frame` / `last_frame` instead of relying on prose.

---

## Worked example (official, dialogue-focused)

Assets: `@Image 1` half-body photo of the female lead · `@Image 2` dormitory scene reference · `@Video 1` camera-movement reference for indoor dialogue · `@Audio 1` indoor ambience.

> Use the girl in @Image 1 as the main character, use @Image 2 as the dormitory scene style reference, and refer to the camera movement in @Video 1.
>
> Shot 1: At dusk, girl @Image 1 walks briskly to the dormitory entrance @Image 2. The camera follows steadily in a medium shot. Warm yellow sunlight spills into the hallway from the window. She pauses at the doorway, takes a deep breath, and looks slightly nervous.
>
> Shot 2: Girl @Image 1 pushes the door open and enters the dormitory. The camera cuts to an indoor medium shot. Her roommates look up at her while organizing their books. One of them smiles and asks {How did the exam go? Did you pass?}. The camera slowly cuts between half-body close-ups of several people.
>
> Shot 3: Girl @Image 1 first lowers her head with a dejected expression. The camera gives her a close-up. Then she raises her head, unable to hold back a smile, laughs out loud, and says {I was kidding}. Her roommates start chasing and play-fighting with her. The camera slowly pulls back and freezes on a wide shot of the dormitory filled with laughter.
>
> The entire video should have a high-definition cinematic documentary style, with warm tones and soft lighting. The character's face remains stable without deformation; movements are natural and smooth, with no stutter or flicker. The ambient sound blends naturally with @Audio 1.

Every rule above is visible in it: asset roles declared up front, one camera move per shot, dialogue in `{ }`, emotion externalised into posture, no durations, quality and stability constraints in a closing paragraph.

---

## Extension vs. segmented stitching

Both produce longer output; they are for different content.

- **Continuous long take (video extension)** - single-scene material: long conversations, emotional progression, movement along one path. Gives an immersive one-shot feel.
- **Segmented stitching** - plot turns and fast complex action: chases, fights, montages. Generate independent clips and edit them together for rhythm and impact.

In production the vendor expects both: extend to get a coherent conversation, then stitch in cutaways and transitions.

---

## Failure modes (all from the official FAQ)

### Character ID drift - the face changes mid-video

Symptom: generated character does not match the reference, or "face swaps" partway through - sometimes drifting toward a celebrity likeness and getting blocked in review.

Root causes: a **mixed reference image** (face + pose + outfit + details all in one picture), and **face ratio too small** in that image, so facial features get too little weight and background elements interfere.

Fix:
1. **Add a dedicated headshot** - head only, no expression, minimal shoulders/neck/background - alongside your full-body reference.
2. **Split the roles in the prompt:** `<Subject 1> facial features reference image 1 (headshot), makeup and styling reference image 2 (full-body photo)`.
3. **Put the face reference first.**

⚠️ **Do not use multi-view character sheets on the 2.0 series.** The docs specifically advise against them: different angles of one character are easily read as **several different subjects**, which makes ID drift worse. Headshot + full body is enough. (2.5 supports multi-view subject images.)

### Unwanted subtitles

Cannot be prevented with certainty - only made less likely:
1. Add the constraint words: *"Keep it subtitle-free"*, *"Avoid generating any text or subtitles"*.
2. If a reference image or video contains non-essential text, **strip the text from the asset first** and feed the clean version.
3. **Generate landscape if your business allows** - the docs state subtitle probability is *significantly lower* in landscape than portrait. Crop to portrait afterwards. (Worth knowing if you are producing vertical social video: generate 16:9, crop to 9:16.)

### Logo or watermark from another platform appears

Add explicit constraints: *"Do not generate watermarks"*, *"Do not generate Logos"*.

### Style drift - anime request comes back photoreal

Cause: a realistic reference image plus a prompt that never states the style. Fix: add explicit style constraint words ("2D Japanese anime style", "3D Chinese-style comic"). For tight control, **convert the reference image into the target style first**, then generate.

### Jump cuts at extension seams

Known issue, fixed in post for now. The documented recipe: at each join, **trim 6 frames from the end of the earlier segment and 1 frame from the start of the later segment**, then check smoothness. Slight jumps may remain - so prefer to **end a generated segment on a transition cut** and start the next one on the new scene.

---

## Cost: why a per-clip price you read somewhere is not your bill

Seedance is billed per token, and the charge depends on the **model, resolution, duration, aspect ratio and whether the input contains video** - all of which you set per request. A headline "$X per clip" figure is one specific configuration.

**ModelArk token rates, USD per 1M tokens, online inference** ([pricing](https://docs.byteplus.com/en/docs/modelark/model-pricing), checked 2026-11-06):

| Model | No video input, 480p/720p | No video input, 1080p | No video input, 4K | Video input, 480p/720p | Video input, 1080p | Video input, 4K |
|---|---|---|---|---|---|---|
| `dreamina-seedance-2-5-260628` | $10.70 | $11.70 | - | $6.40 | $7.00 | - |
| `dreamina-seedance-2-0-260128` | $7.00 | $7.70 | $4.00 | $4.30 | $4.70 | $2.40 |
| `dreamina-seedance-2-0-fast-260128` | $5.60 | - | - | $3.30 | - | - |
| `dreamina-seedance-2-0-mini-260615` | $3.50 | - | - | $2.10 | - | - |

Offline inference is not available for any of these.

**Tokens:** (input video duration + output video duration) × output width × output height × frame rate / 1024. The rate with video input is lower, but input seconds count towards tokens, so the video still costs more: the vendor states this directly. A minimum token consumption also applies whenever the input includes video.

**Published examples, 5-second 16:9 output, no video input:**

| Model | 480p | 720p | 1080p | 4K |
|---|---|---|---|---|
| Seedance 2.5 | $0.514 | $1.156 | $2.843 | - |
| Seedance 2.0 | $0.35 | $0.76 | $1.87 | $3.89 |
| Seedance 2.0 Fast | $0.28 | $0.60 | - | - |
| Seedance 2.0 Mini | $0.18 | $0.38 | - | - |

With video input, the same 5-second output on 2.5 runs $0.553-$2.152 at 480p, $1.244-$4.838 at 720p and $3.062-$11.907 at 1080p, depending on input length (2-30 s). Only successful generations are charged. Actual consumption is `usage.completion_tokens` in the response.

- **Draft mode exists on Seedance 2.5 only** among these models: a 480p preview at the normal 480p rate, whose task ID then generates the final video (valid 14 days). The 2.0 series has no Draft mode, so iterate there at 480p.
- **Aggregators** set their own prices and may not expose video pricing in their API metadata at all. Their model list endpoint may also omit video models unless you filter for video output explicitly.

Practical rule: **iterate at 480p or 720p, read the actual token count after a real call, and only then extrapolate a budget.** On aggregators a model with a lower headline rate can end up costing more than a pricier one once resolution and duration are set.

---

## Verification & Sources

Verified **2026-11-06** against ByteDance/BytePlus-controlled documentation:

- **Seedance 2.5 tutorial** (model ID, capability table, task constraints, usage limits; updated 2026-10-28): https://docs.byteplus.com/en/docs/modelark/seedance-2-5
- **Seedance 2.5 prompt guide** (locked/unlocked tasks, timestamps, differences from 2.0, FAQ; updated 2026-10-16): https://docs.byteplus.com/en/docs/modelark/seedance-2-5-prompt-guide
- **Seedance 2.0 series prompt guide** (the primary source for Steps 1-7; updated 2026-09-28): https://docs.byteplus.com/en/docs/modelark/seedance-2-0-prompt-guide
- **Seedance 2.0 series tutorial** (model IDs, capability matrix, asset limits, rate limits, 4K notes; updated 2026-11-05): https://docs.byteplus.com/en/docs/modelark/seedance-2-0
- **Model list** (IDs, outputs, rate limits; updated 2026-11-03): https://docs.byteplus.com/en/docs/modelark/model-list
- **Pricing** (token rates, formula, examples; updated 2026-10-23): https://docs.byteplus.com/en/docs/modelark/model-pricing
- **Portrait assets guide**: https://docs.byteplus.com/en/docs/modelark/seedance-portrait-asset-guide
- **Lens language reference** (camera vocabulary; documented for the Seedance 1.0 Pro generation and linked from the 2.0 guide): https://docs.byteplus.com/en/docs/ModelArk/1631633
- **Seedance 1.5 Pro prompt guide** (previous generation): https://docs.byteplus.com/en/docs/ModelArk/2168087
- **Official product site:** https://seed.bytedance.com

These pages are JavaScript-rendered; a plain fetch returns an empty shell. Use a headless browser, and wait on `domcontentloaded` plus a fixed delay: the pages never reach network idle, so a `networkidle` wait times out. Pricing tabs must be clicked to reveal the "Input with video" rates. ByteDance also publishes its own prompt-optimisation skills, `sd25-pe` for 2.5 and `sd2-pe` for 2.0.

**Deliberately not claimed here:**
- The date Seedance 2.5 opened. The docs show the API as live but give no release date, and `260628` in the ID is not one.
- Prices on aggregators or on resource packs. The rates above are ModelArk list rates.
- A lip-sync language list for the 2.0 series. Native audio and lip-sync are documented, and the docs give a rule for marking non-Chinese/English dialogue, but no supported-language list is published in those pages. The 11-language list applies to 2.5.
- Polish lip-sync on any Seedance model.
- Benchmark comparisons against other video models.
- Exact per-shot duration control on the 2.0 series - explicitly documented as unstable.
- Any claim that subtitles, logos or watermarks can be fully prevented. The docs say probability can only be reduced, on 2.5 as well.

If you find a claim above is wrong, fix it here and in the body section, with the source URL and the date you checked.
