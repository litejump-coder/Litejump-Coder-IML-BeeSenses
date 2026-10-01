# BeeSenses Project Summary

Oct 1, 2026 · @hope abeacan

## At a glance

BeeSenses lets people see a flower the way a honeybee sees it. From an ordinary photo, it works out the ultraviolet, the bee's colours and an infrared look, then turns each view into a small 3D image you can hold up and turn.

It is part of my app *A Gift For People With Fear Of Honeybees*, a calm, step-by-step garden programme for people who are frightened of bees. Seeing through a bee's eyes is the reward at the end: once someone is comfortable around bees, the whole garden can switch into bee vision.

The first flower, borage, is fully done. The tool is ready to run on the other 49 plants.

- **Built by:** hope abeacan (Ion Music Live), working with Claude
- **Made for:** the Jupiter SR glasses-free 3D screen and VR headsets, as equal partners
- **Licence:** code Apache-2.0, content CC BY 4.0, so others can use and build on it

## What's been built so far

Most of this came together in a long session on 29 September 2026, building on the garden app from the days before.

| What | In plain words |
| --- | --- |
| BeeSenses tool | Seven ComfyUI nodes that turn one flower photo into every bee view, a height map and 3D versions |
| Nine views per flower | Our eyes, bee colours, UV, each bee receptor (UV, blue, green), bee colour map, bee's-eye and infrared |
| 3D for every view | A mesh, a Gaussian splat and a short video moving on two axes, like rudder and ailerons |
| Whole-garden bee vision | Colour tables that switch the entire garden, sky and plants into a bee view |
| Borage info pane | A borage on every platform; open it and the pane shows facts, the views and their credits |
| Opening card | The bee colour hexagon equations with my words about seeing through science |
| 3D blooms | TRELLIS Gaussian splat blooms; best result from four CC BY photos of real borage |
| Credits log | Every outside source logged, with its credit line under each image |
| Plant database | Bee-senses details added to my 50-plant Bee Plants XR list; borage filled in |

**What we learned along the way**

- UV can't be guessed from colour alone, only 18% of flowers come out right that way. Using each species' measured petal spectrum, all 73 tested species come out right.
- Six of my 50 plants already have measured spectra: borage, viper's bugloss, New England aster, Canada goldenrod, common yarrow and garden cosmos.
- For 3D blooms, one flower photographed from three or four sides works far better than four flowers from the front.
- Borage looks UV-blue to a bee and stands out clearly from its leaves.
- The app still passes all 53 VR comfort checks with bee senses added.

## The ten garden platforms

The app walks a person through ten gardens, each a gentle step closer to bees. The gardens are made in World Labs Marble, and every one is meant to feel beautiful and safe.

1. **Home** – a calm terrace, no bees
2. **Calm garden** – a quiet pond, getting settled
3. **Bee at a distance** – one bee, far away
4. **One predictable bee** – a bee on a clump of borage
5. **Guided path** – bees along a flowering path
6. **Your own pace** – you choose how close to walk
7. **A small group** – a few bees, well spaced
8. **Choose a flower** – the bee starts to notice you
9. **Full garden** – a complete country garden
10. **The hive** – a hopeful ending; bee senses unlock here

A separate bee behaviour spec sets out how the bee moves and sounds on each platform. On 30 September I chose a modern, stylised look for the worlds, with a greenhouse, village houses and some Mediterranean touches.

## Parallel project: Gaussian splats and the flower repository

Alongside the app, I'm building an open library of flowers in 3D. Each flower will have its Gaussian splat bloom, all nine bee views, a height map, its facts and full credits, released on GitHub and Zenodo under CC BY 4.0.

Gaussian splats capture real things as millions of tiny soft points of colour, so a flower keeps its true look, sheen and depth rather than looking like a modelled object. That makes them a natural fit for glasses-free 3D screens like the Jupiter SR, where a bloom can float just in front of the glass.

- **Capture:** my camera equipment including a Lab Pano 360 Pilot EE and my 100mp mobile telephone, with a full pipeline already working from capture to trained splat (about 8 minutes to train on my PC)
- **Already proven:** splats import and render properly in Sansar, with full view-dependent shading
- **Blooms from photos:** TRELLIS turns a few photos of one flower into a splat bloom partially correctly trained.
- **Bee senses on splats:** the app already recolours splats into bee vision on the graphics card
- **Why it helps:** researchers, teachers and other developers could use the same flowers, and every new flower makes the bee app richer

## Why it matters: the Gradual Exposure Method

The aim is for people with a moderate fear of honeybees to genuinely overcome it, and feel brave enough when they meet real bees. Bees are the first example; the same method could help with spiders, snakes, flies and other common fears.

Research on spider and height fears suggests the scenery doesn't need to be photo-real. What matters is that the bee moves and sounds like a real one. So the gardens can be stylised and light to run, while the bee stays lifelike.

One interesting open question is what a glasses-free 3D screen like the Jupiter SR does to a fear response. Nobody has tested that for bees yet, and this project is set up to find out.

## Where things stand

The core is working, though it still has a very long way to go: the bee senses tool, the first flower, whole-garden bee vision and a working app build. Everything is saved and backed up however the app is at present, waiting of important development.

At present, the two parts I'm most excited about are seeing flowers as Gaussian splats, in their natural state in 3D, and the bee therapy itself.

Watch this space!
