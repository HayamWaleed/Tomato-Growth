# Production design

**Numerical data and alignment.** Sensor readings (every 30 min) go to a time-series store. On arrival: range checks (drop-outs written as 0 become missing),
timestamps snapped to a 30-min grid in one time zone, probe → zone mapping from a versioned config table. Features are computed only from data before the prediction time.

**Images.** One image per camera per day at a fixed hour (~10:00). Quality gate on sharpness, brightness and green fraction rejects foggy or dark images. Walkway detection, white balance on the walkway, and crops from a stored
camera layout. Each image is matched to its sensor window through camera → bed → zone and capture time (UTC to local).

**Missing data.** Missing sensor: fall back to the greenhouse mean and flag the prediction. Missing or rejected image: the late-fusion design
falls back to the numerical model, flagged "no image". Targets are never imputed.

**Inference and logging.** A daily batch job logs data versions, features, model version, prediction and flags,
for later comparison with hand measurements.

**Versioning.** Code in git; raw data snapshots and feature tables versioned by date (e.g. DVC); models in a registry (e.g. MLflow) with
training-data version, metrics and validation scheme.

**Monitoring.** Data drift: per-feature distribution tests against training data, image brightness/sharpness, sensor uptime.
Performance: whenever new hand measurements arrive, MAE per camera/zone versus the baseline; alerts on thresholds.

**Retraining.** After each measurement campaign or a drift alert. Validation uses leave-one-camera-out and time-based splits; a new model is
promoted only if it beats the current model and the baseline (bootstrap interval), after a shadow period.

**For growers.** A per-bed dashboard: expected weekly growth with an uncertainty band, flags (growth below peers, missing data, no own sensor)
and the latest image. Presented as an early warning to verify by hand, given the current accuracy.
