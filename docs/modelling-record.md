# Bee Senses & Gaussian Views — Modelling Record

Sep 29, 2026 · @hope abeacan

Each plant's info pane will show its bloom as a 3D Gaussian splat, plus the same flower as a honeybee, a UV camera and an infrared camera would see it. This record covers how those views are made, starting with 03 Borage (*Borago officinalis*), and is written up as the work goes.

## Handover for Opus 5.5

At 23:35 on 29 September the latest build is v9 + bee senses + borage pane, on a new private link and in `Bees/Handover/builds/v9-bee-senses-pane`, and Hope is trying it in VR. The app is for Jupiter SR first and VR second, due 15 October 2026. This section is what a new session needs to carry on; the open tasks are the checklist at the end of Results.

| Part | State | Where |
| --- | --- | --- |
| Bee senses pack (ComfyUI) | Working: 7 nodes, borage done | `ComfyUI/custom_nodes/ComfyUI-IML-BeeSenses` |
| Borage views, splats, meshes | Done for 03 Borage, from the deep-blue photo | `Bees/app-assets/plants/03/senses/` |
| Colour tables | 7 `.cube` files; 43 PNG strips built into the app | `Bees/app-assets/luts/` and `src/part-senses-luts.js` |
| App build | v9 + bee senses + opening card + borage pane; 53 of 53 comfort checks | Newest "A Gift For People With Fear Of Honeybees" in Hope's artifacts; `Bees/Handover/builds/v9-bee-senses-pane/` |
| App source | Git branch `v9-bee-senses`, tag `v9-bee-senses-pane` | `bee-garden-viewer-all-saves.bundle` in that build folder (`git clone` it) |
| v9 as tested | Untouched | The live v9 link; `Bees/Handover/builds/v9-backup/` |
| Notion | Borage row filled; splat upload paused until Hope decides | Bee Plants XR Master List |
| CC sources | Every source logged and credited | `Bees/Handover/trellis/CC_SOURCES_LOG.md` |

**Building and testing the app**

1. The app is seven parts in `src/`: `part-head.html`, `part-mod-a.js`, `part-garden.js`, `part-senses-luts.js`, `part-senses.js`, `part-pane-lite.js`, `part-mod-b.js`. `python3 assemble.py` joins them into the claude.ai page (`dist/bee-garden-viewer.html`), the standalone file and a local test copy.
2. To test, serve the test copy with three.js 0.180 and Spark 2.2 beside it and the borage files under `assets/plants/03/senses/`, then run `node tests/bgv-test.js` (53 comfort checks, Playwright with swiftshader). Test hooks are on `window.__bgv`.
3. To publish, send the page plus `assets/plants/03/senses/*`. claude.ai won't serve `.ply`, so each splat goes as base64 text (`<name>.ply.b64.txt`); the pane loads the `.ply` first and falls back to the text.
4. Headless screenshots time out under swiftshader. Stop the animation loop, call `__bgv.frame()` with pauses, and read `canvas.toDataURL()`.

**Rules Hope has set**

- VR comfort comes first: no hard cuts, no panes fixed to the head, no whole-view blur. Colour changes dip to our eyes over 0.3 s and fade in over 0.45 s.
- The bee views open only after becoming comfortable with bees (2 minutes on the last platform), or early with `#senses` on the link.
- Choosing a view on the pane switches the garden until it is chosen again. Grips, B, the panel list, voice and Step down all stay. Nothing puts the bee's-eye blur on the garden.
- Licences: CC0, CC BY, MIT or Apache-2.0 only; no NC, ND or SA. Credit every CC BY source under each image. The credit is "Bee senses by Hope Abeacan / Ion Music Live, CC BY 4.0".
- Hope does every sign-up (GitHub, Zenodo, backup). Downloads need her yes. Never overwrite the live v9 link or the v9 backup.
- If Hope turns down a tool call or says stop, stop and say plainly what did and didn't happen. No forced either/or questions.

**Code to know**

- `part-senses.js` holds the modes, the 33³ colour tables and species tables for plants 3, 20, 30, 31, 32 and 33. Splats change through a Spark dyno modifier; meshes and the sky through `onBeforeCompile` after `colorspace_fragment`. After changing the mix, call `updateVersion()` on each splat, or Spark keeps the old colours.
- `part-pane-lite.js` (marked `[PANE-LITE]`) is a borage-only stand-in for the plants-v1 pane. `spawnOn` calls `paneOnSpawn()`; `frame()` calls `paneFrame()`. When the plants-v1 pane is merged, replace it and keep `onSensesChanged()` and `sensesToggle()` as its link to bee senses.
- VR controls from v9: X pause, Y step down, A jump, B crouch, left stick click to fly, right stick to turn. The grips run bee senses; the trigger points at the borage and the pane.

## Purpose

The pane holds one 3D bloom floating in front of the glass, the 4K photo and the plant facts. This work adds a set of switchable views of the same flower:

- **Our eyes**: the photo as taken.
- **Bee colours**: the standard bee false-colour picture (bee green shown red, bee blue shown green, UV shown blue).
- **UV only**, and **each bee receptor** (UV, blue, green) on its own.
- **Bee colour map**: which colour a bee would sort each part of the flower into.
- **Bee's-eye**: the flower at a chosen distance, blurred to a bee's eyesight and broken into its hexagonal facets.
- **Infrared**: near-IR as an IR-converted camera would show it.
- **Height map** and a **3D relief** of the photo, as a mesh and as a splat, shown moving on two axes like rudder and ailerons.

The tool is a ComfyUI node pack (IML BeeSenses) so every plant runs the same way. Licence rule throughout: CC0, CC BY, MIT or Apache-2.0 only, with every CC BY source credited.

## Hour-by-hour log

Newest first. Times are UK time on 29 September 2026.

### 22:00 to 23:35

| Time | What happened | Result |
| --- | --- | --- |
| 23:33 | Build filed in `Bees/Handover/builds/v9-bee-senses-pane` with its borage assets and a bundle of every save; this record turned into a handover | README updated |
| 23:31 | Checked with the camera held still: garden to bee colours and back | The whole garden changes and returns fully to our eyes |
| 23:30 | New private link published; the v9 link and v9 backup untouched | `#senses` on the link opens the bee views early, because claude.ai links drop `?senses=1` |
| 23:27 | The link can't serve `.ply` files | Splats sent as base64 text beside the page; the pane tries the `.ply` first, then the text. Tested with a copy holding only the text versions |
| 23:25 | VR comfort checks rerun with the pane in | 53 of 53 pass (tests now skip the opening card) |
| 23:17 | Pictures of the pane and garden | The UV splat stands in front of the pane; garden and sky in UV behind it |
| 23:05 | Fixed: garden splats only changed colour while the head moved | Spark regenerates a splat only when its version changes; each fade step now bumps it |
| 22:57 | Borage pane wired in and tested | Borage 1.6 m ahead on every platform; trigger or click opens; views locked until bee senses open; choosing a view toggles the garden; bee's-eye and hexagon stay on the pane |
| 22:41 | Hope asked for this record to become a handover for Opus 5.5 once the app is done | This update |
| 22:00 to 22:40 | Opening card: the hexagon equations with Hope's words when the app loads. Hope in VR: "please include the pane", then "build the deep blue please" | Card checked on screen. The plants-v1 zip wasn't in Handover or Downloads, so a borage pane was built into v9 + bee senses |

### 21:00 to 22:00

| Time | What happened | Result |
| --- | --- | --- |
| 21:48 | Final v9 + bee senses build and a fresh bundle of every save filed in `Bees/Handover/builds` | 53 of 53 VR comfort checks pass |
| 21:46 | Step down also brings the colours back to our eyes | Tested: a bee view returns to normal on step down |
| 21:43 | Controls clarified: the pane click switches the garden until clicked again; the other controls stay | Panel list, B, right grip and "bee vision" step through; left grip and "our eyes" go back; bee's-eye never covers the garden |
| 21:38 | v9 backup and the v9 + bee senses copy filed in `Bees/Handover/builds` | Backup untouched; every git save bundled |
| 21:36 | Pane choices set: each view is its part-3D relief splat; clicking it switches the garden | Waits on the plants-v1 pane |
| 21:30 | Bee senses engine added to a copy of v9, the tested comfortable build | 53 of 53 comfort checks pass; engine switches, fades and returns correctly |
| 21:05 | Credit to sit under every view image, on the pane and the sheet | Each view carries its credit line in `views.json` |
| 20:58 | Bee senses to apply inside the renderers, to garden splats and plant meshes, in VR and on the Jupiter | GPU colour tables; six plants species-accurate |

### 20:15 to 21:00

| Time | What happened | Result |
| --- | --- | --- |
| 20:40 | Splats from all nine views; Notion Master List given seven bee-senses columns; Borage row filled | Splat upload to Notion paused on request |
| 20:22 | Asked for credits and the hexagon chart on every pane, and a GitHub repository for the flowers | Planned after the app build |

### 20:00 to 21:00

| Time | What happened | Result |
| --- | --- | --- |
| 20:14 | Results filed in `Bees/Handover/trellis/03_borage_senses`; spectra credit added to the CC log | Record written up; sign-off waits on your check |
| 20:12 | First full run of 03 Borage through the 12-node workflow, after a ComfyUI restart | Every output in one go; Depth Anything Small weights downloaded |
| 20:10 | Whole-scene modes for the viewer: bee colours, UV, receptors, colour map and infrared exported as `.cube` 3D LUTs | LUT matches the direct calculation within 0.7%; bee's-eye stays a shader |
| 20:08 | Pack installed in `ComfyUI\custom_nodes\ComfyUI-IML-BeeSenses` | Loaded after restart |
| 20:05 | Licensing set: code Apache-2.0, everything else CC BY 4.0, credit Hope Abeacan / Ion Music Live | LICENSE, LICENSE-CONTENT and NOTICE in the pack |
| 20:02 | All six nodes tested end to end outside ComfyUI | Senses 2 s, relief and animation 4 s, sheet and pane export working |

### 19:00 to 20:00

| Time | What happened | Result |
| --- | --- | --- |
| 19:55 | Relief mesh, splat and two-axis animation tested with a stand-in height map | GLB and splat PLY open; 48-frame animation renders in under 1 s |
| 19:52 | Pane views decided: bee colours, UV only, each receptor, colour map, bee's-eye, infrared | Exported as a pane-ready set with captions and credits |
| 19:50 | UV prediction tested across all 73 measured species | With a species spectrum: all 73 within bee discrimination. From colour alone: 18% |
| 19:45 | Borage petal spectrum read: UV rises from 7% at 340 nm to 28% at 380 nm | Borage is UV-blue to a bee |
| 19:40 | Measured flower spectra downloaded (Shrestha et al., CC BY 4.0), with permission | 73 species plus a leaf mean, bundled in the pack |
| 19:35 | Asked for the bee / UV / IR / human analysis with a height map, as a ComfyUI modeller | Node pack started |
| 19:31 | Open collections checked for multi-angle flower photos | iNaturalist, GBIF, Commons usable (CC0/CC BY); OmniObject3D needs sign-up; CO3D, MVImgNet non-commercial |
| 19:26 | TRELLIS run on 4 CC BY photos (2 Katsillis, 2 Ahel) in both modes | Near five petals, truer blue, sepals and stem |
| 19:22 | CC source log started | `CC_SOURCES_LOG.md` and `.csv` in the trellis handover |
| 19:16 | Single-photo TRELLIS runs of two of your photos | Still 8 to 10 petals: generator invents the sides |
| 19:12 | Multi-image TRELLIS on your 4 borage photos, stochastic and multidiffusion | 8 to 10 petals: four different blooms stacked |
| 19:07 | Photos loaded into ComfyUI and both jobs queued | About 1 minute per run on the 4070 Super |

## TRELLIS Gaussian blooms

TRELLIS makes a good borage only when its photos show one real flower from different sides; four front views of different flowers give a muddled, many-petalled star. The best result so far is multidiffusion on four CC BY photos.

| Run | Photos in | Mode | Result | Folder |
| --- | --- | --- | --- | --- |
| 03\_borage\_cc\_bloom\_multidiffusion | 4 CC BY: front and 45° of one bloom, underneath and side of another | multidiffusion | Near five petals, true blue, green sepals, stem. Best so far | `03_borage_cc_bloom_multidiffusion` |
| 03\_borage\_cc\_bloom\_stochastic | Same 4 | stochastic | Similar, petals spread flatter | `03_borage_cc_bloom_stochastic` |
| 03\_borage\_bloom\_single\_b | 1 of your photos (deep-blue bloom) | single | 8 to 10 spiky petals | `03_borage_bloom_single_b` |
| 03\_borage\_bloom\_single | 1 of your photos (lilac bloom) | single | 8 to 10 petals, pale | `03_borage_bloom_single` |
| 03\_borage\_bloom\_multidiffusion | Your 4 photos (4 different blooms, all front-on) | multidiffusion | Pointed star, 8 to 10 petals | `03_borage_bloom_multidiffusion` |
| 03\_borage\_bloom\_stochastic | Same 4 | stochastic | Rounder lilac rosette, stamen cone lost | `03_borage_bloom_stochastic` |

All folders are in `Bees/Handover/trellis`, each with the `.ply` splat, GLB, texture and a turntable video; `03_borage_cc_bloom_turntable.mp4` shows the two CC runs side by side.

What this teaches for the other plants:

- One flower from 3 or 4 angles beats 4 flowers from the front. The underneath and side views are what fix the petal count.
- Multidiffusion holds the shape better than stochastic for star-shaped blooms.
- TRELLIS colours run a little lilac; check against the photo before a bloom becomes the pane's hero.
- A CC observation with more than two angles of one flower is rare; pairing two look-alike blooms worked for borage.

## The bee's-eye model

A honeybee sees with three colour receptors, peaking in ultraviolet (344 nm), blue (436 nm) and green (544 nm), and has no red receptor. The pack works out how strongly each receptor would respond to every part of the photo, in daylight, with the bee's eye adjusted to green leaves.

| Part | Choice | Where it comes from |
| --- | --- | --- |
| Receptor peaks | UV 344, blue 436, green 544 nm | Peitsch et al. 1992, honeybee |
| Receptor curve shapes | Standard visual pigment template | Govardovskii et al. 2000 |
| Light | Daylight (CIE D65), 300 to 700 nm | CIE |
| Eye adjusted to | Measured mean of green leaves | Shrestha et al. 2024 |
| Colour judgement | Bee colour hexagon | Chittka 1992 |
| Eyesight | 2.6° per facet's view, facets 1.9° apart | Honeybee optics, typical values |

Each receptor's response is turned into an excitation between 0 and 1, and the three excitations place the colour on the hexagon:

```latex
E = \frac{P}{P + 1}, \qquad x = \frac{\sqrt{3}}{2}\,(E_G - E_{UV}), \qquad y = E_B - \tfrac{1}{2}(E_{UV} + E_G)
```

P is the receptor's catch relative to the leaves. Distance from the centre is how different the flower looks from the leaves: about 0.062 is the smallest difference a honeybee can tell, and above 0.1 is easy to see. The direction names the bee colour: UV, UV-blue, blue, blue-green, green or UV-green.

**Borage result:** UV-blue, 0.18 hexagon units from the leaves in your deep-blue photo (easy to see); the measured petal spectrum alone also reads UV-blue.

The bee's-eye view blurs the bee-colour picture to a bee's eyesight at a chosen distance and samples it on a hexagonal grid of facets. At 5 cm a 4 cm-wide frame spans about 44° of a bee's view, so the whole bloom is only about 20 facets across.

## UV and infrared

A camera photo holds no UV and no infrared, so both are predicted from the visible colour. UV is predicted reliably once the species' spectrum has been measured; from colour alone it is not. Infrared is a clearly labelled estimate until a real IR photo is supplied.

**How UV is filled in, best first:**

1. A real UV-pass photo of the flower, when you have one, replaces everything else.
2. The species' measured spectrum: petal pixels that match its colour take the whole measured curve, scaled by each pixel's brightness. Stamens, centre and buds use method 3.
3. Colour-only estimate: the violet end of the pixel's visible spectrum (400 to 420 nm) carried flat into the UV. Leaves take the measured leaf UV (about 1.5%).

**Test on all 73 measured species** (error = distance on the bee hexagon between prediction and the real spectrum):

| Method | Median error | 90% of species within | Within bee threshold (0.062) |
| --- | --- | --- | --- |
| With the species spectrum | 0.000 | 0.001 | 100% |
| Colour only | 0.318 | 0.673 | 18% |

The gap comes from UV pigments that are colourless to us, so identical-looking petals can be bright or dark in UV. Black-eyed Susan (*Rudbeckia hirta*) is the classic case: evenly yellow to us, a UV-dark centre to a bee. Borage petals reflect strongly in UV (7% at 340 nm rising to 28% at 380 nm), which is why a bee sees them as UV-blue.

**Infrared estimate:** plant tissue reflects near-IR strongly whatever its pigment, leaves most of all (the Wood effect). The estimate keeps the light and shade, drops the pigment colour and lifts foliage. It is a stand-in for the look of an IR camera, not a measurement.

**Species list:** 73 species are bundled. Those on the Bee Plants XR list include borage, yarrow (*Achillea millefolium*), garden cosmos (*Cosmos bipinnatus*), goldenrod (*Solidago canadensis*), New England aster (*Symphyotrichum novae*, as named in the dataset); the set also has chicory (*Cichorium intybus*) and viper's bugloss (*Echium vulgare*).

## Height map, 3D relief and animation

The height map comes from Depth Anything V2 Small, cut to the flower by a U2-Net mask; on borage it puts the stamen cone nearest and the petals falling away behind it, as expected. The relief is the photo lifted by that height map: 4 cm wide and 1.2 cm deep by default, one-sided, so it is shown from the front.

| Output | Format | Use |
| --- | --- | --- |
| Relief mesh | GLB, textured, Y up, +Z toward the viewer | Previews, Sansar or Unity, the pane as a mesh |
| Relief splat | 3D Gaussian Splatting PLY, one flat splat per grid point | Spark, the pane as a splat |
| Rudder-and-aileron animation | MP4 (640 px, 24 fps, 4 s loop) and frames | Showing the relief moving |

The animation sweeps diagonally: rudder (yaw) 28° and pitch 18° move together, and the ailerons bank 8° into each turn. That stays inside the ±30° a hand-held pane tilts. Setting phase to 90° turns the diagonal into an ellipse.

Known flaw: the preview renderer leaves faint dotted gaps on the steepest parts, the hairy sepals, at full tilt. The GLB mesh has none.

## What makes it new

We found no prior tool that gives species-calibrated bee vision from ordinary photos and carries it into 3D and a live, explorable garden. The science and some image tools already exist; the combination does not, as far as a quick search shows (not a full prior-art review).

| Already done by others | New in IML BeeSenses |
| --- | --- |
| Bee colour hexagon (Chittka 1992); flower spectra databases (FReD, Bayreuth) | UV predicted from an ordinary photo using the species' measured spectrum: all 73 species within bee discrimination, against 18% from colour alone |
| [Bee-view false-colour photos](https://pollinationecology.org/index.php/jpe/article/view/482), needing a UV-converted camera | No UV camera needed; a real UV photo can still be plugged in |
| [toBeeView](https://onlinelibrary.wiley.com/doi/full/10.1002/ece3.2442): how blurry flowers look to a bee | Views carried into 3D: height maps, part-3D splats, meshes, animation |
| Research tools for still images and videos | Whole-garden bee vision, live, in VR and on a glasses-free 3D screen |
| — | Offered as a reward at the end of a graded exposure programme for fear of bees |
| — | An open, repeatable pipeline: ComfyUI nodes, per-plant views, splats, credits and licences in one format |

Suggested wording for the README and Zenodo: "species-calibrated bee vision from ordinary photos", phrased as "we found no prior tool that…" rather than "the first".

## Sources, licences and credits

Everything used is commercially usable; the CC BY items need the credit line wherever the result is shown. The same list, with a line per photo, is in `CC_SOURCES_LOG.md` and `CC_SOURCES_LOG.csv` in `Bees/Handover/trellis`.

| Item | Used for | Licence | Credit line |
| --- | --- | --- | --- |
| [Borage photos, obs. 73154978](https://www.inaturalist.org/observations/73154978), Eleftherios Katsillis | TRELLIS input (front, 45°) | CC BY 4.0 | Borage photos © Eleftherios Katsillis, CC BY 4.0, via iNaturalist |
| [Borage photos, obs. 98801998](https://www.inaturalist.org/observations/98801998), Juraj Ahel | TRELLIS input (underneath, side) | CC BY 4.0 | Borage photos © Juraj Ahel, CC BY 4.0, via iNaturalist |
| [Flower reflectance data](https://doi.org/10.6084/m9.figshare.21081025.v2), Shrestha et al. 2024, Bayreuth | 73 petal spectra and leaf mean (UV and bee colours) | CC BY 4.0 | Flower spectra: Shrestha et al. 2024, University of Bayreuth, CC BY 4.0 |
| [Depth Anything V2](https://github.com/DepthAnything/Depth-Anything-V2) code and Small weights | Height map | Apache-2.0 | Height maps by Depth Anything V2 (Apache-2.0) |
| [TRELLIS](https://github.com/microsoft/TRELLIS) | Gaussian blooms | MIT | Not required; nice to include |
| Your own photos, 03\_borage\_1 to 4 | Tests and pane photo | Yours | None |

Not usable here: Depth Anything V2 Base, Large and Giant weights (CC BY-NC), CO3D and MVImgNet (non-commercial), CC BY-SA images (the model would have to carry the same licence).

Science references used in the model: Peitsch et al. 1992 (bee receptor peaks), Govardovskii et al. 2000 (pigment template), Chittka 1992 (colour hexagon), Wyman, Sloan and Shirley 2013 (colour-matching fit), CIE D65 daylight.

## Bee senses in the viewer

The garden can now switch into each bee view inside the renderers, so the Quest, PC VR and the Jupiter SR all draw it natively with no extra cost per frame. It is built on a copy of v9, the build tested as comfortable, and passes all 53 VR comfort checks.

**How it works.** Each view is a 33 × 33 × 33 colour table made by the ComfyUI pack. Spark looks up every splat's colour in it on the graphics card, and plant meshes do the same per pixel. Six plants use their own measured-spectrum tables: 03 Borage, 20 Viper's Bugloss, 30 New England Aster, 31 Canada Goldenrod, 32 Common Yarrow and 33 Garden Cosmos. Everything else uses the colour-only table. All 43 tables are built into the file (666 KB), so it works offline and in a headset browser.

| Control | What it does |
| --- | --- |
| Click a view on a plant's info pane | Switches the garden to that view until the same view is clicked again |
| Panel list, `B`, right grip, say "bee vision" | Next view |
| Left grip, say "our eyes", or Step down | Back to normal colours |

**Comfort.** A change dips gently to normal colours (0.3 s) and into the new view (0.45 s), never a hard cut. The bee's-eye blur is never applied to the garden; it stays on the pane, and clicking it leaves the garden as it is.

**When it opens.** After 2 minutes on platform 9, the hive garden, at the end of the programme. A facilitator can open it early by adding `?senses=1` to the address, or #senses on a claude.ai link. Once open it stays open on that device.

**Builds** (`Bees/Handover/builds`):

- `v9-backup`: v9 exactly as tested, plus a bundle of every git save
- `v9-bee-senses`: v9 with bee senses; branch `v9-bee-senses` in the repository
- `v9-bee-senses-pane`: adds the opening card and the borage pane, with its assets and a bundle of every save; tag `v9-bee-senses-pane`

**The borage pane.** A deep-blue borage stands 1.6 m ahead of you on every platform. Point and pull the trigger (VR) or click it (screen), and its pane opens in front of you and stays put in the garden; walk 3.5 m away and it closes. The pane shows the plant facts, the chosen view as its part-3D splat with caption and credit underneath, and a row of views. Before bee senses open, only Our eyes is there. It is a stand-in until the plants-v1 pane is merged.

## Results and sign-off

The first full run of 03 Borage through IML BeeSenses worked at 20:12 on 29 September: every view, the height map, three reliefs with animations, the pane set and the LUTs, in one ComfyUI workflow. Sign-off waits on your check against the real flower.

**Borage for a bee:** UV-blue, 0.18 hexagon units from green leaves, so easy to see; the measured petal spectrum puts it in the same place.

**Where everything is** (`Bees/Handover/trellis/03_borage_senses/`):

- `03_borage_senses_sheet.png`: every view on one sheet with the report
- `pane\03\`: the pane set, full size, with `views.json` (captions, estimated flags, credits) and `report.txt`
- `luts\`: seven whole-scene `.cube` LUTs for the viewer's modes
- reliefs for all nine views: GLB, part-3D splat PLY and rudder-and-aileron MP4 each
- the workflow: `IML_BeeSenses_03_borage` in ComfyUI's workflow list
- **In the app's assets** (`Bees/app-assets/plants/03/`): `photo.jpg`, `bloom.ply` (the TRELLIS bloom from the CC photos) and `senses\` with every view, its splat and mesh, and `views.json` giving each view's garden mode and its credit line; the colour tables in `Bees/app-assets/luts/`
- **In Notion** (Bee Plants XR Master List, Borage row): all 11 views, Bee Colour UV-blue, contrast 0.182, UV source and credits, in seven new bee-senses columns
- **Builds:** `Bees/Handover/builds` (see Bee senses in the viewer)

**Open before sign-off:**

- [ ] Hope: check the bee colours, UV and height map against the real borage
- [ ] Hope: try v9 + bee senses + borage pane in the headset and on the PC screen (`#senses` on the link opens the bee views early)
- [ ] Drop the plants-v1 zip into `Bees/Handover` so its full pane replaces the borage stand-in
- [ ] Notion: send the nine view splats (zipped, 2 MB each) when you've settled how Notion should hold them
- [ ] Decide which bloom the pane shows by default: the TRELLIS bloom or the photo relief
- [ ] Fill the preview renderer's dotted gaps if the animation is used in public
- [ ] Run the other 49 plants through the modeller (six with measured spectra, the rest estimated)
- [ ] Publish the pack, and a flowers repository, on GitHub and Zenodo under Hope Abeacan / Ion Music Live
- [ ] Companion view on the Jupiter for an accompanying person: when the screen arrives

**Back to the app next:** the pane and bee senses are in the v9 build; next are the other plants and the plants-v1 pane, for 15 October.
