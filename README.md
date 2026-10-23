# Multimodal tomato growth prediction 

**Question:** does combining top-down RGB plant images with greenhouse sensor data improve the prediction of
plant growth compared with either modality alone?

**Short answer:** not in this dataset. The numerical model (plant measurements on day 0 + zone sensor history) is the
best model under leave-one-camera-out validation (MAE 6.8 cm vs 8.4 cm for the mean baseline). None of the fusion
models beats it; the differences are inside bootstrap noise, and adding many image features makes results slightly worse.
The reasons (top-down camera vs. a height target, overlapping canopy, 60 plants, 6 sensor zones) are analysed in the notebooks
and summarised in [`reports/technical_summary.md`](reports/technical_summary.md).

---

## Repository structure

```
tomato-growth/
├── README.md
├── requirements.txt
├── .gitignore
├── notebooks/
│   ├── 01_data_prep.ipynb              # loading, cleaning, EDA (sensors, plants, images), assumptions
│   ├── 02_numerical_model.ipynb        # target definition, sensor/phenotype features, numerical models
│   ├── 03_image_model.ipynb            # image regions, illumination correction, image features, image models
│   └── 04_fusion_error_analysis.ipynb  # fusion strategies, final comparison, bootstrap, error analysis
└── reports/
    ├── technical_summary.md            # one-page summary (approach, results, limitations, next steps)
    ├── biological_reasoning.md         # answers to the biological-reasoning questions
    ├── production_design.md            # deployment design (< 300 words)
    └── figures/                        # key figures exported from the notebooks
```

All notebooks are committed **with their outputs**, so results can be read without re-running anything.

---

## Data

The work uses the curated June 2025 subset of the **Horti-M3-Tomato** dataset provided for the case study
(`25 tomato dataset/2025/202506/`): three sensor devices (30-min readings), hand measurements for 60–108 plants,
and 745 top-down images from 5 cameras. The data is **not** included in this repository.

Expected layout (as provided):

```
25 tomato dataset/
├── README.md, Fertilization Scheme.docx, Plant Orientation Diagram.png, ...
└── 2025/202506/
    ├── raw_data/sensor_data/sensor1.xlsx, sensor2.xlsx, sensor3.xlsx
    ├── raw_data/growth_index/20250607.csv, 20250613.csv
    ├── processed_data/growth_index(final)/20250601.csv … 20250613.csv
    └── images/11,411/  111,211/  214,311/  511,611/  514,614/
```

---

## How to reproduce

### Option A: Google Colab (what was used)

1. Upload the `25 tomato dataset` folder to Google Drive (e.g. `MyDrive/25 tomato dataset`).
2. Open the notebooks in Colab **in order** (01 → 02 → 03 → 04).
3. In the first code cell of each notebook check the line
   `DATA_DIR = Path('/content/drive/MyDrive/25 tomato dataset')` and adapt it if your folder is elsewhere.
4. `Runtime → Run all`. Each notebook mounts Drive and writes its outputs to `MyDrive/alprotein_outputs/`
   (next to the data folder); the next notebook reads them from there.

| Notebook | Runtime | Approx. time |
|---|---|---|
| 01 | CPU | ~10 min (opens all 745 images once at low resolution; cached afterwards) |
| 02 | CPU | ~2 min |
| 03 | **GPU recommended** (T4) | ~10–15 min (ResNet-50 features; cached afterwards) |
| 04 | CPU | ~10–15 min (many cross-validation runs) |

If you change a notebook, re-run it **and every notebook after it**: the notebooks communicate only through the
files in `alprotein_outputs/`. Notebook 03 caches image features; set `FORCE = True` there if you change the image feature code.

### Option B: local machine

```bash
pip install -r requirements.txt
export DATA_DIR="/path/to/25 tomato dataset"
jupyter lab notebooks/
```
Outside Colab the notebooks read `DATA_DIR` from the environment. The pretrained ResNet-50 weights are downloaded
by torchvision on first use (internet needed once).

Random seeds are fixed (`SEED = 42`); results are deterministic up to small numerical differences between library versions.

### Files produced in `alprotein_outputs/`

| File | Written by | Content |
|---|---|---|
| `sensors_long.parquet`, `sensors_zone_30min.parquet` | 01 | cleaned sensor readings; zone-level 30-min table |
| `growth_daily.parquet`, `growth_measured_all.parquet` | 01 | plant measurements (60 plants × 13 days with `is_measured`; 108 plants × 2 real days) |
| `image_index.parquet`, `image_quality.parquet`, `plant_map.csv` | 01 | image index and quality stats; plant → camera → sensor zone |
| `model_table_numerical.parquet`, `results_numerical*.csv`, `target_window.json` | 02 | features, target and numerical results |
| `image_features_plant.parquet`, `image_regions.parquet`, `cell_mapping_validation.csv`, `results_image.csv` | 03 | image features and results |
| `results_final.csv`, `bootstrap_comparison.csv`, `predictions_per_plant.csv`, `figures/` | 04 | final comparison and error analysis |

---

## Approach in brief

- **Target:** plant-height gain between the only two days with real hand measurements (7 → 13 June).
  The other daily values in `growth_index(final)` are interpolated and are never used as targets.
- **Numerical model:** the plant's own measurements on 7 June (height, stem, leaves, NDVI/LAI/…, variety) +
  conditions of its sensor zone during 7 → 13 June (temperature, growing degree-days, light, RH, VPD, daytime CO₂, soil-moisture change).
- **Image model:** automatic walkway detection, per-image illumination correction using the walkway as a grey reference,
  beds split into a 2 × 3 grid of plant cells, hand-crafted features (canopy fraction, flowers, fruit, greenness, texture)
  and frozen ResNet-50 embeddings (PCA inside each fold). Two settings: **forecast** (7 June images only) and
  **estimation** (7 + 13 June images).
- **Fusion:** early fusion (feature concatenation) and late fusion (averaging separately trained models).
- **Evaluation:** MAE, RMSE, R² from out-of-fold predictions; **leave-one-camera-out** CV as the headline scheme
  (random repeated 5-fold reported for comparison); paired bootstrap on per-plant errors; detailed error analysis of the
  most under- and over-estimated plants with images and environmental history.

## Main assumptions

1. Only 7 and 13 June are real measurements; the target is the height gain between them.
2. Measurements were taken in the morning, so the sensor window is [7 June 00:00, 13 June 00:00).
3. A plant experiences the climate of its treatment zone. T3 and T4 have no sensor, and T2/T5 have no data after 10 June; these get the greenhouse mean and are flagged (`zone_is_proxy`).
4. Sensor readings of exactly 0 (temperature, RH, CO₂) are device drop-outs.
5. Soil probes are not calibrated against each other, so only changes relative to each probe's own baseline are used.
6. Image file names hold the capture time as Unix time (UTC); local time = UTC + 8 h.
7. The plant position in an image follows the Plant Orientation Diagram. This mapping was **tested** (notebook 03): cells do not
   isolate single plants reliably, so image information is effectively bed-level.
8. Treatment differences are described, not interpreted causally (3–6 plants per group, confounded with greenhouse position).

The full list is at the end of notebook 01.

---


## Requirements

See [`requirements.txt`](requirements.txt). Main libraries: pandas, numpy, scikit-learn, scipy, matplotlib, Pillow,
pyarrow, openpyxl, torch, torchvision.
