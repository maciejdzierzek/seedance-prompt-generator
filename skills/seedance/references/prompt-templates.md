# Seedance 2.0 prompt templates

Fill-in templates built on the official 2.0 series formula and sentence patterns. Every template ends with a stability/constraint paragraph - the docs treat those words as necessary configuration, not polish. **For Seedance 2.5 read "Seedance 2.5: what differs" in `SKILL.md` first:** it adds timestamps, fixed parameters for edit and extend, and more assets.

Reminders that apply to all of them:
- **One camera movement per shot.**
- **No durations per shot on the 2.0 series** - precise timing is documented as unstable. 2.5 supports timestamps.
- **Prefer slow, gentle, continuous small movements** over big dynamic action.
- Symbols: `{ }` dialogue · `< >` sound effect · `（ ）` music · `【 】` subtitles.
- For **edit** and **extend** tasks say `<Video 1>` / "strictly edit `<Video 1>`" - **never "reference `<Video 1>`"**.

---

## A. Standard stability tail

Append to any prompt:

```
The entire video should be high-definition with cinematic texture, natural colors and soft lighting.
[Style statement]. The character's face remains stable without deformation; movements are natural and
smooth, with no stutter or flicker. Keep it subtitle-free. Do not generate a logo or watermark.
```

---

## B. Text-to-video, single scene

```
[Shot size and camera movement]. [Subject with 2-3 identifying features] [action, refined to body
parts with speed and force]. [Setting and one or two supporting elements]. [Light direction, quality,
colour temperature]. < [ambient sound] >. （ [music character] ）
[Standard stability tail]
```

## C. Multi-shot narrative from references

```
Use [subject] in @Image 1 as the main character, use @Image 2 as the [scene/style] reference,
and refer to the camera movement in @Video 1.

Shot 1: [camera move]. [Subject]@Image 1 [action with body-part detail]. [Position/space].
        [Light]. < [sfx] >
Shot 2: [camera cut or move]. [Action]. [Second subject reaction]. One of them says
        {[dialogue]}. [Position/space].
Shot 3: [camera move]. [Emotional beat externalised into posture and face]. Says {[dialogue]}.
        [Camera resolves: pulls back / freezes on ...].

[Standard stability tail] The ambient sound blends naturally with @Audio 1.
```

## D. Character definition block (put this first when identity matters)

```
Define [2-3 stable features] in @Image 1 as [Label].
<[Label]> facial features reference image 1 (headshot), makeup and styling reference image 2 (full-body photo).
```

Then use `[Label]` consistently, or bind on each mention as `[Label]@Image 1`. Headshot first, and **never a multi-view character sheet** - the model may read the angles as different people.

## E. Multimodal reference (new video from asset elements)

```
Reference [Subject 1] in <Image 1> to generate ...
Reference the [action / camera movement / style / sound effect] in <Video 1> to generate ...
Reference the timbre in <Audio 1> to generate ...

[Scene, action, light, camera per shot]
[Standard stability tail]
```

## F. Video editing

```
Strictly edit <Video 1>, and modify [original characteristic] in it to [new characteristic].
Keep [everything that must not change: camera angle, motion path, spatial layout, positioning] unchanged.
[If compositing something in:] Ensure the lighting and shadows on the subject match the target
environment, with a natural, non-composited look and no sticker-like artifacts.
```

Object swap with a reference:
```
Replace the [object] in <Video 1> with the [object] from <Image 1>. [How it should sit and behave
in the scene]. Keep all other elements of the video unchanged.
```

Attribute change:
```
Strictly edit <Video 1>. Change [attribute 1] to [value], and [attribute 2] to [value].
Refer to <Image 1> for the expected result. Keep all other elements unchanged.
```

Deletion:
```
Strictly edit <Video 1>. Remove [element]. [Describe what should be visible in its place].
Keep [elements that must stay] exactly as they are.
```

One edit intent per generation. Anything unmentioned stays as it is by default - but say it anyway for anything you care about.

## G. Video extension

```
Extend <Video 1> forward to generate: [continuation of the action, same subject, same light,
same style]. [Camera continues / holds]. < [continuing ambience] >
[Standard stability tail]
```

Track completion across clips:
```
<Video 1> + [transition description] + followed by <Video 2> + [transition description] + followed by <Video 3>
```

**Post-production for seams:** trim 6 frames off the end of the earlier segment and 1 frame off the start of the next, at every join. Better still, end each generated segment on a transition cut.

## H. Combined reference + edit

```
Reference [reference dimension] of <Image 1>, strictly edit <Video 1>, [specific edits].
Keep [unchanged list].
```

## I. Vertical social video

Counter-intuitive but documented: subtitle bleed-through is **significantly more likely in portrait**. If your pipeline allows it, generate **16:9 and crop to 9:16** in post.

```
[Prompt as normal, 16:9]
... Keep it subtitle-free. Avoid generating any text or subtitles. Do not generate a logo or watermark.
Compose action and faces within the central vertical third of the frame so the shot survives a 9:16 crop.
```

## J. Dialogue and audio cheat sheet

```
{Hello, how did it go?}                     dialogue, drives lip-sync
says in Polish {Dzień dobry}                non-CN/EN dialogue - mark the language
< footsteps on gravel >                     sound effect
（slow piano, sparse and melancholy）        music
【Chapter One: Departure】                   on-screen subtitle you actually want
```

Keep dialogue in one language per video; do not mix Chinese and English except for proper nouns.

## K. Asset plan on the 2.0 series (4-5 assets, not the maximum)

| Slot | Asset | Purpose |
|---|---|---|
| 1 | character headshot | lock the face - **place first** |
| 2 | character full body | wardrobe and styling |
| 3 | scene image | environment and tone |
| 4 | camera-movement video | shot language and rhythm |
| 5 | audio clip | emotion, timbre, pace |

Hard limits on the 2.0 series are 0-9 images, 0-3 videos, 0-3 audio, but the vendor advises against filling them - too many assets and the model cannot prioritise features. "Text + audio" and audio-only inputs are not supported.

## L. Iteration ladder (spend little until the prompt is right)

1. **480p**, shortest usable duration - is the beat readable? On Seedance 2.5 use Draft mode, which is a 480p preview you can promote to a final. The 2.0 series has no Draft mode.
2. **720p** - do faces hold, does the camera behave?
3. Final resolution and duration. **4K only on Seedance 2.0**, 10-bit H.265, concurrency 1 - transcode for review. Seedance 2.5 stops at 1080p.

Read the token count the provider actually reports after a real call before extrapolating any budget.
