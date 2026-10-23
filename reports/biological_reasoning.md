# Biological reasoning

All statements below are **predictive associations** found in 60 (or 108) plants over one 6-day interval. Sensor zones
are confounded with fertiliser treatment and greenhouse position, so none of them is a causal claim.

## 1. Which environmental variables are most associated with the target?
Spearman correlation with height gain (7 → 13 June, 60 plants):

| Positive | ρ | Negative | ρ |
|---|---|---|---|
| daytime CO₂ | +0.34 | soil temperature (relative to probe baseline) | −0.37 |
| soil-moisture change | +0.30 | light sum / daytime light | −0.33 |
| VPD | +0.10 | minimum daily temperature | −0.31 |

Variety (cherry +0.44) and stem diameter on 7 June (+0.32) are stronger than any environmental variable.
These sensor features take only **6 distinct values** (one per zone), so each correlation rests on ~6 independent points.
The negative light association is a clear example of **confounding**: the T6 zone has the highest recorded light
(its sensor is placed differently: T1 and T6 read about twice the light of the other zones under the same roof) and T6 plants had the lowest median gain
in both varieties (11 and 14 cm across 108 plants), plausibly because of their fertiliser regime or position. With this design the effects cannot be separated.

## 2. What does the image contribute that the sensors do not capture?
- **Plant state:** canopy cover, greenness (a proxy for leaf nitrogen; greener cells on 7 June were associated with more growth, ρ = +0.41), flowers and fruit.
- **Management events:** pruning and de-leafing. L113's cell lost twice as much canopy as its peers, matching its negative height change; canopy cover drops
  steadily in the second half of June, consistent with removal of lower leaves during fruiting.
- **Within-zone variation:** one sensor serves a whole zone; the camera sees each bed.
- **Data-quality problems:** fog/condensation on the lenses on 10 June (all cameras), missing images.

In this dataset these signals did not improve height prediction (top-down view, overlapping canopy, small sample).

## 3. Which important biological or experimental variables are missing?
Actual fertiliser and irrigation timing/amounts per plant; pruning, de-leafing and stem-training events; pests and diseases linked to individual
plants (the growth record table is per treatment, not per plant); fruit load (fruit sinks compete with vegetative growth); developmental stage per plant;
microclimate at plant level; sensor calibration and light units; the exact time of day of the hand measurements.

## 4. How would these factors affect the interpretation?
- **Genotype:** Jinzhu is an indeterminate cherry type and grew more in every treatment; variety must always be in the model, and effects can differ by variety (e.g. T4).
- **Nutrient availability:** T6 grew least, but treatment is confounded with position and each group has only 3–12 plants; a randomised layout would be needed.
- **Disease:** can reduce growth and greenness without any change in sensor readings, so it would look like unexplained low growth.
- **Pruning / topping:** directly reduces height or leaf area (L113); unrecorded, it looks like "poor growth" to a model.
- **Developmental stage:** in fruiting, assimilates go to fruit, so vegetative growth slows; plants at different stages respond differently to the same climate.
- **Irrigation history:** soil-moisture change appears associated with growth, but probes are uncalibrated and few; the link could reflect probe placement.
- **Measurement variability:** measuring ~2 m plants by hand has an error of a few cm, a large share of a ~19 cm mean gain. Implausible values (−13 cm, +57 cm, a +27 cm change in leaf width) point to this.

## 5. Predictive relationships vs. claims that need experiments

| Observed predictive association | What would be needed before a biological claim |
|---|---|
| Cherry variety grows more | Consistent and biologically expected; confirm with randomised plot layout |
| T6 grows least | Randomised treatment positions, replicated over seasons |
| More light ↔ less growth | Almost certainly confounding; controlled light experiment with calibrated sensors |
| Higher daytime CO₂ ↔ more growth | CO₂ enrichment experiment |
| Greener leaves on day 0 ↔ more growth | Measured leaf N + fertiliser dose–response trial |
| Canopy drop ↔ pruning | Cross-check with dated farming records per plant |
