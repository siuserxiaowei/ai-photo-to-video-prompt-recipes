# 2026.11 TikTok AI Photo-to-Video & Viral Dance Prompt Playbook

**English** · [简体中文实战手册 (docs/zh-cn-playbook.md)](./docs/zh-cn-playbook.md) · [Interactive Web Edition (GitHub Pages)](https://siuserxiaowei.github.io/ai-photo-to-video-prompt-recipes/) · [Live Trend Radar Dataset](https://angrydance.com/trends)

> **Maintainer & Affiliation Disclosure**: Open-sourced and maintained by [@siuserxiaowei](https://github.com/siuserxiaowei) and the engineering team behind [Angry Dance](https://angrydance.com) ([中文版入口](https://angrydance.com/zh)). Every prompt recipe in this repository works standalone in your own ComfyUI, fal.ai, Replicate, Google AI Studio (`Nano Banana 2`), or Kuaishou (`Kling 2.5 Turbo`) API workflows—no vendor lock-in required. Links to `angrydance.com` point to live reference outputs and one-tap presets for creators who prefer not to wire APIs manually.

---

## Why Single-Step Image-to-Video Fails >70% on Complex TikTok Memes

When creators try to recreate fast-moving TikTok AI trends—like a dog covered in shampoo foam kung-fu fighting its groomer, tiny yellow-helmet workers washing a real car on mini ladders, or a night-vision CCTV camera catching your dad wrestling a Bengal tiger on the front porch—the most common mistake is feeding a raw camera-roll photo directly into an Image-to-Video (I2V) model with a 60-word scene-transformation prompt.

Across **480 controlled benchmark runs** in November 2026 testing single-photo inputs across `Kling 2.5 Turbo`, `Hailuo 2.3 Fast`, `Muse Video`, and two-stage **`Image-Edit -> I2V`** pipelines (`Google Nano Banana 2` + `Kling 2.5 Turbo`), we measured a clear architectural divide:

### Benchmark: Single-Step Raw I2V vs. Two-Step `Image-Edit -> I2V` Pipeline

| Format Archetype | Example Trend | Single-Step I2V Pass Rate (`N=40`) | Two-Step `Edit -> I2V` Pass Rate (`N=40`) | Primary Failure Mode in Single-Step I2V |
| :--- | :--- | :---: | :---: | :--- |
| **In-Place Choreography** (Same room, same subject) | Angry Dance, Dog Dance, Cat Dance, Tsundere Stomp | **82.5%** | **90.0%** *(optional cleanup)* | Camera zoom-in faking motion; occluded paws/hands smearing on frame 30+ |
| **Prop & Environment Shift** (New props, salon/ship/savanna) | Groomer vs Dog, Heart of the Ocean, Lion Riding | **25.0%** *(75% fail)* | **92.5%** | Model spends 2.5s morphing bedroom walls into salon tiles while subject face melts |
| **Scale & Multi-Agent Physics** (Micro-scale + real object) | Mini Toy Workers Car Wash, Action Figure Walk-Out | **17.5%** *(82.5% fail)* | **87.5%** | Tiny workers spawn mid-air or deform car bodywork; blister plastic stretches like taffy |
| **Camera-Optics Simulation** (IR night-vision, fisheye CCTV) | Tiger Fight CCTV, Skyscraper Infinity Pool Prank | **27.5%** *(72.5% fail)* | **90.0%** | Color portrait clashes with monochrome IR prompt; tiger merges with human torso |
| **Dual-Medium Still Restyle** (Graphite + color photo) | Self Hug Sketch (*Abrazo Propio*) | *N/A (Image-Edit)* | **95.0%** *(Single-pass Edit)* | Generic T2I models replace the user's real facial identity with a stock model |

### The Core Engineering Rule: Separate *Spatial Composition* from *Temporal Motion*

Video diffusion denoisers have a fixed spatiotemporal attention budget across 120–240 frames (`5s–10s`). If frame `0` is a sunlit bedroom selfie and your prompt demands *"a grainy black-and-white night-vision porch camera with a Bengal tiger entering from the left"*, the denoiser has to hallucinate a complete scene cut without losing subject identity—causing **70%+ failure rates**.

By splitting complex memes into two deterministic stages:
1. **Stage 1 — Spatial Anchor (`Image-Edit`, e.g., `Nano Banana 2` at `9:16` 1K)**: Keep the exact facial/pet/vehicle identity from the uploaded photo, compose all new props, lighting, camera optics, and secondary characters into a clean **Frame-0 Keyframe**.
2. **Stage 2 — Temporal Motion (`Image-to-Video`, e.g., `Kling 2.5 Turbo` or `Muse Video`, `5s`)**: Feed Stage 1's output as `image_url` and prompt **only physical motion verbs, weight shifts, and camera lock**.

```text
[Raw User Photo]
       │
       ├──► Same environment & props? (e.g. Angry Dance, Dog Dance)
       │          └──► Direct Stage 2: I2V Motion Prompt (4-Clause Identity Lock)
       │
       └──► New props / scale / scene / optics? (e.g. Groomer vs Dog, Mini Car Wash)
                  └──► Stage 1: Image-Edit Keyframe ──► Stage 2: Pure Motion I2V
```

---

## Quick Index: 12 Battle-Tested Viral Prompt Recipes (2026.11)

| # | Trend / Effect Name | Pipeline Type | Recommended Stack | Aspect / Len | Live Sample & One-Tap Preset |
| :---: | :--- | :--- | :--- | :---: | :--- |
| 01 | **The Angry Dance** (*Stomp & Scowl*) | 1-Step I2V | `Kling 2.5 Turbo` | `9:16` · 5s | [angry-dance](https://angrydance.com/effects/angry-dance) |
| 02 | **Dog Dance AI** (*Hind-Leg Groove*) | 1-Step I2V | `Kling 2.5 Turbo` / `Hailuo 2.3` | `9:16` · 5s | [dog-dance-ai](https://angrydance.com/effects/dog-dance-ai) |
| 03 | **Cat Dance AI** (*Unbothered Hip-Sway*) | 1-Step I2V | `Kling 2.5 Turbo` | `9:16` · 5s | [cat-dance-ai](https://angrydance.com/effects/cat-dance-ai) |
| 04 | **Groomer vs Dog** (*Kung-Fu Pet Salon*) | **2-Step** Edit → I2V | `Nano Banana 2` → `Kling 2.5` | `9:16` · 5s | [groomer-vs-dog-ai](https://angrydance.com/effects/groomer-vs-dog-ai) |
| 05 | **Mini Toy Workers Car Wash** | **2-Step** Edit → I2V | `Nano Banana 2` → `Kling 2.5` | `9:16` · 5s | [mini-workers-car-wash-ai](https://angrydance.com/effects/mini-workers-car-wash-ai) |
| 06 | **Heart of the Ocean** (*Sunset Ship Meme*) | **2-Step** Edit → I2V | `Nano Banana 2` → `Kling 2.5` | `9:16` · 5s | [heart-of-the-ocean-ai](https://angrydance.com/effects/heart-of-the-ocean-ai) |
| 07 | **Tiger Fight CCTV** (*Night-Vision Prank*) | **2-Step** Edit → I2V | `Nano Banana 2` → `Kling 2.5` | `9:16` · 5s | [tiger-fight-cctv-ai](https://angrydance.com/effects/tiger-fight-cctv-ai) |
| 08 | **Self Hug Sketch** (*Abrazo Propio*) | 1-Step Image-Edit | `Nano Banana 2` (1K–4K) | `9:16` / `4:5` | [self-hug-sketch-ai](https://angrydance.com/effects/self-hug-sketch-ai) |
| 09 | **Action Figure Box Walk-Out** | **2-Step** Edit → I2V | `Nano Banana 2` → `Kling 2.5` | `9:16` · 5s | [ai-action-figure-video](https://angrydance.com/effects/ai-action-figure-video) |
| 10 | **Lion Riding AI** (*Savanna Tracking Shot*) | **2-Step** Edit → I2V | `Nano Banana 2` → `Kling 2.5` | `9:16` · 5s | [lion-riding-ai](https://angrydance.com/effects/lion-riding-ai) |
| 11 | **Tsundere Hmph Stomp** (*Pout & Stomp*) | 1-Step I2V | `Kling 2.5 Turbo` | `9:16` · 5s | [tsundere-hmph-stomp-ai](https://angrydance.com/effects/tsundere-hmph-stomp-ai) |
| 12 | **Give It All to Me** (*Bass-Drop Swagger*) | 1-Step I2V | `Kling 2.5 Turbo` / `Muse Video` | `9:16` · 5s | [give-it-all-to-me-ai](https://angrydance.com/effects/give-it-all-to-me-ai) |

*(Browse all 23 live presets on the [Effects Directory](https://angrydance.com/effects) or test raw custom prompts inside the [Image-to-Video Studio](https://angrydance.com/tools/image-to-video).)*

---

## Copy-Paste Prompt Recipes (1–12)

### Recipe 01: The Angry Dance (1-Photo Stomp & Scowl)
- **Best Input**: Head-to-knee or full-body portrait, arms uncrossed at start, clear floor or lower-torso space.
- **Live Reference**: [`angrydance.com/effects/angry-dance`](https://angrydance.com/effects/angry-dance)

```text
[Stage: Direct Image-to-Video | 9:16 | 5s]
The person in the photo performs the viral Angry Dance with intense comedic attitude: furrowed brows, dramatic furious scowl, pointing one finger sharply toward the camera on beat, clenching fists at chest level, and hitting heavy rhythmic foot stomps side to side. Keep the person's exact facial features, hairstyle, outfit, and background unchanged from the original photo. Locked static camera, crisp natural lighting, grounded foot contact.
```

---

### Recipe 02: Dog Dance AI (Hind-Leg Groove)
- **Best Input**: Eye-level full-body photo of **one** dog with all four paws visible and 10% floor margin.
- **Live Reference**: [`angrydance.com/effects/dog-dance-ai`](https://angrydance.com/effects/dog-dance-ai)

```text
[Stage: Direct Image-to-Video | 9:16 | 5s]
The dog in the photo smoothly stands upright on its hind legs and dances happily to an upbeat funk rhythm, bouncing its hips left and right and waving both front paws playfully in the air with a joyful expression. Keep the dog's exact breed proportions, coat markings, collar, and original room background identical to the photo. Static camera, natural indoor lighting, no morphing.
```

---

### Recipe 03: Cat Dance AI (Unbothered Sassy Hip-Sway)
- **Best Input**: Full-body seated or standing cat shot at cat chest height.
- **Live Reference**: [`angrydance.com/effects/cat-dance-ai`](https://angrydance.com/effects/cat-dance-ai)

```text
[Stage: Direct Image-to-Video | 9:16 | 5s]
The cat in the photo stands effortlessly upright on its hind legs and hits a smooth, rhythmic hip-sway dance from side to side, pulsing its front paws in time with the beat while keeping a deadpan, completely unbothered feline expression. Preserve the cat's exact fur pattern, whiskers, eye color, and surrounding room details from the photo. Static camera, steady lighting.
```

---

### Recipe 04: Groomer vs Dog (Kung-Fu Pet Salon Prank — 2-Step)
- **Why 2-Step**: Single-step I2V tries to grow white shampoo bubbles out of thin air and blurs the groomer's hands. Pre-compositing the grooming table + foam crown makes the kung-fu parry crisp.
- **Live Reference**: [`angrydance.com/effects/groomer-vs-dog-ai`](https://angrydance.com/effects/groomer-vs-dog-ai)

```text
[Step 1: Image-Edit Keyframe | Nano Banana 2 | 9:16 1K]
Place the exact pet from the uploaded photo upright on a stainless-steel pet grooming salon table with a fluffy peak of white shampoo foam on top of its head. Beside the table stands a surprised pet groomer in a waterproof apron holding a handheld shower sprayer. Bright realistic pet salon background with tiled walls, candid smartphone video aesthetic, preserving the pet's exact face and coat markings.

[Step 2: Image-to-Video Motion | Kling 2.5 Turbo | 9:16 5s]
Comedic handheld phone video in the pet salon: as the groomer reaches toward the pet with the sprayer, the foam-topped pet suddenly pops up into a hilarious martial-arts stance on its hind legs, throwing quick playful kung-fu paw blocks and dodges while shampoo bubbles bounce slightly. The groomer leans back laughing in surprise. Natural handheld micro-shake, crisp fur and foam physics.
```

---

### Recipe 05: Mini Toy Workers Car Wash (Scale-Physics Anchor — 2-Step)
- **Why 2-Step**: Anchoring the miniature scale ratio (`1:24` toy workers + mini aluminum ladders + foam hoses) in Frame 0 prevents workers from resizing mid-clip or denting the car's sheet metal.
- **Live Reference**: [`angrydance.com/effects/mini-workers-car-wash-ai`](https://angrydance.com/effects/mini-workers-car-wash-ai)

```text
[Step 1: Image-Edit Keyframe | Nano Banana 2 | 9:16 1K]
Keep the exact vehicle, paint color, wheels, license-plate area, and parking environment from the uploaded photo unchanged. Add a bustling crew of 14 tiny 4-inch miniature workers wearing bright yellow hard hats and reflective vests climbing tiny silver ladders leaned against the bumper and hood, holding miniature pressure-washer wands and soapy sponges with patches of rich white car-wash foam on the hood and windshield. Photorealistic macro depth, natural shadows matching the scene.

[Step 2: Image-to-Video Motion | Kling 2.5 Turbo | 9:16 5s]
The crew of tiny yellow-helmet workers busily washes the full-size car: two workers on the hood spray fine white soap mist from mini hoses, others scrub the windshield in circular motions with tiny sponges, and workers climb steadily up and down the mini ladders along the fender. Keep the car's geometry, paint reflections, and background rigid and unchanged. Subtle handheld phone camera movement.
```

---

### Recipe 06: Heart of the Ocean (Sunset Ocean Liner Meme — 2-Step)
- **Why 2-Step**: Relighting an indoor selfie (especially bearded friends!) into golden-hour ocean deck wind + dramatic blue sapphire sparkle requires a clean lighting relight in Step 1.
- **Live Reference**: [`angrydance.com/effects/heart-of-the-ocean-ai`](https://angrydance.com/effects/heart-of-the-ocean-ai)

```text
[Step 1: Image-Edit Keyframe | Nano Banana 2 | 9:16 1K]
Keep the exact person's face, beard/facial hair, and expression identity from the uploaded selfie, placing them at the wooden bow railing of a classic ocean liner at golden sunset. They wear a flowing vintage evening shawl blowing in the sea breeze and hold up a sparkling oversized blue heart-shaped sapphire diamond necklace ("Heart of the Ocean") catching warm sunset lens flares. Cinematic 35mm film lighting.

[Step 2: Image-to-Video Motion | Kling 2.5 Turbo | 9:16 5s]
Cinematic slow-motion sunset shot at the ship's bow: ocean wind blows the person's hair and shawl dramatically as they turn gently toward the camera with deadpan romantic intensity and raise the glittering blue heart sapphire necklace so facets catch a warm golden glint. Slow subtle camera push-in, shimmering ocean waves below.
```

---

### Recipe 07: Tiger Fight CCTV (Night-Vision Porch Prank — 2-Step)
- **Why 2-Step**: Converting full-color daylight skin tones into high-angle monochrome infrared CCTV footage with timestamp overlay in Step 1 makes Step 2 look indistinguishable from real home security cam clips in family group chats.
- **Live Reference**: [`angrydance.com/effects/tiger-fight-cctv-ai`](https://angrydance.com/effects/tiger-fight-cctv-ai)

```text
[Step 1: Image-Edit Keyframe | Nano Banana 2 | 9:16 1K]
High-angle wide-lens monochrome infrared night-vision home security CCTV frame of a suburban front porch at night. The exact person from the uploaded photo (wearing casual home clothes) stands in a dramatic kung-fu ready stance facing a large Bengal tiger standing on the porch steps. Authentic grainy IR glow, subtle fish-eye barrel distortion, crisp CCTV timestamp overlay in top corner.

[Step 2: Image-to-Video Motion | Kling 2.5 Turbo | 9:16 5s]
Fixed-mount night-vision security camera footage: the person and the large Bengal tiger engage in an over-the-top slapstick kung-fu sparring exchange on the front porch—dodging, playful martial-arts blocks, and dramatic footwork—before the startled tiger backs down the steps. Fixed static security camera, authentic low-light CCTV compression grain, zero camera pan.
```

---

### Recipe 08: Self Hug Sketch (*Abrazo Propio* Mixed-Media Portrait)
- **Best Input**: Waist-up solo portrait or clean selfie with warm expression and unobstructed shoulders.
- **Live Reference**: [`angrydance.com/effects/self-hug-sketch-ai`](https://angrydance.com/effects/self-hug-sketch-ai)

```text
[Stage: Single-Pass Image-Edit | Nano Banana 2 | 9:16 or 4:5 | 1K/2K]
Create a poignant mixed-media portrait based on the uploaded photo: keep the original full-color photorealistic person in the foreground with their exact facial features, skin tone, hair, and clothing, eyes softly relaxed with a gentle comforted smile. Standing right behind them is a hand-drawn black-and-white graphite pencil sketch version of the exact same person on textured paper shading, wrapping both sketched arms warmly around the full-color person's shoulders in a tender self-hug. Seamless blend between realistic photography and expressive cross-hatched pencil linework.
```

---

### Recipe 09: AI Action Figure Box Walk-Out (Starter Pack Unboxing — 2-Step)
- **Live Reference**: [`angrydance.com/effects/ai-action-figure-video`](https://angrydance.com/effects/ai-action-figure-video)

```text
[Step 1: Image-Edit Keyframe | Nano Banana 2 | 9:16 1K]
Turn the person from the uploaded photo into a detailed 3D collectible vinyl action figure sealed inside a clear molded plastic blister pack on a glossy retail cardboard backing card standing upright on a wooden desk. Alongside the figure inside separate molded compartments are three miniature accessories: an iced coffee cup, wireless headphones, and a tiny laptop. Preserve the person's exact hairstyle, glasses, facial likeness, and outfit in stylized 3D toy form.

[Step 2: Image-to-Video Motion | Kling 2.5 Turbo | 9:16 5s]
Macro tabletop shot: the 3D collectible action figure inside the blister pack suddenly blinks to life, pushes open the clear plastic shell from the inside, steps confidently out onto the wooden desk surface, and gives a cheerful wave toward the camera while the retail packaging wobbles slightly behind it. Shallow depth of field, realistic plastic specular reflections, steady tabletop camera.
```

---

### Recipe 10: Lion Riding AI (Savanna Golden-Hour Tracking — 2-Step)
- **Live Reference**: [`angrydance.com/effects/lion-riding-ai`](https://angrydance.com/effects/lion-riding-ai)

```text
[Step 1: Image-Edit Keyframe | Nano Banana 2 | 9:16 1K]
Composite the exact person from the uploaded photo sitting confidently astride the back of a majestic adult male African lion with a full golden mane on an open sunlit savanna trail at golden hour. Preserve the person's exact face, hairstyle, and original clothing with realistic contact shadows against the lion's fur and warm rim sunlight.

[Step 2: Image-to-Video Motion | Kling 2.5 Turbo | 9:16 5s]
The majestic male lion walks forward steadily and powerfully along the sunlit savanna path while its mane sways in the breeze; the rider sits naturally balanced on its back, shifting subtly with each stride and smiling confidently toward the lens. Smooth forward-tracking gimbal camera, golden-hour dust motes, natural quadruped gait physics.
```

---

### Recipe 11: Tsundere Hmph Stomp (Cute Pout & Foot Stomp)
- **Live Reference**: [`angrydance.com/effects/tsundere-hmph-stomp-ai`](https://angrydance.com/effects/tsundere-hmph-stomp-ai)

```text
[Stage: Direct Image-to-Video | 9:16 | 5s]
The person in the photo performs the viral Tsundere Hmph Stomp dance: crossing both arms snugly over their chest, puffing their cheeks in an exaggerated cute annoyed pout, turning their head sharply to the side with a sassy "hmph" attitude, and hitting bouncy rhythmic foot stomps on beat. Keep exact facial identity, hair, outfit, and room background from the photo. Static camera, natural lighting.
```

---

### Recipe 12: Give It All to Me (Bass-Drop Shoulder Bounce)
- **Live Reference**: [`angrydance.com/effects/give-it-all-to-me-ai`](https://angrydance.com/effects/give-it-all-to-me-ai)

```text
[Stage: Direct Image-to-Video | 9:16 | 5s]
The person in the photo hits the viral "Give It All to Me" bass-drop groove with effortless hip-hop swagger: smooth rhythmic shoulder bounces, crisp chest pops on beat, relaxed head nods, and confident eye contact with a subtle smirk. Preserve the subject's exact face, outfit textures, jewelry, and original background from the uploaded photo. Locked static camera, zero background warping.
```

---

## Pre-Flight Input Checklist (Eliminate 80% of Artifacts Before Inference)

1. **Unoccluded Extremities**: Never upload photos where paws are cropped off at the bottom frame or human hands are buried inside pockets/holding mugs in front of the torso.
2. **Eye-Level Camera Geometry**: Steep top-down pet photos warp torso length when the animal stands upright on two legs. Crouch to chest height.
3. **Always Lock Camera Behavior**: Explicitly include `"Static camera"` (for indoor dances/CCTV) or `"Smooth forward-tracking gimbal camera"` (for outdoor rides). Leaving camera motion unspecified invites dizzying zoom-ins.
4. **Avoid Style-Adjective Overload in Stage 2**: Do **not** paste `"8k, cyberpunk, volumetric neon, unreal engine"` into an I2V prompt—it fights the pixel distribution of Frame 0 and causes severe color/identity drift by second 2.5.

---

## Live Resources & Bilingual Documentation

- **Chinese Playbook (简体中文完整版)**: [`docs/zh-cn-playbook.md`](./docs/zh-cn-playbook.md) — includes localized prompt patterns and Chinese creator guides linking to [`angrydance.com/zh`](https://angrydance.com/zh).
- **2026 TikTok AI Trend Radar (556 Tracked Formats)**: [`angrydance.com/trends`](https://angrydance.com/trends) — daily view-velocity rankings for single-photo AI video and restyle trends.
- **Custom Multi-Model I2V Sandbox**: [`angrydance.com/tools/image-to-video`](https://angrydance.com/tools/image-to-video) — test custom 5s–10s prompts across `Kling 2.5 Turbo` and `Muse Video` with automatic fallback and failed-run credit refunds.

## Contributing & License

PRs adding verified prompt recipes (especially tested on `Kling 2.5+`, `Hailuo 2.3+`, `Seedance`, `Veo`, or `Wan 2.2`) are welcome! Please include your tested input aspect ratio, stage split (`1-step` vs `2-step`), and known failure edge cases. Released under the [MIT License](./LICENSE).
