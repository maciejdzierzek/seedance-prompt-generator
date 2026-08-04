---
name: seedance
description: Write and improve prompts for ByteDance Seedance 2.0 video generation, and pick the right variant. Covers the four task types (multimodal reference, video editing, video extension, combined), the shot-by-shot storyboard format the model expects, subject definition and binding to reference assets, the official symbol conventions for dialogue, sound effects, music and subtitles, camera language, constraint words, asset selection strategy, and fixes for character ID drift, unwanted subtitles and style drift.
when_to_use: Use for anything involving Seedance, Seedance 2.0, Seedance 2.0 Fast, Seedance 2.0 Mini, Seedance 1.5 Pro, Dreamina, ByteDance video generation, ModelArk video models, or the model IDs dreamina-seedance-2-0-* and bytedance/seedance-*. Trigger phrases include "Seedance", "generate a video with audio", "multi-shot AI video", "edit an existing video with AI", "extend this video", "lip sync video", "character keeps changing face in my video", "unwanted subtitles in generated video", "video prompt".
---

# Seedance 2.0 Prompting Guide

Seedance 2.0 is ByteDance's video generation family (branded **Dreamina Seedance** on BytePlus ModelArk). It generates **audio and video jointly and natively** - not audio added afterwards - and accepts text, images, video and audio as simultaneous input.

Facts here are verified against ByteDance's own ModelArk documentation on **2026-08-05**. See "Verification & Sources".

**The mental model that makes this work** (from the official guide): Seedance 2.0 is a multimodal director that decomposes your input into a **spatial layer** (what is in the frame) and a **temporal layer** (how things change over time). So a good prompt is *"not simply a copywriting-style description, but an engineering-style instruction"*: who, in what scene, doing what, how the camera moves, and in what order events happen - each delivered to the right layer.

---

## Model lineup

| | **Seedance 2.0** | **Seedance 2.0 Fast** | **Seedance 2.0 Mini** |
|---|---|---|---|
| ModelArk model ID | `dreamina-seedance-2-0-260128` | `dreamina-seedance-2-0-fast-260128` | `dreamina-seedance-2-0-mini-260615` |
| Positioning (vendor's words) | highest generation quality | balance of cost and speed | best cost performance |
| Resolution | 480p, 720p, 1080p, **4K** | 480p, 720p | 480p, 720p |
| Aspect ratios | 21:9, 16:9, 4:3, 1:1, 3:4, 9:16 | same | same |
| Duration | **4-15 s** | 4-15 s | 4-15 s |
| Text-to-video, image-to-video (first frame, first+last frame) | ✅ | ✅ | ✅ |
| Multimodal reference (image / video / combined) | ✅ | ✅ | ✅ |
| Edit video, extend video | ✅ | ✅ | ✅ |
| Native audio (`generate_audio`) | ✅ | ✅ | ✅ |
| Draft mode, `return_last_frame`, `service_tier: flex` | ✅ | ✅ | ✅ |

**The feature sets are essentially identical.** The three variants differ in quality/cost and in the resolution ceiling. Only **Seedance 2.0** does 4K, and 4K output is **10-bit H.265 (HEVC)** - beautiful gradients, but some players and browsers will not play it back directly, so plan a transcode for review.

Rate limits are much tighter at 4K: individual users get **15 RPM and concurrency 1** at 4K, versus 180 RPM and concurrency 3 below 4K. A 4K batch is serialised whether you plan for it or not.

**Seedance 2.5** has been announced on ModelArk with a model detail page, but Playground and API access were *"available soon"* as of 2026-08-05. Do not build around it yet. **Seedance 1.5 Pro** is the previous generation and has its own separate prompt guide - if a user is on 1.5, the formulas below still broadly apply but check that guide for specifics.

**Access routes:** BytePlus ModelArk directly (prepaid resource packs required before the models can even be activated), or third-party aggregators which expose their own IDs such as `bytedance/seedance-2.0` and `bytedance/seedance-2.0-fast`. See Cost at the bottom - the two routes bill differently.

---

## Step 1: pick the task type, because it changes the sentence pattern

Reference-based generation splits into three tasks plus combinations, and **each has its own official sentence pattern**. Using the wrong one is the most common structural mistake.

### Multimodal reference - build a new video from elements of your assets

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

**Do not assign durations to shots.** The docs are explicit: support for precise timing such as "0-3 seconds" is **unstable, and forcing it may produce abnormal results**. Let pacing follow the plot. Order shots primary-first.

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

Hard input limits: **0-9 images, 0-3 videos, 0-3 audio files**. Also: **"text + audio" and "audio-only" inputs are not supported** - if you pass audio, pass an image or video with it.

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

⚠️ **Do not use multi-view character sheets.** The docs specifically advise against them: different angles of one character are easily read as **several different subjects**, which makes ID drift worse. Headshot + full body is enough.

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

Seedance is billed by the provider, and the charge depends on the **model variant, resolution, duration, aspect ratio and whether audio is generated** - all of which you set per request. A headline "$X per clip" figure is a base rate for one specific configuration.

- **ModelArk** requires **prepaid resource packs** before the models can be activated at all, and offers an offline/cheaper path via `service_tier: "flex"` plus a **draft mode** for iteration.
- **Aggregators** set their own prices and may not expose video pricing in their API metadata at all. Their model list endpoint may also omit video models unless you filter for video output explicitly.

Practical rule: **iterate in draft mode or at 480p/720p, read the actual cost field the provider returns after a real call, and only then extrapolate a budget.** A single measurement at one configuration is not a price list - and a model with a lower headline rate can end up costing more than a pricier one once resolution and duration are set.

---

## Verification & Sources

Verified **2026-08-05** against ByteDance/BytePlus-controlled documentation:

- **Seedance 2.0 series prompt guide** (the primary source for everything above): https://docs.byteplus.com/en/docs/ModelArk/2222480
- **Seedance 2.0 series tutorial** (model IDs, capability matrix, resolutions, ratios, durations, asset limits, rate limits, 4K notes): https://docs.byteplus.com/en/docs/ModelArk/2291680
- **Lens language reference** (camera vocabulary; documented for the Seedance 1.0 Pro generation and linked from the 2.0 guide): https://docs.byteplus.com/en/docs/ModelArk/1631633
- **Seedance 1.5 Pro prompt guide** (previous generation): https://docs.byteplus.com/en/docs/ModelArk/2168087
- **Official product site:** https://seed.bytedance.com

These pages are JavaScript-rendered; a plain fetch returns an empty shell. Use a headless browser to re-verify.

**Deliberately not claimed here:**
- Any specific per-second or per-clip price. Pricing is provider-specific and configuration-dependent; the docs gate ModelArk access behind resource packs whose rates are on a separate page.
- A lip-sync language count. Native audio and lip-sync are documented capabilities, and the docs give a rule for marking non-Chinese/English dialogue, but no supported-language list is published in these pages.
- Availability of Seedance 2.5 - announced with a model detail page, but Playground and API were "available soon" as of 2026-08-05.
- Benchmark comparisons against other video models.
- Exact per-shot duration control - explicitly documented as unstable.
- Any claim that subtitles, logos or watermarks can be fully prevented. The docs say probability can only be reduced.

If you find a claim above is wrong, fix it here and in the body section, with the source URL and the date you checked.
