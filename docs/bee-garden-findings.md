# Bee Garden Findings: Splats, Plants and the Info Pane

Hope Abeacan / Ion Music Live · first written 29 September 2026, revised 2 October 2026

These are the technical findings behind the garden app *A Gift For People With Fear Of Honeybees*: how Gaussian splats and plant models work together in the renderer, how the 50 plants are spread across the ten gardens, and how each plant's info pane works on the Jupiter SR glasses-free 3D screen and in VR.

## Contents

- [Key findings](#key-findings)
- [Status note](#status-note)
- [Spark renderer: what the tests showed](#spark-renderer-what-the-tests-showed)
- [Browser page, wrapper or native app](#browser-page-wrapper-or-native-app)
- [The 50 plants across the ten gardens](#the-50-plants-across-the-ten-gardens)
- [Arcs of progression](#arcs-of-progression)
- [The info pane](#the-info-pane)
- [3D blooms from photographs](#3d-blooms-from-photographs)
- [Progress and next steps](#progress-and-next-steps)
- [Sources](#sources)

## Key findings

- **Spark works as the splat renderer.** A Tripo plant model (FBX) loaded straight into a Spark splat scene: 49,000 triangles in 0.3 s, textures on, and depth correct in both directions. Spark handles many separate splat objects at once, so a bloom splat can sit on the info pane in front of the garden, and it renders both eye views from one sort, which is what the Jupiter SR side-by-side picture needs.
- **One real Jupiter SR bug was found, with a one-line fix.** Spark's default `preUpdate: true` runs its update inside the render pass, which breaks one eye's picture whenever the view moves. `preUpdate: false` moves the update between frames (the same path Spark already uses for VR headsets).
- **Plants ship as GLB, not FBX.** Three.js reads Tripo's FBX files directly, but GLB loads faster, is far smaller and keeps the PBR materials intact.
- **Blooms come from photographs.** Each pane holds one high-quality 3D bloom floating 5–10 cm in front of the glass, made from a real capture when the flower is in season, or from a photo through TRELLIS or Tripo otherwise. The photographs are made and chosen with Midjourney 8.2, Nano Banana (standard, 2 and Pro) and ChatGPT image generation (2 and 2.5).
- **Tripo generation is very helpful, and the blooms need to render better.** Tripo has been very helpful for generating the 3D models. The blooms don't yet render as well as they need to, and a workaround is in progress.
- **Fifty plants, five per garden.** Every platform gets five hero plants, matched to its garden and to how the bee behaves there, with the bee pull rising as the platforms climb.

## Status note

These findings were written on 29 September 2026 for the browser (WebXR) build of the app. Since then the browser build has been judged too lightweight and simple in appearance, and a native PC VR app is being looked at instead; the engine is not chosen yet. The Spark, plant and pane findings below still describe the browser build. The Jupiter SR and VR versions are treated as equal partners.

## Spark renderer: what the tests showed

All tests ran in headless Chromium on three.js 0.180 and Spark 2.2, the versions the app uses.

| Test | Result |
| --- | --- |
| White Clover FBX (binary FBX 7.4, 1.5 MB, 4096-pixel maps) loaded with Three.js's FBX loader | 1 mesh, 48,906 triangles, loaded in 0.26 s; base colour and normal map picked up; roughness added by hand; drawn as a PBR material |
| Mesh and splats together, depth both ways | A splat post in front hides the plant; the plant hides the splat hedge behind it; the plant stands on the splat ground |
| Side by side through the app's stereo camera, camera still | Both eye views correct; the post shifts 61 px and the plant 42 px between the eyes, as the projection predicts for 64 mm eye spacing |
| Side by side while the camera moves, Spark default (`preUpdate: true`) | Broken on 12 of 12 frames: the first eye's splats drawn across the whole picture |
| Side by side while the camera moves, `preUpdate: false` | Correct on 12 of 12 frames, four draw calls per frame as expected |
| Regression check added to the app's Jupiter SR tests | Passes. With Spark's default switched back on, the picture mismatches by 30.7 (of 255); with the fix, 0.0 |

### Fault 1: one eye drawn across the whole picture

Spark's `SparkRenderer` is an ordinary Three.js mesh whose `onBeforeRender` runs the splat update (accumulate and sort) when `preUpdate` is on, which is the default outside VR. During that update it renders to its own targets and then restores the renderer's stored viewport, which is the full canvas. Three.js had just set the first eye's half-width viewport, so that eye's splats spread across the whole screen on every frame where the view changed.

**Fix:** `preUpdate: false`. The sort then lags the view by one frame, which Spark treats as normal. The DisplayXR glasses-free display project hit the same fault and worked around it the same way.

### Fault 2: both eyes placed at the origin

The app's eye cameras come from `THREE.StereoCamera`, which writes each eye's world matrix directly and leaves its position at zero. Spark asks each eye where it is with `getWorldPosition()`, which rebuilds the matrix from that zero position. So Spark believed both eyes sat at the world origin, sorted the splats for the wrong place and never re-sorted as you walked.

**Fix:** the app now writes each eye's position and rotation from its matrix, so Spark sees the true eye.

With both fixes in, all 53 VR comfort checks and all Jupiter SR checks pass. The VR route is unaffected, because Spark uses a different, correct path in a headset.

### What Spark offers for plants and blooms

- Many `SplatMesh` objects can coexist, each positioned and rotated on its own (scale is uniform only), all sorted together.
- Splats depth-test against opaque meshes, so GLB plants sit correctly in a splat garden. Splats do not write depth, so transparent meshes and depth-based post effects will not see them.
- Loads `.ply`, `.spz`, `.splat`, `.ksplat` and `.sog`, keeps spherical harmonics to degree 3, and offers a level-of-detail tree and streaming for big captures (desktop budget 1–5 million splats, Quest 3 about 1 million).
- WebGL 2 only; the WebGPU renderer is not supported.
- Version 2.2.0 (11 September 2026) was current at the time of testing.

### World Labs Marble exports

Marble exports SPZ and PLY at about 2 million splats (a low-resolution version at about 500,000), plus a collider GLB, and a textured mesh on the Pro plan. Exporting needs the Standard plan or above. Marble's axes are +x left, +y down, +z forward, so the app flips Marble files on load.

## Browser page, wrapper or native app

This is the comparison made on 29 September, when the browser build was the plan. See the [status note](#status-note) for where this stands now.

| Route | What you get | What it costs | Assessment on 29 Sep |
| --- | --- | --- | --- |
| Web page in Chrome or Edge, fullscreen (`--kiosk`), assets in a folder beside it | Everything built so far, VR included; one codebase; the automated checks keep running | A browser must be present; a local file needs a small local server or the `--allow-file-access-from-files` flag for module scripts | First choice for the first build |
| The same page inside a wrapper (Electron, Tauri or WebView2) | Double-click to launch, no browser chrome, autostart on the SR computer, file access without flags, an installer for clinics | A few hours to set up; about 100 MB extra on disk; VR still runs through the browser build | Later polish, if the SR computer runs as a kiosk |
| Native Unity or Unreal port | Best picture; Unity Gaussian splat plug-ins exist | A rebuild of everything; the comfort rules need re-testing in the headset | Only if the picture demands it or Jupiter offers an SDK |

### Why GLB rather than FBX

Three.js's FBX loader gives a Phong material and skips the roughness and metallic maps (for the clover it reported that the `ShininessExponent` and `ReflectionFactor` maps are not supported). Rebuilding the material by hand works, but a one-off conversion is cleaner: Blender writes each plant as a GLB with base colour, normal and roughness in the right slots, textures reduced from 4096 to 2048 pixels, and the metallic map dropped (a plant's metalness is zero).

The clover's 24 MB of FBX and textures becomes roughly 3 MB, and a scene of five plants loads in a few seconds. Very high-poly exports (Manuka's two models are around 50 MB each) are decimated first, for example with Quad Remesher, to under 100,000 triangles.

### Memory budget per garden

Five plants at 2048-pixel textures use about 300 MB of video memory, which is fine on an RTX 4070 Super and on a Quest 3. Each platform's plants load when that platform opens and are released on leaving, the same way the Marble garden splat is handled. Plants and gardens load from a local asset folder beside the app, which is fastest and needs no network.

## The 50 plants across the ten gardens

Each platform gets five hero plants, and every plant is used once, so all 50 appear. The Marble garden paints the background; the hero plants are the GLB models placed where the bee works and where the info pane opens, so they are botanically exact. Bee pull rises with the platform number, and each garden keeps one season so the ten don't look alike.

Platforms are numbered 0 to 9 here, as in the app (the [project summary](project-summary.md) counts them 1 to 10). Numbers in brackets are each plant's number in the project's 50-plant list.

| Platform | Garden and bee | Five hero plants | Pane opens on |
| --- | --- | --- | --- |
| 0 Home | Sunlit terrace, hills beyond; no bees | Lemon Blossom (43), Orange Blossom (42), Acacia (41), California Lilac (38), Silver Linden (50) | Lemon Blossom |
| 1 Calm garden | Stream and pond, willow overhead; no bee on screen, optional far hum | White Willow (11), Snowdrop (27), Spring Crocus (26), English Bluebell (28), Dandelion (8) | White Willow |
| 2 Bee at a distance | Long grass corridor; one softened bee 15 m away | Oilseed Rape (19), Alfalfa (35), Buckwheat (34), Common Hawthorn (16), Sycamore Maple (17) | Oilseed Rape |
| 3 One predictable bee | Cottage lawn; one bee 7 m away on a steady loop | Borage (3), White Clover (1), Viper's Bugloss (20), Black Locust (10), Common Yarrow (32) | Borage |
| 4 Guided foraging path | Winding flower path; 1–2 lifelike bees, 2.5 m at the near bed | English Lavender (2), Catmint (47), Common Foxglove (21), Anise Hyssop (46), Fireweed (37) | English Lavender |
| 5 Your own pace | Walled kitchen garden, gravel path; 2 bees, you choose how close, down to 1 m | Common Thyme (22), Oregano (24), Common Sage (23), Spearmint (25), Coriander (44) | Common Thyme |
| 6 A small group | Meadow with three patches; 3 bees, one per patch, 4 m | Cornflower (29), Canada Goldenrod (31), New England Aster (30), Bird's-foot Trefoil (36), Black Tupelo (39) | Cornflower |
| 7 Choose a flower | Five raised beds in an arc; 0–4 bees per bed, one fly-by at 1 m | Sunflower (6), Garden Cosmos (33), Manuka (5), Common Heather (4), Joe-Pye Weed (48) | Sunflower |
| 8 Full garden | Full country garden; 6–8 bees, 0.5 m, one lands by your fingertip | Lacy Phacelia (7), Fennel (45), Sweet Chestnut (12), Horse Chestnut (18), Eucalyptus (49) | Lacy Phacelia |
| 9 The hive | Orchard at golden hour; 20+ bees at the hive 20 m away, 2 foragers near | Apple Blossom (13), Wild Cherry Blossom (14), Pear Blossom (15), Small-Leaved Lime (9), Sourwood (40) | Apple Blossom |

On 29 September, 22 of the 50 plants had a Tripo model; the rest are to follow.

### Why these groupings

- **Platform 0** carries the honey trees (lemon and orange in pots, acacia, the linden), so the hub can say where honey comes from before any bee appears.
- **Platform 1** is early spring by water: willow catkins and dandelions are the first forage of the year, which fits the platform's lesson (hear a bee, learn how it forages).
- **Platform 2** is a farm track: the bright rape field at the far end is a strong depth cue on the SR and puts a real bee magnet safely 15 m away.
- **Platforms 3, 4 and 5** use the plants the bee behaviour spec already names (borage and clover; lavender, catmint and foxglove; the herb bed), so the bee's flight paths land on the right flowers.
- **Platform 6's** three patches are three colours (blue cornflower, yellow goldenrod, purple aster), so each bee can be read against its own patch.
- **Platform 7's** five beds are five very different shapes, and the bee count per bed rises with real bee pull: cosmos 0–1, Joe-Pye weed 1, heather 2, manuka 3, sunflower 4.
- **Platform 8** puts phacelia, the finest forage on the list, in the near bed where the fingertip landing happens, with the big honey trees (sweet chestnut, horse chestnut, eucalyptus) at the back.
- **Platform 9** is a May orchard with a lime tree, the one that draws thousands of bees, beside the hive.

## Arcs of progression

The bee behaviour spec turns five dials across the platforms: distance, number, predictability, buzz and realism. The plants add two quiet dials that make each step believable without adding bees: **plant pull** (how strongly the hero plants draw bees in real life) and **where the flowers stand** relative to the viewer. A third, **season**, keeps the ten gardens distinct.

| Platform | Bee (from the spec) | Plant pull | Where the flowers stand | Season |
| --- | --- | --- | --- | --- |
| 0 | None | None: citrus in pots, linden out of flower | Around the terrace edge, nothing in reach | Warm late spring, Mediterranean terrace |
| 1 | None on screen, optional far hum | Low: the first forage of the year | Across the water, far bank | Early spring |
| 2 | One, 15 m, softened | A magnet, but 15 m away (the rape field) | Both sides of the corridor, well back; the field at the far end | Late April farmland |
| 3 | One, 7 m, steady loop | Moderate: borage and clover | Far side of the lawn | June cottage garden |
| 4 | One or two, 2.5 m, lifelike | Moderate to high: lavender and catmint | A path from far to near; the near bed 2.5 m away | July |
| 5 | Two, you choose, down to 1 m | Moderate: herbs in flower | Along the path, as close as you walk | July kitchen garden |
| 6 | Three, one per patch, 4 m | High: aster, goldenrod, cornflower | Three patches at 4 m with open air between | Late summer meadow |
| 7 | 0–4 per bed, one fly-by at 1 m | You choose: cosmos low, sunflower high | Beds within reach, about 1 m | August |
| 8 | Six to eight, 0.5 m, a landing | Highest: phacelia in the near bed | The near bed, 0.5 m, under your hand | High summer |
| 9 | 20+ at the hive, two foragers near | Mass flowering: orchard and lime | Near bed close, hive 20 m away | May orchard at golden hour |

Two rules come out of the table:

- **Pull and distance move against each other early on.** Platform 2 shows a real bee magnet, but 15 m away, so the user learns that abundance far off is safe before meeting abundance close up.
- **Season is a signature, not a ladder.** Every garden holds one season so it reads as its own place. That gives the varied contexts the spec asks for (Craske's inhibitory learning), and the pane can say when each plant flowers.

The pane opens only from a plant: touch a hero flower or tree and its pane opens, so the pane always explains the thing the user reached for. The plant the bee is working is the one most users will touch first. Previous and Next on an open pane move through that platform's five plants.

## The info pane

The pane holds three things: one high-quality 3D bloom floating 5–10 cm in front of the glass, a 4K photo you can zoom into, and the plant facts. Nothing else.

### On the Jupiter SR

The card sits on the glass and the garden stays behind it. The app draws two eye views side by side, so anything laid flat over the whole canvas would appear stretched and doubled. Instead the pane is drawn once per eye, identical in both, which puts it at zero parallax: on the glass, where a lenticular screen is sharpest and text stays crisp. The bloom is a real 3D object in the scene, so it gets true stereo for free.

| Element | Depth on the SR | How |
| --- | --- | --- |
| One high-quality 3D bloom | 5–10 cm in front of the glass, spinning slowly until touched | A Spark splat (a real capture, or TRELLIS from a photo) or a Tripo mesh. A touch stops it and drags it round; in VR, holding the trigger on it grabs it |
| The 4K photo, zoomable | On the glass; pinch or tap to zoom | The full 4096-pixel file; a 1080p panel shows it oversampled twice, so zooming reveals real detail |
| The facts: name, Latin name, About, Plant facts, For bees | On the glass | HTML duplicated per eye and fitted to the layout, like the rest of the on-screen display |
| Close, Step down, Previous, Next | On the glass | Touch input corrected for the stretched halves |

**Depth limits.** Calibration dots (the K markers) sit 5 cm and 10 cm in front of the glass, so a ruler check covers the bloom's whole range once the display arrives. Sony's guidance for spatial-reality displays is that large pop-outs cause image loss and discomfort, so the bloom stays small and near the glass, and the pane stays still while it is open.

**Layout at 1920 × 1080.** The design pane is 1600 × 900, so it scales by exactly 1.2 to fill a 1080p panel: card 1116 × 640, hero photo 480 × 480, body text 23 px. If the Jupiter SR only accepts half-width side by side, each eye has 960 columns and text blurs a little sideways; then body text goes to 26 px in Atkinson Hyperlegible and the card widens to fill the glass. If it accepts full-width side by side, nothing changes.

**Opening it.** The pane opens only when a flower or tree is interacted with: a touch on a hero plant on the SR, or a point and click in VR. It fades in (no motion, for comfort). There is no pane button, and the pane never opens on its own. The garden keeps running behind it, the bee keeps foraging, and Step down and Pause stay available, as the spec requires.

**The physical frame.** The on-screen pane frame (honeycomb flap top left, clover bottom right) mirrors the 3D-printed screen case, so the flap on the glass and the flap on the case say the same thing: step down.

### In VR

The pane is held in one hand and can be moved, and the bloom pops out of it in 3D.

## 3D blooms from photographs

One high-quality 3D bloom per plant, made from the project's own pictures rather than taken from other people's collections. Spark shows the bloom; it does not make it.

### Where the photographs come from

The plant photographs, including the 4K photo on each pane and the pictures the blooms are made from, are made and chosen with these image generators:

- **Midjourney 8.2**
- **Nano Banana:** standard, Nano Banana 2 and Nano Banana Pro
- **ChatGPT image generation:** versions 2 and 2.5

Each picture is checked against a reference photo of the real plant before it is used (see the accuracy check below).

| Source of the bloom | What you get | Where it is exact | Time per plant | Needs |
| --- | --- | --- | --- | --- |
| A real flower captured and trained in LichtFeld (first choice, in season) | A true splat with real sheen (keep SH3), `.spz` straight into Spark | Everywhere: it is a capture | About an hour: shoot, align, about 8 minutes of training | The flower in season; 30–100 photos on a slow orbit against a plain backdrop, masked |
| One photo into TRELLIS (out of season) | A Gaussian-splat bloom (`.ply`, straight into Spark) plus a textured GLB | The side the photo shows; the far side is invented, so use the best front-on 4K photo | About 5 minutes | TRELLIS in ComfyUI (ComfyUI-IF_Trellis). Microsoft lists 16 GB of video memory; it runs on a 12 GB RTX 4070 Super. MIT licence |
| One photo into Tripo (fallback) | A textured GLB mesh with 4K PBR maps, like the plant models | As TRELLIS; a mesh spins cleanly and lights with the garden | About 10 minutes | A Tripo account |

**What research says about small blooms.** Plant splats work: a University of Saskatchewan study reconstructed wheat plants from one 360° video of about 100 frames at a metre, at roughly 100,000 splats per plant, and found that under 30 images made alignment fail while over 100 added nothing. Macro captures of insects use a turntable with focus stacking and a plain blue backdrop keyed out, and thin translucent parts survive. Two cautions: dense plants come out a little fuzzy, and a turntable only works if the background is masked away, because the trainer assumes the camera moves, not the subject. A single bloom in a jar against plain card suits all of this.

**Tripo.** The plant models are already Tripo image-to-3D; the same route from a bloom photo gives a GLB for the pane when a mesh is wanted instead of a splat.

**Video generators (PixVerse and similar).** They make a short clip from a photo, not a 3D model. A clip that swings round a flower can be fed to a splat pipeline, but the frames are not geometrically consistent (petals drift), so alignment fails or the splat comes out soft. A clip is better used as a 2D living bloom on the pane: a bee landing, petals in a breeze.

**Marble is for gardens, not blooms.** World Labs' own guidance asks for floors, walls and ground planes and warns against extreme close-ups without spatial context; it has no object mode.

**Accuracy check.** Before a generated image becomes a pane's hero, it gets a quick check against a reference photo (petal count, leaf shape, colour), because the pane is the one place the app claims to be exact.

**Capture season.** In late September in southern England, garden cosmos, New England aster, goldenrod, catmint (second flush), fennel, borage (until frost), Joe-Pye weed, anise hyssop, yarrow, late sunflowers and potted heather can all be captured live. The spring plants (crocus, snowdrop, bluebell, dandelion, willow and the blossom trees) wait for spring 2027, so they use the photo routes for now.

## Progress and next steps

**Done**

1. Fixed the Spark side-by-side fault (`preUpdate: false`) and added a check that renders side by side while the camera moves, so it can't come back.
2. A Blender batch script converts the existing FBX plants to GLB (2048-pixel base colour, normal and roughness, metallic dropped, high-poly models decimated).
3. A plant manifest per platform: one JSON listing its five plants, positions, scale and pane order, loaded when the platform opens, with a starting layout and real plant heights. Positions are tuned per Marble garden.
4. The info pane is in the app: drawn per eye, filled from the plant list export, with the 4K hero photo and touch-to-open through the stretched SR halves, tested with a stand-in bloom and the clover GLB.

**Still to do**

1. One 3D bloom per plant: a TRELLIS trial on the White Clover photo first, then the ten pane defaults, then the rest; Tripo for any TRELLIS failures.
2. Capture the in-season blooms (cosmos, aster, goldenrod, catmint, fennel, borage, heather) as true splats and swap them in as they finish.
3. Bring in the Marble gardens: load each export into its platform, check the ground and the metre scale against a hero plant, and set the spawn point.
4. Model the plants that don't have one yet, from photos or generated bloom images, platforms 0 and 6 first.
5. When the Jupiter SR arrives: fullscreen, choose the layout, both depth modes, the K markers with a ruler, the real picture size in millimetres, and a frame-rate reading.
6. Merge the Jupiter SR work into the main build and publish, after a VR session confirms it still feels as comfortable as before.

## Sources

Pages consulted on 29 September 2026.

- **Spark:** [docs](https://sparkjs.dev/docs/), [SparkRenderer](https://sparkjs.dev/docs/spark-renderer/), [SplatMesh](https://sparkjs.dev/docs/splat-mesh/), [system design](https://sparkjs.dev/docs/system-design/), [loading splats](https://sparkjs.dev/docs/loading-splats/), [performance](https://sparkjs.dev/docs/performance/), [level of detail](https://sparkjs.dev/docs/lod-getting-started/), [release 2.2.0](https://github.com/sparkjsdev/spark/releases/tag/v2.2.0), [WebGPU issue #394](https://github.com/sparkjsdev/spark/issues/394), and the `SparkRenderer.ts` and `splatVertex.glsl` source in the 2.2.0 package (where the `preUpdate` path and viewport handling were read)
- **DisplayXR:** [displayxr-web pull request #97](https://github.com/DisplayXR/displayxr-web/pull/97), the same side-by-side sorting fault on a glasses-free display
- **World Labs Marble:** [export specs](https://docs.worldlabs.ai/marble/export/specs), [Gaussian splat export for Spark](https://docs.worldlabs.ai/marble/export/gaussian-splat/spark.md), [mesh export](https://docs.worldlabs.ai/marble/export/mesh.md), [Chisel basics](https://docs.worldlabs.ai/marble/create/chisel-tools/chisel-basics), [image prompt guide](https://docs.worldlabs.ai/marble/create/prompt-guides/image-prompt.md), [account and billing](https://docs.worldlabs.ai/marble/support/account-billing.md)
- **Image to 3D:** [Microsoft TRELLIS](https://github.com/microsoft/TRELLIS), [TRELLIS.2](https://github.com/microsoft/TRELLIS.2), [ComfyUI-IF_Trellis](https://github.com/if-ai/ComfyUI-IF_Trellis), [Hunyuan3D-2 licence](https://github.com/Tencent-Hunyuan/Hunyuan3D-2/blob/main/LICENSE) (excludes the UK), [LGM](https://github.com/3DTopia/LGM), [Stable Fast 3D](https://github.com/Stability-AI/stable-fast-3d)
- **Plant and macro splats:** [Splanting, University of Saskatchewan](https://splant.usask.ca/) and its [preprint](https://splant.usask.ca/static/assets/splanting-preprint.pdf), [macro Gaussian splatting of insects](https://lidarnews.com/macro-gaussian-splatting-of-insects/), [KIRI Engine capture guide](https://www.kiriengine.app/blog/how-to-capture-3d-gaussian-splats-kiri-engine), [nerfstudio turntable issue #3327](https://github.com/nerfstudio-project/nerfstudio/issues/3327)
- **Display comfort:** [Sony Spatial Reality Display app guidance](https://www.sony.net/Products/Developer-Spatial-Reality-display/en/tips/HowToMakeApps.html)
- **Plant photographs:** made and chosen with Midjourney 8.2, Nano Banana (standard, 2 and Pro) and ChatGPT image generation (2 and 2.5)
- **Project material:** the bee behaviour spec, the Marble platform prompts, the info pane design and the 50-plant list
