# PRONOSTIA Bearing RUL Benchmark

## 1. Benchmark scope

This document compares the implemented Remaining Useful Life models on the PRONOSTIA dataset.

Two evaluation scopes are reported separately:

- Development validation: six leave-one-bearing-out folds using the six learning bearings.
- Official test: eleven independent PRONOSTIA test bearings with positive endpoint RUL targets.

Development results are used for model selection. Official test results are used only after a model configuration has been frozen.

## 2. Evaluated models

| Model | Type | Input | Purpose |
|---|---|---|---|
| Naive Median | Naive baseline | Training RUL labels | Minimum reference baseline |
| Random Forest | Traditional Machine Learning | 37 handcrafted features | Handcrafted-feature benchmark |
| VibrationOnlyBaseline | Deep Learning baseline | Vibration | Single-modality comparison |
| TemperatureOnlyBaseline | Deep Learning baseline | Temperature and mask | Single-modality comparison |
| LateConcatBaseline | Multimodal baseline | Vibration and temperature | Simple fusion comparison |
| CNNBiLSTMAttentionRUL | Proposed main model | Vibration, temperature, and masks | Main research architecture |
| MainModel_NoAttention | Ablation | Vibration, temperature, and masks | Tests the contribution of Attention |
| MainModel_NoModalityDropout | Ablation | Vibration, temperature, and masks | Tests modality-dropout robustness |

The proposed main model is `CNNBiLSTMAttentionRUL`. The current best development candidate is `OPT2_06`, which belongs to `VibrationOnlyBaseline` and is not the proposed multimodal model.

## 3. Metrics

| Metric | Meaning | Direction |
|---|---|---|
| Window MAE | Mean absolute error across all sequence windows | Lower is better |
| Window RMSE | Root mean squared error across all sequence windows | Lower is better |
| Endpoint MAE | Mean absolute error at one endpoint per bearing | Lower is better |
| Endpoint RMSE | Root mean squared endpoint error | Lower is better |
| Trajectory NMAE | Bearing-level trajectory MAE normalized by lifetime | Lower is better |
| Trajectory NRMSE | Bearing-level trajectory RMSE normalized by lifetime | Lower is better |
| IEEE PHM 2012 score | Asymmetric official endpoint accuracy score | Higher is better; best value is 1 |

The development endpoints have an actual RUL of zero. Therefore, the percentage-based IEEE PHM score is undefined for development endpoint evaluation and is reported only for the official test set.

## 4. Six-fold development benchmark

The development benchmark contains 468 validation windows and six validation bearing endpoints.

| Model | Window MAE (s) | Window RMSE (s) | Endpoint MAE (s) | Endpoint RMSE (s) |
|---|---:|---:|---:|---:|
| Naive Median | 6,043.4 | 8,241.8 | 6,401.7 | 6,460.8 |
| Random Forest | 6,859.8 | 8,930.6 | 2,553.8 | 3,197.3 |
| CNNBiLSTMAttentionRUL | 5,396.8 | 7,929.6 | 2,979.8 | 3,190.2 |
| MainModel_NoAttention | 5,100.7 | 7,680.1 | 2,552.0 | 3,072.2 |
| MainModel_NoModalityDropout | 5,702.3 | 8,191.5 | 4,089.0 | 4,783.9 |
| LateConcatBaseline | 5,248.4 | 7,420.4 | **1,326.4** | 1,960.6 |
| TemperatureOnlyBaseline | 5,570.1 | 7,740.8 | 5,839.7 | 6,011.6 |
| VibrationOnlyBaseline | **4,964.8** | **6,987.3** | 1,738.0 | **1,947.8** |

Development findings:

- `VibrationOnlyBaseline` has the best Window MAE, Window RMSE, and Endpoint RMSE among the original model suite.
- `LateConcatBaseline` has the best Endpoint MAE among the original model suite.
- `CNNBiLSTMAttentionRUL` does not outperform the vibration-only model.
- `MainModel_NoAttention` outperforms the main model, so the current ablation does not demonstrate an Attention benefit.
- `TemperatureOnlyBaseline` performs poorly, showing that vibration is currently the dominant predictive modality.

## 5. Development optimization benchmark

Twelve candidates were screened and the three best trajectory-based candidates were confirmed across all six development folds.

| Candidate | Architecture | Mean trajectory NRMSE | Mean trajectory NMAE | NRMSE sample std | Global Window MAE (s) | Global Window RMSE (s) |
|---|---|---:|---:|---:|---:|---:|
| **OPT2_06** | **VibrationOnlyBaseline** | **0.28789** | **0.22602** | **0.09767** | **4,279.7** | **6,486.8** |
| OPT2_05 | VibrationOnlyBaseline | 0.31834 | 0.27172 | 0.14005 | 4,740.2 | 6,855.1 |
| OPT2_09 | CNNBiLSTMAttentionRUL | 0.34792 | 0.29597 | 0.10429 | 4,936.6 | 6,714.7 |

Selection result:

- `OPT2_06` is the accepted development winner.
- Primary selection metric: mean trajectory NRMSE.
- `OPT2_06` configuration: VibrationOnlyBaseline, learning rate `0.0005`, hidden dimension `64`, Huber loss with delta `1.0`.
- Its mean trajectory NRMSE of `0.28789` represents approximately 28.8% normalized trajectory error.
- The official test performance of `OPT2_06` has not been evaluated.

Development endpoint errors are diagnostic only because every development endpoint has an actual RUL of zero. They are not used as the primary optimization criterion.

## 6. Official test benchmark

The official benchmark contains 866 windows and eleven test bearing endpoints.

| Model | Window MAE (s) | Window RMSE (s) | Endpoint MAE (s) | Endpoint RMSE (s) | IEEE PHM score |
|---|---:|---:|---:|---:|---:|
| Random Forest | **6,558.1** | **7,645.7** | 3,502.9 | 4,378.0 | 0.11422 |
| CNNBiLSTMAttentionRUL, seed 42 | 8,450.9 | 10,251.3 | 3,751.8 | 5,093.6 | **0.13292** |
| CNNBiLSTMAttentionRUL, seed 123 | 8,090.6 | 9,772.5 | **2,972.5** | **3,529.0** | 0.09949 |
| CNNBiLSTMAttentionRUL, seed 999 | 8,447.1 | 10,227.8 | 3,349.9 | 4,018.0 | 0.08955 |
| CNNBiLSTMAttentionRUL, ensemble | 8,191.5 | 10,000.6 | 3,264.0 | 4,140.0 | 0.08465 |

Official findings:

- The ensemble improves Endpoint MAE and Endpoint RMSE over Random Forest.
- Random Forest remains better on Window MAE, Window RMSE, and IEEE PHM score.
- Seed 123 provides the best endpoint accuracy for the proposed model.
- Seed 42 provides the highest IEEE PHM score.
- The ensemble is not the best configuration for every metric.

## 7. Official endpoint tolerance

| Model | Within 10% | Within 20% |
|---|---:|---:|
| Random Forest | 1/11, 9.09% | 1/11, 9.09% |
| CNNBiLSTMAttentionRUL, seed 42 | 1/11, 9.09% | 2/11, 18.18% |
| CNNBiLSTMAttentionRUL, seed 123 | 0/11, 0.00% | 1/11, 9.09% |
| CNNBiLSTMAttentionRUL, seed 999 | 0/11, 0.00% | 0/11, 0.00% |
| CNNBiLSTMAttentionRUL, ensemble | 0/11, 0.00% | 1/11, 9.09% |

The current tolerance rates are low and do not support a claim of consistently accurate official endpoint prediction.

## 8. Missing-temperature benchmark

| Model | Temperature subgroup | Endpoint MAE (s) | Endpoint RMSE (s) | IEEE PHM score |
|---|---|---:|---:|---:|
| Random Forest | Usable temperature | 3,677.2 | 4,592.8 | 0.05001 |
| Random Forest | Fully missing temperature | 3,038.3 | 3,745.6 | 0.28544 |
| Proposed ensemble | Usable temperature | 3,364.7 | 4,315.6 | 0.08592 |
| Proposed ensemble | Fully missing temperature | 2,995.4 | 3,630.5 | 0.08126 |

The proposed architecture completes inference without NaN or infinite values when temperature is fully missing. This demonstrates operational robustness, but the current results do not demonstrate that temperature improves accuracy.

## 9. Benchmark against project targets

| Project target | Current evidence | Assessment |
|---|---|---|
| Complete end-to-end multimodal pipeline | Preprocessing, training, predictions, evaluation, and traceable reports are available | **Achieved** |
| Independent official test evaluation | Eleven official test bearings evaluated after configuration freezing | **Achieved** |
| Robustness to missing temperature | Stable inference on all fully missing-temperature bearings | **Partially Achieved** |
| Outperform handcrafted baseline | Better ensemble endpoint errors, but worse window errors and PHM score | **Partially Achieved** |
| Outperform single-modality Deep Learning | Vibration-only model outperforms the proposed model on development validation | **Not Achieved** |
| Demonstrate Attention benefit | No-Attention ablation performs better | **Not Achieved** |
| RMSE below 10% of lifetime | Official relative endpoint RMSE is approximately 20-23%; best development trajectory NRMSE is approximately 28.8% | **Not Achieved** |
| Compare CNN-BiLSTM with Transformer | Transformer result is not available | **Not Yet Evaluated** |

## 10. Final benchmark conclusion

- The implemented pipeline and evaluation protocol are complete for the current review stage.
- The proposed main model is `CNNBiLSTMAttentionRUL`.
- The strongest current development candidate is `OPT2_06`, a vibration-only model.
- The proposed multimodal model has not yet outperformed the vibration-only baseline.
- Attention has not shown a positive contribution in the current ablation.
- The RMSE target below 10% of lifetime has not been achieved.
- Transformer comparison and official evaluation of `OPT2_06` remain incomplete.

The current results should therefore be presented as a reproducible preliminary benchmark, not as final confirmation that every project performance target has been achieved.
