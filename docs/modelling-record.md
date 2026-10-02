# Bee Senses and Gaussian Views: Modelling Record

Hope Abeacan / Ion Music Live · first written 29 September 2026, revised 2 October 2026

Each plant's info pane shows its bloom as a 3D Gaussian splat, plus the same flower as a honeybee, a UV camera and an infrared camera would see it. This record explains how those views are made, starting with plant 03, borage (*Borago officinalis*), and what has been learned so far.

## Contents

- [At a glance](#at-a-glance)
- [The views](#the-views)
- [Design rules](#design-rules)
- [The bee's-eye model](#the-bees-eye-model)
- [UV and infrared](#uv-and-infrared)
- [Height map, 3D relief and animation](#height-map-3d-relief-and-animation)
- [TRELLIS Gaussian blooms](#trellis-gaussian-blooms)
- [Bee senses in the viewer](#bee-senses-in-the-viewer)
- [What's new about it](#whats-new-about-it)
- [Results for borage](#results-for-borage)
- [Still to do](#still-to-do)
- [How it was built](#how-it-was-built)
- [Sources, licences and credits](#sources-licences-and-credits)

## At a glance

| Part | State |
| --- | --- |
| Bee senses node pack for ComfyUI (`ComfyUI-IML-BeeSenses`) | Working: 7 nodes; the first plant, borage, done |
| Borage views, splats and meshes | Done, from a deep-blue borage photo |
| Whole-garden colour tables | 7 `.cube` LUTs; 43 colour tables built into the viewer |
| Viewer | Bee senses, an opening card and a borage pane added; passes all 53 VR comfort checks |
| Credits | Every outside source logged, with its credit line under each image |

## The views

The pane holds one 3D bloom floating in front of the glass, the 4K photo and the plant facts. Bee senses adds a set of switchable views of the same flower:

- **Our eyes:** the photo as taken.
- **Bee colours:** the standard bee false-colour picture (bee green shown as red, bee blue as green, UV as blue).
- **UV only**, and **each bee receptor** (UV, blue, green) on its own.
- **Bee colour map:** which colour a bee would sort each part of the flower into.
- **Bee's-eye:** the flower at a chosen distance, blurred to a bee's eyesight and broken into its hexagonal facets.
- **Infrared:** near-IR as an IR-converted camera would show it.
- **Height map** and a **3D relief** of the photo, as a mesh and as a splat, shown moving on two axes like a plane's rudder and ailerons.

The tool is a ComfyUI node pack, IML BeeSenses, so every plant runs through it the same way.

## Design rules

- **VR comfort comes first.** No hard cuts, no panes fixed to the head, no whole-view blur. A change of view dips to our eyes over 0.3 s and fades into the new view over 0.45 s.
- **Bee senses are a reward.** The bee views unlock only once the person is comfortable with bees (2 minutes on the last platform, the hive). A facilitator can open them early by adding `?senses=1` to the address.
- **Views change only from the pane.** A view is chosen by clicking that view's high-resolution image on the info pane, not by buttons, keys or voice. Choosing a view switches the whole garden into it until the same view is clicked again.
- **The bee's-eye blur stays on the pane.** It is never put over the garden.
- **Licences:** CC0, CC BY, MIT or Apache-2.0 only; no NC, ND or SA material. Every CC BY source is credited beneath each image it appears in.
- **Credit line:** "Bee senses by Hope Abeacan / Ion Music Live, CC BY 4.0".

## The bee's-eye model

A honeybee sees with three colour receptors, peaking in ultraviolet (344 nm), blue (436 nm) and green (544 nm), and has no red receptor. The pack works out how strongly each receptor would respond to every part of the photo, in daylight, with the bee's eye adapted to green leaves.

| Part | Choice | Source |
| --- | --- | --- |
| Receptor peaks | UV 344, blue 436, green 544 nm | Peitsch et al. 1992, honeybee |
| Receptor curve shapes | Standard visual pigment template | Govardovskii et al. 2000 |
| Light | Daylight (CIE D65), 300 to 700 nm | CIE |
| Eye adapted to | Measured mean of green leaves | Shrestha et al. 2024 |
| Colour judgement | Bee colour hexagon | Chittka 1992 |
| Eyesight | 2.6° per facet's view, facets 1.9° apart | Typical honeybee optics |

Each receptor's response becomes an excitation between 0 and 1, and the three excitations place the colour on the hexagon:

```math
E = \frac{P}{P + 1}, \qquad x = \frac{\sqrt{3}}{2}\,(E_G - E_{UV}), \qquad y = E_B - \tfrac{1}{2}(E_{UV} + E_G)
```

*P* is the receptor's catch relative to the leaves. Distance from the centre of the hexagon is how different the flower looks from the leaves: about 0.062 is the smallest difference a honeybee can tell, and above 0.1 is easy to see. The direction gives the bee colour: UV, UV-blue, blue, blue-green, green or UV-green.

The bee's-eye view blurs the bee-colour picture to a bee's eyesight at a chosen distance and samples it on a hexagonal grid of facets. At 5 cm, a 4 cm-wide frame spans about 44° of a bee's view, so the whole bloom is only about 20 facets across.

## UV and infrared

A normal camera photo holds no UV and no infrared, so both are predicted from the visible colour. UV can be predicted reliably once the species' spectrum has been measured; from colour alone it cannot. Infrared is a clearly labelled estimate until a real IR photo is supplied.

**How UV is filled in, best first:**

1. A real UV-pass photo of the flower, when there is one, replaces everything else.
2. The species' measured spectrum: petal pixels that match its colour take the whole measured curve, scaled by each pixel's brightness. Stamens, centre and buds use method 3.
3. A colour-only estimate: the violet end of the pixel's visible spectrum (400 to 420 nm) carried flat into the UV. Leaves take the measured leaf UV (about 1.5%).

**Test on all 73 measured species** (error is the distance on the bee hexagon between the prediction and the real spectrum):

| Method | Median error | 90% of species within | Within bee threshold (0.062) |
| --- | --- | --- | --- |
| With the species spectrum | 0.000 | 0.001 | 100% |
| Colour only | 0.318 | 0.673 | 18% |

The gap comes from UV pigments that are colourless to us, so petals that look identical can be bright or dark in UV. Black-eyed Susan (*Rudbeckia hirta*) is the classic case: evenly yellow to us, with a UV-dark centre to a bee. Borage petals reflect strongly in UV (7% at 340 nm rising to 28% at 380 nm), which is why a bee sees them as UV-blue.

**Infrared estimate.** Plant tissue reflects near-IR strongly whatever its pigment, leaves most of all (the Wood effect). The estimate keeps the light and shade, drops the pigment colour and lifts the foliage. It stands in for the look of an IR camera; it is not a measurement.

**Measured species.** The pack bundles 73 measured species plus a leaf mean. Six are on the project's 50-plant list: borage, viper's bugloss (*Echium vulgare*), New England aster (listed as *Symphyotrichum novae* in the dataset), Canada goldenrod (*Solidago canadensis*), common yarrow (*Achillea millefolium*) and garden cosmos (*Cosmos bipinnatus*). The set also includes chicory (*Cichorium intybus*).

## Height map, 3D relief and animation

The height map comes from Depth Anything V2 Small, cut to the flower by a U2-Net mask. On borage it puts the stamen cone nearest and the petals falling away behind it, as expected. The relief is the photo lifted by that height map: 4 cm wide and 1.2 cm deep by default, and one-sided, so it is shown from the front.

| Output | Format | Use |
| --- | --- | --- |
| Relief mesh | GLB, textured, Y up, +Z toward the viewer | Previews, Sansar or Unity, the pane as a mesh |
| Relief splat | 3D Gaussian Splatting PLY, one flat splat per grid point | Spark, the pane as a splat |
| Rudder-and-aileron animation | MP4 (640 px, 24 fps, 4 s loop) and frames | Showing the relief moving |

The animation sweeps diagonally: yaw (rudder) of 28° and pitch of 18° move together, and the relief banks 8° into each turn (ailerons). That stays inside the ±30° a hand-held pane tilts. Setting the phase to 90° turns the diagonal into an ellipse.

**Known flaw:** the preview renderer leaves faint dotted gaps on the steepest parts (the hairy sepals) at full tilt. The GLB mesh has none.

## TRELLIS Gaussian blooms

TRELLIS makes a good borage only when its photos show one real flower from different sides. Four front views of different flowers give a muddled, many-petalled star. The best result so far is multidiffusion on four CC BY photos.

| Run | Photos in | Mode | Result |
| --- | --- | --- | --- |
| `03_borage_cc_bloom_multidiffusion` | 4 CC BY: front and 45° of one bloom, underneath and side of another | Multidiffusion | Near five petals, true blue, green sepals, stem. **Best so far** |
| `03_borage_cc_bloom_stochastic` | Same 4 | Stochastic | Similar, petals spread flatter |
| `03_borage_bloom_single_b` | 1 own photo (deep-blue bloom) | Single | 8 to 10 spiky petals |
| `03_borage_bloom_single` | 1 own photo (lilac bloom) | Single | 8 to 10 petals, pale |
| `03_borage_bloom_multidiffusion` | 4 own photos (4 different blooms, all front-on) | Multidiffusion | Pointed star, 8 to 10 petals |
| `03_borage_bloom_stochastic` | Same 4 | Stochastic | Rounder lilac rosette, stamen cone lost |

Each run produced a `.ply` splat, a GLB, a texture and a turntable video. Each run took about a minute on an RTX 4070 Super.

**Lessons for the other plants:**

- One flower from 3 or 4 angles beats 4 flowers from the front. The underneath and side views are what fix the petal count.
- Multidiffusion holds the shape better than stochastic for star-shaped blooms.
- TRELLIS colours run a little lilac, so check against the photo before a bloom becomes a pane's hero.
- Open-licence photos showing more than two angles of one flower are rare; pairing two look-alike blooms worked for borage.

## Bee senses in the viewer

The garden can switch into each bee view inside the renderers, so the Quest, PC VR and the Jupiter SR all draw it natively at no extra cost per frame. It passes all 53 VR comfort checks.

**How it works.** Each view is a 33 × 33 × 33 colour table made by the ComfyUI pack. Spark looks up every splat's colour in it on the graphics card, and plant meshes and the sky do the same per pixel. Six plants use their own measured-spectrum tables: 03 Borage, 20 Viper's Bugloss, 30 New England Aster, 31 Canada Goldenrod, 32 Common Yarrow and 33 Garden Cosmos. Everything else uses the colour-only table. All 43 tables are built into the app (666 KB), so it works offline and in a headset browser.

**Choosing a view.** Clicking a view's image on a plant's info pane switches the whole garden to that view, until the same view is clicked again. Stepping down a platform also returns the garden to normal colours.

**Comfort.** A change dips gently to normal colours (0.3 s) and into the new view (0.45 s), never a hard cut. The bee's-eye blur is never applied to the garden; it stays on the pane.

**When it opens.** After 2 minutes on platform 9, the hive garden, at the end of the programme, or early for a facilitator with `?senses=1` on the address. Once open, it stays open on that device.

**The opening card.** When the app loads, a card shows the bee colour hexagon equations with a short piece about seeing through science.

**The borage pane (stand-in).** Until the full plants pane is merged, a deep-blue borage stands 1.6 m ahead on every platform. Pointing and pulling the trigger (VR) or clicking it (screen) opens its pane, which shows the plant facts, the chosen view as its part-3D splat with caption and credit underneath, and a row of views. Before bee senses unlock, only Our eyes is shown.

**Technical notes (Spark and Three.js)**

- Splats change colour through a Spark dyno modifier; meshes and the sky through `onBeforeCompile`, after `colorspace_fragment`.
- After changing the colour mix, call `updateVersion()` on each splat. Spark only regenerates a splat when its version changes, so without this the garden only changed colour while the head moved.

## What's new about it

We found no prior tool that gives species-calibrated bee vision from ordinary photos and carries it into 3D and a live, explorable garden. The science and some image tools already exist; the combination, as far as a quick search shows, does not. This is not a full prior-art review.

| Already done by others | New in IML BeeSenses |
| --- | --- |
| Bee colour hexagon (Chittka 1992); flower spectra databases (FReD, Bayreuth) | UV predicted from an ordinary photo using the species' measured spectrum: all 73 species within bee discrimination, against 18% from colour alone |
| [Bee-view false-colour photos](https://pollinationecology.org/index.php/jpe/article/view/482), needing a UV-converted camera | No UV camera needed; a real UV photo can still be plugged in |
| [toBeeView](https://onlinelibrary.wiley.com/doi/full/10.1002/ece3.2442): how blurry flowers look to a bee | Views carried into 3D: height maps, part-3D splats, meshes, animation |
| Research tools for still images and videos | Whole-garden bee vision, live, in VR and on a glasses-free 3D screen |
| — | Offered as a reward at the end of a graded exposure programme for fear of bees |
| — | An open, repeatable pipeline: ComfyUI nodes, per-plant views, splats, credits and licences in one format |

## Results for borage

The first full run of borage through IML BeeSenses worked on 29 September 2026: every view, the height map, three reliefs with animations, the pane set and the LUTs, from one ComfyUI workflow.

**Borage to a bee:** UV-blue, 0.18 hexagon units from green leaves, so easy to see. The measured petal spectrum on its own puts it in the same place.

**Outputs per plant:**

- A sheet with every view on one page, plus a report
- The pane set at full size, with `views.json` (captions, which views are estimates, each view's garden mode and credit line) and `report.txt`
- Seven whole-scene `.cube` LUTs for the viewer's modes
- A relief for each of the nine views: GLB mesh, part-3D splat (PLY) and rudder-and-aileron MP4
- The photo, the TRELLIS bloom (`bloom.ply`) and the ComfyUI workflow (`IML_BeeSenses_03_borage`)

## Still to do

- [ ] Check the bee colours, UV and height map against the real borage (sign-off)
- [ ] Replace the stand-in borage pane with the full plants pane
- [ ] Decide which bloom the pane shows by default: the TRELLIS bloom or the photo relief
- [ ] Fill the preview renderer's dotted gaps before the animation is used in public
- [ ] Run the other 49 plants through the modeller (five more with measured spectra, the rest estimated)
- [ ] Publish the node pack and the flower repository on GitHub and Zenodo
- [ ] A companion view on the Jupiter SR for an accompanying person, when the screen arrives

## How it was built

A short timeline of 29 September 2026 (UK time).

- **19:07–19:31** TRELLIS trials on own borage photos, then on four CC BY photos, which gave the best bloom. A source log was started, and open photo collections were checked for usable licences (iNaturalist, GBIF and Wikimedia Commons have CC0 and CC BY material; CO3D and MVImgNet are non-commercial; OmniObject3D needs a sign-up).
- **19:35–19:52** The node pack was started. Measured flower spectra (Shrestha et al., CC BY 4.0) were added, UV prediction was tested across all 73 species, and the pane views were chosen.
- **19:55–20:14** Relief mesh, splat and two-axis animation tested; licensing set (code Apache-2.0, everything else CC BY 4.0); whole-scene LUTs exported, matching the direct calculation within 0.7%; the first full borage run completed.
- **20:40** Splats made from all nine views.
- **20:58–21:48** The bee senses engine was added to the viewer, applying the colour tables inside the renderers to garden splats, plant meshes and the sky, with credits under every view. All 53 VR comfort checks pass.
- **22:00–23:35** The opening card and the borage pane were added. A fault where garden splats only changed colour while the head moved was fixed (see the technical notes above), and the comfort checks were rerun, all passing.

## Sources, licences and credits

Everything used is commercially usable. CC BY items need their credit line wherever the result is shown. A per-photo source log will be published with the flower files.

| Item | Used for | Licence | Credit line |
| --- | --- | --- | --- |
| [Borage photos, obs. 73154978](https://www.inaturalist.org/observations/73154978), Eleftherios Katsillis | TRELLIS input (front, 45°) | CC BY 4.0 | Borage photos © Eleftherios Katsillis, CC BY 4.0, via iNaturalist |
| [Borage photos, obs. 98801998](https://www.inaturalist.org/observations/98801998), Juraj Ahel | TRELLIS input (underneath, side) | CC BY 4.0 | Borage photos © Juraj Ahel, CC BY 4.0, via iNaturalist |
| [Flower reflectance data](https://doi.org/10.6084/m9.figshare.21081025.v2), Shrestha et al. 2024, University of Bayreuth | 73 petal spectra and the leaf mean (UV and bee colours) | CC BY 4.0 | Flower spectra: Shrestha et al. 2024, University of Bayreuth, CC BY 4.0 |
| [Depth Anything V2](https://github.com/DepthAnything/Depth-Anything-V2) code and Small weights | Height maps | Apache-2.0 | Height maps by Depth Anything V2 (Apache-2.0) |
| [TRELLIS](https://github.com/microsoft/TRELLIS) | Gaussian blooms | MIT | Not required; included as a courtesy |
| Own borage photos (03_borage_1 to 4) | Tests and the pane photo | Own work | Hope Abeacan / Ion Music Live |

**Not usable here:** Depth Anything V2 Base, Large and Giant weights (CC BY-NC); CO3D and MVImgNet (non-commercial); CC BY-SA images (the result would have to carry the same licence).

**Science references used in the model:** Peitsch et al. 1992 (bee receptor peaks), Govardovskii et al. 2000 (pigment template), Chittka 1992 (colour hexagon), Wyman, Sloan and Shirley 2013 (colour-matching fit), CIE D65 daylight.
