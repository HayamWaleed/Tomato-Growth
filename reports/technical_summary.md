# Technical summary

**Question.** Does combining top-down RGB images with greenhouse sensor data predict tomato growth better than either modality alone?
**Answer.** No, not in this dataset: a model on plant measurements + zone sensors is best, and no fusion strategy beats it.

## Data and target
June 2025 subset of Horti-M3-Tomato: 2 varieties (Guanghui 201, Jinzhu cherry) × 7 fertiliser treatments; 3 sensor devices
(30-min readings, 5 zones; T3/T4 have no sensor and T2/T5 stop on 10 June); 745 top-down images from 5 cameras (each shows 2 beds = 12 plants).
Hand measurements exist for **only two days (7 and 13 June)**; the other daily values in the processed files are interpolated.
**Target:** height gain 7 → 13 June for the 60 plants that have images (mean 18.9 cm, SD 10.8, range −13 to +57).

## Approach
- **Numerical:** 7-June phenotype (height, stem, leaf size, NDVI, LAI, …, variety) + sensor features of the plant's zone over the interval
  (temperature, GDD, light, RH, VPD, daytime CO₂, soil-moisture change relative to each probe's baseline). Zones without data use the greenhouse mean (flagged).
- **Image:** walkway detected automatically → per-image white balance using the walkway as a fixed grey reference → beds split into a 2 × 3 grid of plant cells →
  hand-crafted features (canopy fraction, flowers, fruit, greenness, texture, brightness) + frozen ResNet-50 embeddings (PCA 10, fitted inside folds).
  *Forecast* setting uses 7-June images only; *estimation* setting also uses 13-June images.
- **Fusion:** early (concatenated features) and late (average of separately trained models).
- **Validation:** out-of-fold predictions with **leave-one-camera-out** (headline) and repeated random 5-fold; predictions clipped to the training range; paired bootstrap (5000×) on per-plant absolute errors.

## Results (leave-one-camera-out, 60 plants)

| Model | MAE (cm) | R² |
|---|---|---|
| Baseline (training mean) | 8.37 | −0.05 |
| **Numerical (phenotype + sensor), RF** | **6.81** | **0.20** |
| Image only, t0 + t1 (RF) | 8.38 | 0.00 |
| Image only, t0 forecast (RF) | 7.66 | 0.01 |
| Late fusion, forecast | 7.06 | 0.17 |
| Late fusion, estimation | 7.34 | 0.18 |
| Early fusion, all image features | 7.78 | 0.09 |

- Numerical beats the baseline (bootstrap 95 % CI of the MAE gain: 0.58 to 2.45 cm).
- **No fusion model beats numerical-only**; all CIs contain 0 and point estimates are slightly worse. Early fusion with all embeddings is close to significantly worse.
- Image models score R² ≈ 0.20 with random folds but ≈ 0 for unseen cameras: they mostly learn camera/bed identity, not plant growth.

## Why images did not help
1. The camera looks down; the target is stem elongation (height), which is mostly invisible from above.
2. A within-bed permutation test shows cells do **not** isolate single plants (1 of 24 cell-feature/manual-measurement pairs significant, ≈ chance) because the canopy is closed → image information is bed-level, the target is plant-level.
3. Lighting: 13-June images are brighter in every camera; the walkway correction halves the spurious canopy change (0.060 → 0.029) but cannot remove it fully.
4. 60 plants, 5 cameras and 6 sensor zones: thousands of image features overfit.

## Error analysis (example)
**L423** (cherry, T4) grew 57 cm; every model predicted ~21–25 cm. T4 has no sensor, so the "favourable" temperature and light the model saw are the
greenhouse mean, not this plant's conditions; the top-down image shows no visible change; the plant was 20 cm shorter than its peers on 7 June with higher NDVI
(possible catch-up growth, re-training on the string or a measurement error). Across all plants, those without their own sensor are under-predicted by +4.4 cm on average vs −1.6 cm for the rest.
**L113** (height −13 cm) is not growth: its cell lost twice as much canopy as its peers after illumination correction, consistent with pruning/topping, an event no input records.

## Limitations
One growth interval; small sample; sensor zones confounded with treatment and position; no sensor for two treatments; interpolated daily data; foggy images on 10 June;
no records of pruning or irrigation per plant. A sensitivity check with all **108** hand-measured plants (no images) drops the numerical model to R² ≈ 0, so even the numerical signal is weak and unstable.

## With more time
More frequent hand measurements (a real time series), one sensor per bed, a side-view or depth camera for height, plant instance segmentation instead of a fixed grid,
linking the farming/pruning records to plants, and a plant-specific pretrained backbone.
