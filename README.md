# PRONOSTIA Bearing RUL Prediction

## Requirements

- Python 3.10 (64-bit; verified environment)
- Recommended memory: 16 GB or more

Install the environment from this directory:

```powershell
py -3.10 -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
jupyter lab
```

After JupyterLab opens, select the `.venv` Python kernel.

## Use the prepared dataset

Open `data_process.ipynb`, select the `.venv` Python kernel, and run the cells through `pronostia_loader`.

```python
artifact = PROJECT_ROOT / "artifacts" / "pronostia_cache" / "official"

train_loader = pronostia_loader(artifact, "train", batch_size=32)
test_loader = pronostia_loader(artifact, "test", batch_size=32)
```

Development folds are available under `artifacts/pronostia_cache/fold_0` through `fold_5`.

## Rebuild from raw data

Place the raw dataset at:

```text
data/raw/pronostia/
|-- Learning_set/
|-- Test_set/
`-- Full_Test_Set/
```

Open `data_pipeline_scripts.ipynb`, restart the kernel, and select **Run All**. A new output is created under `runs/`.

The clean rebuild extracts 37 handcrafted baseline features per acquisition: vibration time-domain statistics, spectral-kurtosis summaries, BPFO/BPFI/BSF harmonic bandpower, and causal temperature statistics. Fold-specific normalization is fitted only on the training bearings. The normalized feature sequence is available from each loader batch as `handcrafted_features`; missing temperature-derived features are identified by `handcrafted_feature_mask`.

To use other locations, set these environment variables before starting Jupyter:

```powershell
$env:PRONOSTIA_RAW_ROOT = "E:\datasets\pronostia"
$env:PRONOSTIA_OUTPUT_ROOT = "E:\outputs\pronostia_run"
jupyter lab
```

No notebook source changes are required when the package is moved to another computer or drive.

## Run training and evaluation

The prepared cache is already included. To reproduce the delivered training and evaluation results, restart the kernel before each notebook and run them in this order:

1. `01_training_data_gate.ipynb`
2. `02_handcrafted_baselines.ipynb`
3. `03_deep_learning_training.ipynb`
4. `04_evaluation_and_ablation.ipynb`

Detailed outputs, execution behavior, metrics, and known limitations are documented in `TRAINING_README.md`. The material for the second review is in `REVIEW_2_REPORT.md`.
