# Bee Garden Findings: Splats, Plants and Panes

Sep 29, 2026 · @hope abeacan

Spark stays as the renderer, the app stays a web page, your FBX plants go into the Marble splat gardens as GLB files, and the blooms on the info panes come from your own photographs.

## The short answer

**Spark does the job, and you were right to bet on it.** Your White Clover FBX loaded straight into a Spark splat scene in a headless browser: 49,000 triangles in 0.3 s, textures on, and the depth is right in both directions (a splat post in front hides the plant; the plant hides the splat hedge behind it). Spark takes as many separate splat objects as we like, so a bloom splat can sit on the info pane in front of the garden, and it renders both eye views from one sort, which is what the Jupiter SR side-by-side needs.

**No standalone app is needed for the FBX files.** Three.js reads Tripo's binary FBX 7.4 files directly, and for the finished app they convert to GLB, which loads faster and smaller. A thin fullscreen wrapper for the SR computer is optional polish later, not a rebuild (Standalone app or web page, below).

**One real Jupiter SR bug was found and has a one-line fix.** Spark's default `preUpdate: true` runs its update inside the render pass, which knocks one eye's picture off in any frame where you move. `preUpdate: false` moves that update between frames (the same path Spark uses for VR headsets).

**Photograph first, as you decided.** Each plant's pane holds one high-quality 3D bloom floating 5–10 cm in front of the glass, spinning slowly until touched, plus your 4K photo (zoomable) and the facts. The bloom is a true capture through your LichtFeld pipeline when the flower is in season, and otherwise comes from your photo through TRELLIS (already on your PC) or Tripo. PixVerse is a video maker, not a 3D maker: keep it for an optional 2D living clip.

**Fifty plants, five per platform.** The table below gives every platform its five hero plants, matched to its Marble prompt and to the bee's behaviour on that platform, with the bee pull rising as the platforms climb.

## Spark: what the tests proved

**Your Tripo FBX works inside a Spark scene in the browser, and the Jupiter SR side-by-side picture now renders correctly while you move.** Everything below was run in a headless Chromium on the exact three.js 0.180 and Spark 2.2 builds the app uses.

| Test | Result |
| --- | --- |
| White Clover FBX (binary FBX 7.4, 1.5 MB, 4096-pixel maps) loaded with Three.js's FBX loader | 1 mesh, 48,906 triangles, loaded in 0.26 s; base colour and normal map picked up; roughness added by hand; drawn as a PBR material |
| Mesh and splats together, depth both ways | A splat post in front hides the plant; the plant hides the splat hedge behind it; the plant stands on the splat ground |
| Side by side through the app's own ArrayCamera path, camera still | Both eye views correct; the post shifts 61 px and the plant 42 px between eyes, exactly as the projection predicts for 64 mm eyes |
| Side by side while the camera moves, Spark default (`preUpdate: true`) | Broken on 12 of 12 frames: the first eye's splats drawn across the whole picture (the post straddling the seam at twice its width) |
| Side by side while the camera moves, `preUpdate: false` | Correct on 12 of 12 frames, four draw calls per frame as expected |
| The app itself, new check 13 in `tests/sr-test.js` | 13 of 13 pass; the check re-enables Spark's default to prove it bites (30.7 of 255 mismatch), then measures the shipped build (0.0) |

**The first fault, and its fix.** Spark's `SparkRenderer` is an ordinary Three.js mesh whose `onBeforeRender` runs the splat update (accumulate and sort) when `preUpdate` is on, which is the default outside VR. Inside that update it renders to its own targets and then restores the renderer's stored viewport, which is the full canvas. Three.js had just set the first eye's half-width viewport, so that eye's splats spread across the whole glass on every frame where the view changed. `preUpdate: false` moves the update between frames, the path Spark already uses for headsets, at the cost of the sort lagging the view by one frame, which Spark treats as normal. This is the same fault the DisplayXR glasses-free display project hit and worked around.

**The second fault, found on the way.** The app's two eye cameras come from `THREE.StereoCamera`, which writes each eye's world matrix directly and leaves its position at zero. Spark asks each eye where it is with `getWorldPosition()`, which rebuilds the matrix from that zero position: so Spark believed both eyes sat at the world origin, sorted the splats for the wrong place, never re-sorted as you walked, and, with the update inside the render pass, drew the first eye from the origin. `srComputeEyes()` now writes each eye's position and rotation from its matrix, so Spark sees the true eye. Both fixes are committed on `jupiter-sr-first` as the save `sr-ready-plan-a-2`, with a paler lime dot 10 cm in front of the glass added to the K markers. The VR route is untouched (Spark uses a different, correct path in a headset): 53 of 53 comfort checks and 13 of 13 SR checks pass. The live artifact is still v9, on purpose, until you have felt the new build in the headset.

**What Spark gives us for the plants and blooms.** Multiple `SplatMesh` objects coexist, each positioned and rotated on its own (scale is uniform only), all sorted together. Splats depth-test against opaque meshes, so GLB plants sit in a splat garden correctly, though splats do not write depth, so transparent meshes and depth-based post effects will not see them. It loads `.ply`, `.spz`, `.splat`, `.ksplat` and `.sog`, keeps spherical harmonics to degree 3, and offers a level-of-detail tree and streaming for big captures (desktop budget 1–5 million splats, Quest 3 about 1 million). It is WebGL 2 only; the WebGPU renderer is not supported. Version 2.2.0 (11 September 2026) is current.

**Marble exports fit.** World Labs exports SPZ and PLY at about 2 million splats (a low-resolution version at about 500,000), a collider GLB, and on the Pro plan a textured mesh; exporting needs the Standard plan or above. Marble's axes are +x left, +y down, +z forward, so the app's existing flip for Marble files is right.

## Standalone app or web page

**Stay with the web page, delivered as a folder, and add a fullscreen wrapper only if the SR computer needs it.** The FBX files were the reason to consider a native app, and they turn out not to need one: Three.js loads them today, and the finished app should carry them as GLB anyway.

| Route | What you get | What it costs | Verdict |
| --- | --- | --- | --- |
| Web page in Chrome or Edge, fullscreen (`--kiosk`), assets in a folder beside it | Everything built so far, VR included; one codebase; Spark and Three.js as now; the 53 + 12 checks keep running | A browser must be present; a local file needs a tiny local server or the `--allow-file-access-from-files` flag for module scripts | **Ship this for 15 October** |
| The same page inside a wrapper (Electron, Tauri or WebView2) | Double-click to launch, no browser chrome, autostart on the SR computer, file access without flags, an installer if you ever hand it to a clinic | A few hours to set up; \~100 MB extra on disk; VR still runs through the browser build | Add after 15 October if the SR computer is a kiosk |
| Native Unity or Unreal port (plan C in `SR-INTEGRATION.md`) | Best picture and eye-tracked weaving if Jupiter ever offers an SDK; Unity's Gaussian splat plug-ins exist | A rebuild of everything; the comfort rules would need re-testing in the headset; not before the deadline | Only if Jupiter provides an SDK and the picture demands it |

**Why GLB, not FBX, in the shipped app.** Three.js's FBX loader gives a Phong material and skips the roughness and metallic maps (it logged exactly that for the clover: `ShininessExponent` and `ReflectionFactor` maps not supported). In the test I rebuilt the material by hand into a PBR one, which works, but a one-off conversion is cleaner: Blender (or Sansar Prep's reader) writes each plant as a GLB with base colour, normal and roughness in the right slots, textures reduced from 4096 to 2048, and the metallic map dropped (a plant's metalness is zero). The clover's 24 MB of FBX and textures becomes roughly 3 MB, and a scene of five plants loads in a few seconds. Manuka's two models are 53 MB and 49 MB FBX files, so they are high-poly exports and want decimating first (Quad Remesher, which you have) to under 100,000 triangles.

**Budget per garden.** Five plants at 2048-pixel textures use about 300 MB of video memory, fine on the RTX 4070 Super and on Quest 3. Load each platform's plants when that platform opens and release them on leaving, the same way the Marble splat is handled now.

**Where the files live.** The published artifact page is limited to 16 MB, so the plants and gardens cannot be baked into it. Two homes work: the artifact's own supporting files (up to 256 MB per version, so all 50 plants at \~3 MB each plus the ten Marble splats at \~5–20 MB each fit), or a local folder next to the standalone file on the SR computer, which is faster and needs no network. Do both: the folder is the show copy, the artifact is the shareable one.

## The 50 plants across the ten gardens

Each platform gets five hero plants, every plant used once, so all 50 appear. The Marble garden paints the background; the hero plants are the GLB models placed where the bee works and where the info pane opens, so their identity is exact. Bee pull rises with the platform number, and each garden keeps one season so the ten don't look alike. Numbers are the Notion numbers.

| Platform | Garden and bee | Five hero plants | Pane opens on | Models now |
| --- | --- | --- | --- | --- |
| 0 Home / hub | Sunlit terrace, hills beyond; no bees | Lemon Blossom (43), Orange Blossom (42), Acacia (41), California Lilac (38), Silver Linden (50) | Lemon Blossom | 0 of 5 |
| 1 Calm garden orientation | Stream and pond, willow overhead; no bee on screen, optional far hum | White Willow (11), Snowdrop (27), Spring Crocus (26), English Bluebell (28), Dandelion (8) | White Willow | 2 of 5 |
| 2 Bee at a distance | Long grass corridor; one softened bee 15 m away | Oilseed Rape (19), Alfalfa (35), Buckwheat (34), Common Hawthorn (16), Sycamore Maple (17) | Oilseed Rape | 2 of 5 |
| 3 One predictable bee | Cottage lawn; one bee 7 m away on a steady loop | Borage (3), White Clover (1), Viper's Bugloss (20), Black Locust (10), Common Yarrow (32) | Borage | 4 of 5 |
| 4 Guided foraging path | Winding flower path; 1–2 lifelike bees, 2.5 m at the near bed | English Lavender (2), Catmint (47), Common Foxglove (21), Anise Hyssop (46), Fireweed (37) | English Lavender | 2 of 5 |
| 5 User-controlled closer view | Walled kitchen garden, gravel path; 2 bees, you choose how close, down to 1 m | Common Thyme (22), Oregano (24), Common Sage (23), Spearmint (25), Coriander (44) | Common Thyme | 3 of 5 |
| 6 Small group of bees | Meadow with three patches; 3 bees, one per patch, 4 m | Cornflower (29), Canada Goldenrod (31), New England Aster (30), Bird's-foot Trefoil (36), Black Tupelo (39) | Cornflower | 0 of 5 |
| 7 Interactive flower choice | Five raised beds in an arc; 0–4 bees per bed, one fly-by at 1 m | Sunflower (6), Garden Cosmos (33), Manuka (5), Common Heather (4), Joe-Pye Weed (48) | Sunflower | 3 of 5 |
| 8 Full garden experience | Full country garden; 6–8 bees, 0.5 m, one lands by your fingertip | Lacy Phacelia (7), Fennel (45), Sweet Chestnut (12), Horse Chestnut (18), Eucalyptus (49) | Lacy Phacelia | 2 of 5 |
| 9 Epilogue: hive activity | Orchard at golden hour; 20+ bees at the hive 20 m away, 2 foragers near | Apple Blossom (13), Wild Cherry Blossom (14), Pear Blossom (15), Small-Leaved Lime (9), Sourwood (40) | Apple Blossom | 4 of 5 |

**Why these groupings.**

- **Platform 0** carries the world's honey trees (lemon and orange in pots, acacia, the linden), so the hub can say where honey comes from before any bee appears. No bee pull is spent here.
- **Platform 1** is early spring by water: willow catkins and dandelions are the first forage of the year, which is the platform's lesson (hear a bee, learn how it forages).
- **Platform 2** is a farm track: the bright rape field at the far end is a strong depth cue on the SR and puts a real bee magnet safely 15 m away.
- **Platforms 3, 4 and 5** use the plants the Bee Behaviour Spec already names (borage and clover; lavender, catmint and foxglove; the herb bed), so the bee's flight paths land on the right flowers.
- **Platform 6's** three patches are three colours (blue cornflower, yellow goldenrod, purple aster), so each bee can be read against its own patch.
- **Platform 7's** five beds are five very different shapes, and the bee count per bed rises with real bee pull: cosmos 0–1, Joe-Pye weed 1, heather 2, manuka 3, sunflower 4.
- **Platform 8** puts phacelia, the finest forage on the list, in the near bed where the fingertip landing happens, with the big honey trees (sweet chestnut, horse chestnut, eucalyptus) at the back.
- **Platform 9** is a May orchard with a lime tree, the one that draws thousands of bees, beside the hive.

**Models now** counts the Tripo FBX files already in `Bees/Bee Plants XR Backup`: 22 plants have one, phacelia has maps but no model, and 27 are not started. Platforms 3 and 9 are nearly complete already; platforms 0 and 6 have nothing yet, which is where the photograph-to-3D work below goes first.

## Arcs of progression

The Bee Behaviour Spec turns five dials (distance, number, predictability, buzz, realism), one or two per platform. The plants add two quiet dials that make each step believable without adding bees: **plant pull** (how strongly the hero plants draw bees in real life) and **where the flowers stand** relative to the viewer. A third, **season**, keeps the ten gardens distinct.

| Platform | Bee (from the spec) | Plant pull | Where the flowers stand | Season |
| --- | --- | --- | --- | --- |
| 0 | None | None spent: citrus in pots, linden out of flower | Around the terrace edge, nothing in reach | Warm late spring, Mediterranean terrace |
| 1 | None on screen, optional far hum | Low: the first forage of the year | Across the water, far bank | Early spring |
| 2 | One, 15 m, softened | A magnet, but 15 m away (the rape field) | Both sides of the corridor, well back; the field at the far end | Late April farmland |
| 3 | One, 7 m, steady loop | Moderate: borage and clover | Far side of the lawn | June cottage garden |
| 4 | One or two, 2.5 m, lifelike | Moderate to high: lavender and catmint | A path from far to near; the near bed 2.5 m away | July |
| 5 | Two, you choose, down to 1 m | Moderate: herbs in flower | Along the path, as close as you walk | July kitchen garden |
| 6 | Three, one per patch, 4 m | High: aster, goldenrod, cornflower | Three patches at 4 m with open air between | Late summer meadow |
| 7 | 0–4 per bed, one fly-by at 1 m | You choose: cosmos low, sunflower high | Beds within reach, about 1 m | August |
| 8 | Six to eight, 0.5 m, a landing | Highest: phacelia in the near bed | The near bed, 0.5 m, under your hand | High summer |
| 9 | 20+ at the hive, two foragers near | Mass flowering: orchard and lime | Near bed close, hive 20 m away | May orchard at golden hour |

Two rules fall out of the table. **Pull and distance move against each other early on:** platform 2 shows a real bee magnet, but 15 m away, so the user learns that abundance far off is safe before abundance close up. **Season is a signature, not a ladder:** every garden holds one season so it reads as its own place, which gives the varied contexts the spec asks for (Craske's inhibitory learning), and the pane can say when the plant flowers.

The pane opens only from a plant: touch a hero flower or tree and its pane opens, so the pane always explains the thing the user reached for. The plant the bee is working (table above) is the one most users will touch first. Previous and Next on an open pane move through that platform's five.

## The info pane on the Jupiter SR screen

**The pane is three things: one high-quality 3D bloom floating 5–10 cm in front of the glass, the 4K photo you can zoom into, and the facts. The card sits on the glass; the garden stays behind.** On the SR the app draws two eye views side by side, so anything laid flat over the canvas once would appear stretched and doubled. The pane is drawn once per eye, exactly as the HUD already is (`layoutHud`, one copy per half), identical in both eyes, which puts it at zero parallax: on the glass, where a lenticular screen is sharpest and where text stays crisp. The bloom is a real 3D object in the scene, so it gets true stereo for free.

| Element | Depth on the SR | How |
| --- | --- | --- |
| One high-quality 3D bloom | 5–10 cm in front of the glass, spinning slowly until touched | A Spark splat (a LichtFeld capture, or TRELLIS from your photo) or a Tripo mesh, placed at the screen-plane distance minus 5–10 cm. On the SR a touch stops it and drags it round; in VR the trigger held on it grabs it |
| The 4K photo, zoomable | On the glass (zero parallax); pinch or tap to zoom | The same 4096-pixel file; a 1080p panel shows it oversampled twice, so zooming reveals real detail |
| The facts: name, Latin name, About, Plant facts, For bees | On the glass | HTML from the design canvas, duplicated per eye and squeezed to the layout, exactly like the HUD |
| Close, Step down, Previous, Next | On the glass | Touch mapped through `guardHit`, which already corrects for the stretched halves |

**Depth limits are already calibrated.** The K markers put a lime dot 5 cm in front of the glass; a second dot at 10 cm goes in next, so the ruler check covers the bloom's whole 5–10 cm range when the display arrives. Sony's guidance for spatial-reality displays (cited in the Bee Behaviour Spec) is that large pop-outs cause image loss and discomfort, so the bloom stays small and near the glass, and the pane never moves while it is open.

**Layout at 1920 × 1080.** The design canvas pane is 1600 × 900, so it scales by exactly 1.2 to fill a 1080p panel: card 1116 × 640, hero photo 480 × 480, body text 23 px. If the Jupiter SR only accepts half-width side by side, each eye has 960 columns and text blurs a little sideways; then body text goes to 26 px in Atkinson Hyperlegible and the card widens to fill the glass. If it accepts full-width side by side, nothing changes.

**Opening it.** The pane opens only when a flower or tree is interacted with: a touch on a hero plant on the SR, or a point and click on one in VR, opens that plant's pane with a fade (no motion, for comfort). There is no pane button, and the pane never opens on its own. The garden keeps running behind the glass; the bee keeps foraging; Step down and Pause stay available on the pane, as the spec requires. In VR the same pane becomes a world-locked panel about a metre ahead, following lazily like the toasts do.

**The physical frame.** The on-screen pane frame (honeycomb flap top left, clover bottom right) mirrors your printed case, so the flap on the glass and the flap on the case say the same thing: step down.

## Photographic 3D blooms from photographs

**One high-quality 3D bloom per plant, from your own pictures, so the rights stay yours.** Spark shows it; it does not make it. The bloom comes from a true capture whenever the flower is in season, and from the photo through TRELLIS or Tripo otherwise. Nothing else sits on the pane but the 4K photo and the facts.

| Source of the bloom | What you get | Where it is exact | Time per plant | Needs |
| --- | --- | --- | --- | --- |
| A real flower captured and trained in LichtFeld (first choice, in season) | A true splat with real sheen (keep SH3), `.spz` straight into Spark | Everywhere: it is a capture | About an hour: shoot, align, 8 minutes of training | The flower in season; 30–100 photos on a slow orbit against a plain backdrop, masked; your 360 Splat Pro and LichtFeld pipeline |
| One photo into TRELLIS (out of season) | A Gaussian-splat bloom (`.ply`, straight into Spark) plus a textured GLB | The side the photo shows; the far side is invented, so give it your best front-on 4K photo | About 5 minutes | Already on your PC (`trellis_env`, ComfyUI-IF\_Trellis). Microsoft's page asks for 16 GB of video memory; the ComfyUI node claims 8 GB, so the 4070 Super's 12 GB is to be tried. MIT licence |
| One photo into Tripo (fallback) | A textured GLB mesh with 4K PBR maps, like your plant models | As TRELLIS; a mesh spins cleanly and lights with the garden | About 10 minutes | Your paid Tripo plan |

**What the research says about tiny blooms.** Plant splats work: a University of Saskatchewan study reconstructed wheat plants from one 360° video of about 100 frames at a metre, at roughly 100,000 splats per plant, and found under 30 images made alignment fail while over 100 added nothing. Macro captures of insects use a turntable with focus stacking and a plain blue backdrop keyed out, and thin translucent parts survive. Two cautions: dense plants come out a little fuzzy, and a turntable only works if the background is masked away, because the trainer assumes the camera moves, not the subject. For a single bloom in a jar against a plain card, all of that is in your favour.

**Tripo stays in the picture.** Your plant models are Tripo image-to-3D already; the same route from a bloom photo gives the GLB for a pane bloom when a mesh is wanted instead of a splat.

**PixVerse and the other video makers.** They generate a short clip from a photo, not a 3D model. A clip that swings a camera round a flower can be fed to the splat pipeline, but the frames are not geometrically consistent (petals drift), so COLMAP either fails to align or the splat comes out soft and floaty, which is the opposite of exacting. Where a clip earns its place is as a 2D living bloom on the pane: a bee landing, petals in a breeze, played flat on the glass.

**Marble is for the gardens, not the blooms.** World Labs' own guidance asks for floors, walls and ground planes and warns against extreme close-ups without spatial context; it has no object mode.

**Accuracy check on generated pictures.** Your Notion prompts already ask for botanically accurate blooms with no invented petals. Keep a five-second check against a reference photo per plant (petal count, leaf shape, colour) before a generated image becomes the pane's hero, because the pane is the one place the app claims to be exact.

**In flower now, near you, for real captures (late September, Devon).** Garden cosmos, New England aster, goldenrod, catmint (second flush), fennel, borage (until frost), Joe-Pye weed, anise hyssop, yarrow, late sunflowers, and heather in pots at any garden centre. Lavender and oregano are going over. Everything spring (crocus, snowdrop, bluebell, dandelion, willow and the blossom trees) waits for 2027, so those take routes 1 and 2 now.

## What to do next, in order

The order puts the things that protect the 15 October build first, and the things that need the display or your PC where they can wait.

1. **Fix the Spark eye fault in the app** (`preUpdate: false` in `part-mod-a.js` on the `jupiter-sr-first` branch), rerun the 53 + 12 checks, and add a check that renders side by side while the camera moves so it can never come back. Done: save sr-ready-plan-a-2, 53 of 53 and 13 of 13 checks.
2. **Convert the 22 existing FBX plants to GLB** with a Blender batch script: 2048-pixel base colour, normal and roughness, metallic dropped, Manuka decimated. Runs on your PC (Blender 5.2), ten minutes for the lot; Done on my side: tools/fbx\_to\_glb.py is in the repo with the one-line command in its header; it fills Bees/app-assets with plant.glb and photo.jpg for every plant that has a model.
3. **Give the app a plant manifest**: one JSON per platform listing its five plants, positions, scale and pane order, loaded when the platform opens. The hero positions follow the bee's flowers in the spec (the borage clump, the near lavender bed, the herb bed, the three patches, the five beds, the phacelia bed, the orchard). Done: tools/make\_manifests.py writes the ten with a starting layout and real plant heights; the positions get tuned per Marble garden.
4. **Bring the info pane into the app** from the design canvas, drawn per eye, wired to the Notion export (`bee-plants.json`), with the 4K hero photo and Full 4K photo mode. It opens only when a flower or tree is interacted with (a touch on the SR, a point and click in VR); there is no pane button. Done (save plants-v1): the pane, the hero plants standing on the measured ground, and touch-to-open through the stretched SR halves, proven with a stand-in bloom and the clover GLB; 53 of 53 and 15 of 15 checks. The live artifact is still v9.
5. **One 3D bloom per plant**: a TRELLIS trial on the White Clover photo first (quality, and the 12 GB question), then the ten pane defaults, then the rest at five a day; Tripo image-to-3D for any TRELLIS spoils. The bloom sits 5–10 cm out, spins slowly, and stops when touched (trigger held, in VR).
6. **Capture the in-season blooms (cosmos, aster, goldenrod, catmint, fennel, borage, heather) through 360 Splat Pro and LichtFeld for the truest splats, and swap them in as they finish**.
7. **Marble gardens in**: as each of the ten exports arrives, load it into its platform, check the ground mapping and the metre scale against a hero plant, and set the spawn point.
8. **The 27 plants without models**: Tripo image-to-3D from your photos or generated bloom images, five a day, platforms 0 and 6 first.
9. **When the Jupiter SR arrives**: fullscreen, choose the layout, both depth modes, the K markers with a ruler, the real picture size in millimetres, and a frame-rate reading from the diagnostics report.
10. **Merge `jupiter-sr-first` into `main` and publish**, after one VR session confirms it still feels exactly as v9 did.

## Sources

Pages opened for these findings, on 29 September 2026.

- Spark: [docs](https://sparkjs.dev/docs/), [SparkRenderer](https://sparkjs.dev/docs/spark-renderer/), [SplatMesh](https://sparkjs.dev/docs/splat-mesh/), [system design](https://sparkjs.dev/docs/system-design/), [loading splats](https://sparkjs.dev/docs/loading-splats/), [performance](https://sparkjs.dev/docs/performance/), [level of detail](https://sparkjs.dev/docs/lod-getting-started/), [release 2.2.0](https://github.com/sparkjsdev/spark/releases/tag/v2.2.0), [WebGPU issue #394](https://github.com/sparkjsdev/spark/issues/394), and the `SparkRenderer.ts` and `splatVertex.glsl` source in the 2.2.0 package (the `preUpdate` path and viewport handling were read there)
- DisplayXR: [displayxr-web pull request #97](https://github.com/DisplayXR/displayxr-web/pull/97), the same side-by-side sorting fault on a glasses-free display
- World Labs Marble: [export specs](https://docs.worldlabs.ai/marble/export/specs), [Gaussian splat export for Spark](https://docs.worldlabs.ai/marble/export/gaussian-splat/spark.md), [mesh export](https://docs.worldlabs.ai/marble/export/mesh.md), [Chisel basics](https://docs.worldlabs.ai/marble/create/chisel-tools/chisel-basics), [image prompt guide](https://docs.worldlabs.ai/marble/create/prompt-guides/image-prompt.md), [account and billing](https://docs.worldlabs.ai/marble/support/account-billing.md)
- Image to 3D: [Microsoft TRELLIS](https://github.com/microsoft/TRELLIS), [TRELLIS.2](https://github.com/microsoft/TRELLIS.2), [ComfyUI-IF\_Trellis](https://github.com/if-ai/ComfyUI-IF_Trellis), [Hunyuan3D-2 licence](https://github.com/Tencent-Hunyuan/Hunyuan3D-2/blob/main/LICENSE) (excludes the UK), [LGM](https://github.com/3DTopia/LGM), [Stable Fast 3D](https://github.com/Stability-AI/stable-fast-3d)
- Plant and macro splats: [Splanting, University of Saskatchewan](https://splant.usask.ca/) and its [preprint](https://splant.usask.ca/static/assets/splanting-preprint.pdf), [macro Gaussian splatting of insects](https://lidarnews.com/macro-gaussian-splatting-of-insects/), [KIRI Engine capture guide](https://www.kiriengine.app/blog/how-to-capture-3d-gaussian-splats-kiri-engine), [nerfstudio turntable issue #3327](https://github.com/nerfstudio-project/nerfstudio/issues/3327)
- Display comfort: [Sony Spatial Reality Display app guidance](https://www.sony.net/Products/Developer-Spatial-Reality-display/en/tips/HowToMakeApps.html), as cited in the Bee Behaviour Spec
- Your own material: `HANDOVER.md`, `SR-INTEGRATION.md`, the Bee Behaviour Spec, the Marble platform prompts, the design canvas info pane, and the Notion export `bee-plants.json` with the `Bees/Bee Plants XR Backup` folder
