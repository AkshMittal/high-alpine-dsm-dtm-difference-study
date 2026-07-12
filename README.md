> **Note: the analysis code in this repo is outdated.**
> It derives slope and aspect from the DTM, which is both the layer under investigation and
> the horizontally less accurate of the two products. The conclusion it reaches (horizontal
> misregistration, evidenced by an aspect dipole) does not hold. See
> [Correction and post-mortem](#correction-and-post-mortem) at the bottom for what was wrong,
> what was tested, and what survives. The code is left as-is rather than rewritten, so the
> original reasoning stays inspectable.

# DSM − DTM artifact analysis over alpine glaciers

This started as a data check for a viewshed project and became a study of what the
difference between two swisstopo elevation models actually measures in high-alpine terrain.

Short answer: it's  measurement artifact for sure, but real height difference not proven 
since the datasets used were different epochs. And the artifacts come from two different 
mechanisms in two different places.

## The question

To place an observer and compute sightlines you need a bare-earth model (DTM) and a
surface model (DSM). The difference (DSM − DTM) is normally canopy and buildings. But my
study area (Weisshorn) is entirely above the treeline, so there's no vegetation. So what
is the difference measuring? That's what this investigates.

## What I found

The extreme differences are acquisition artifacts for sure, and they split into two groups:

- **Inside glaciers (89% of extreme cells):** rugged, crevassed ice. The difference is large
  because the two acquisitions are 3 years apart over moving, melting ice, and because
  stereo-matching fails on smooth bright snow. High-difference cells are clearly rougher
  (higher TRI) than calm cells. misregistration probably comes into play here too as they 
  come in outside of galciers.

- **Outside glaciers:** near-vertical cliffs. On a near-vertical face, a small horizontal
  misregistration between the two acquisitions (sub-meter) turns into a huge vertical
  difference, because vertical geometry amplifies horizontal offset. Extreme-difference
  cells outside glaciers have a median slope of ~70°, while calm terrain is a normal
  bell around 40°. The offset is directional, so positive and negative errors sit on
  opposite-facing slopes — an aspect dipole, which is the confirming signature.

## Evidence

- Symmetric distribution of the difference around zero (real features would be
  positive-only; symmetry points to measurement position disagreement).
- GLIMS glacier inventory (independent data): median difference 50x higher inside glaciers,
  89% of extreme cells inside.
- TRI (ruggedness) is clearly higher for high-difference cells, on both the DSM and DTM.
- Slope and aspect analysis of the outside-glacier extremes (see figures).

## Figures

![TRI of high vs low difference cells](figures/TRI_highlow.png)

High-difference cells (blue) sit at much higher ruggedness than calm cells (orange).

![Slope of extreme vs calm cells outside glaciers](figures/slope_outside.png)

Extreme-difference cells outside glaciers pile up against 90° (cliffs); calm terrain is a
normal bell around 40°.

![Aspect dipole](figures/aspect_dipole.png)

Positive and negative extremes sit on opposite-facing slopes — the signature of a
directional horizontal offset between the two acquisitions.

## What this is (and isn't)

This isn't a canopy analysis anymore — with no vegetation above the treeline, DSM − DTM
is effectively a DEM difference that should represent alpine terrain change, closer to
glacier DEM-differencing than to a canopy model. But here both the sensors and the epochs
were different, which did not amount to a proper glacier study. That was not the point
either, but an evaluation of why I was seeing difference values in such big clusters needed
to be done.

High-difference values are concentrated on rough terrain inside glaciers (icefalls, broken
zones) and on cliff edges outside. I want to be precise about what that does and doesn't
show: I've shown, statistically through TRI, that the values *cluster* where crevassed/broken
ice is. I have NOT delineated crevasse features or validated them against imagery, so I'm
not claiming this "maps crevasses." That would need polygonising the high-difference zones
and the low constant zones and checking them against optical imagery, which I haven't done
here because it isn't right to do given the proven sensor misregistration artifacts. Those
values are ambiguous in their source of origin (sensor artifact or true change), so I deferred.

The natural next step is proper glacier DEM-differencing with matched sensors, to separate
real surface change from the sensor artifacts this study found — where I have some ideas,
like how smooth snow/ice in a different-epoch DEM difference should be a smooth transition
or near-constant value, whereas positional glacier change through movement over time should
show high-magnitude values, both positive and negative.

## Data

- swisstopo swissSURFACE3D (DSM, LiDAR, 2021) and swissALTI3D (DTM, 2024) — https://www.swisstopo.admin.ch
- Glacier outlines: GLIMS — https://www.glims.org

Tiles are not included in the repo (too large); download from the sources above.

## Limits

Single site (Weisshorn). The DTM is likely stereo-derived above 2000m but this is somewhat
ambiguous, so I describe the outside-glacier mechanism as horizontal misregistration between
two acquisitions rather than pinning it to a specific method difference.

## Correction and post-mortem

The original conclusion in this repo was wrong. This section explains how, and what
survives.

### What I claimed

That the extreme DSM − DTM values outside glaciers were caused by horizontal
misregistration between the two acquisitions, evidenced by an "aspect dipole": positive
and negative extremes sitting on opposite-facing slopes, which is how a horizontal shift
would behave.

### Why it was wrong

**The dipole was never demonstrated.** The evidence was two separated peaks in a marginal
histogram of aspect for positive and negative extremes. But a dipole is a claim about
*pairing*: that a given positive extreme sits opposite a given negative one on the same
feature. A marginal distribution cannot show that. Two populations can prefer different
aspects for unrelated reasons with no cell-to-cell relationship at all.

Tested directly, by conditioning on proximity first and then measuring aspect separation
between neighbouring opposite-sign extremes:

- median aspect separation between paired extremes: **12.4°** (i.e. same-facing)
- fraction of pairs more than 120° apart: **1.1%**

A Nuth-Kääb style coregistration fit over all off-glacier terrain independently returned
**R² = 0.003**. There is no global horizontal offset between these two products.

**A methodological error contributed.** Slope and aspect were derived from the DTM. The DTM
is the layer under investigation, and it is the horizontally inferior one (swissALTI3D
above 2000 m is stereocorrelation, ±1–3 m; swissSURFACE3D is LiDAR, ~±0.2 m horizontal).
Using it as the geometric reference frame is circular. Slope and aspect should be derived
from the LiDAR DSM. This is corrected in the analysis, though it is not what produced the
false conclusion.

### What was tested and ruled out

Four candidate mechanisms, all rejected against data:

| Hypothesis | Prediction | Result |
|---|---|---|
| Global horizontal misregistration | pos/neg extremes on opposing aspects; cosine fit recovers shift | 12.4° median separation, 1.1% opposed; fit R² = 0.003 |
| TIN interpolation across data voids | DTM locally *smoother* than DSM inside extreme patches | TRI ratio DTM/DSM = **1.568** on extremes (rougher), 0.978 on calm |
| Random stereo matching noise | signs scatter between adjacent cells | sign-flip rate **0.0018** on extremes vs **0.266** on calm; extremes are sign-coherent |
| Production seam at the glacier polygon edge | effect pinned to the polygon, sharpens under erosion | insensitive to 5/10/25 m erosion; effect is ~100 m wide, far too broad for a seam |

### What survives

Description, not mechanism:

- Extremes form large, internally **uniform** patches (median CV = std/|median| = **0.148**),
  bodily displaced by tens of metres rather than smoothly graded.
- They sit on near-vertical terrain (median patch slope **68.7°**).
- They are concentrated within roughly 100 m of the glacier margin, ~30× enriched even with
  slope held constant (55–75° band).
- Inside these patches the DTM's derived aspect disagrees with the DSM's by a median of
  **28.8°**, while on calm terrain the two agree to within **1.3°**. The DTM is locally wrong
  where the extremes are, and correct everywhere else.

This is consistent with stereo-correlation failure in the 2024 photogrammetric DTM, on the
steep, poorly-illuminated, low-texture terrain that swisstopo's own product documentation
flags as problematic (swissALTI3D product info, §3.2.5 and §4). I am not claiming it as
demonstrated. Distinguishing between specific photogrammetric failure modes would require
access to the source stereo pairs, which I don't have.

### The lesson

The error was not a bug. The code did what I told it to. The error was reading a *mechanism*
out of a *marginal distribution*, when the mechanism made a claim about pairing that the
marginal could not test. The conditional test took ten minutes to write and killed the
conclusion immediately. I should have run it before publishing, not after. Also towards the end 
I was not able to identify anything else specific that could explain the values and their patterns, 
every test was exclusionary. Several photogrammetic phenomenons could explain this clubbed together, 
but I could not distinguish between them. 
