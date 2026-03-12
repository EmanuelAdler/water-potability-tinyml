# Water Potability TinyML

> Random Forest classifier trained in Python, deployed to an ESP32-S3 with **emlearn** — no cloud, no OS, inference at the edge.

---

## Description and Functionality

This program implements a complete TinyML pipeline for water quality classification: a Jupyter notebook handles data preprocessing, class balancing, selection of six physicochemical parameters obtainable from electronic sensors, and training of a Random Forest classifier tuned with Optuna; the **emlearn** library then converts the trained model into a C header file (`rfc_model.h`) that is compiled directly into the firmware of an **ESP32-S3** microcontroller, which runs inference entirely on-device — with no operating system, no connectivity, and no external dependencies — and reports performance metrics (accuracy, precision, recall, and F1-score) via the serial monitor.

---

## Keywords

`TinyML` · `Machine Learning` · `Random Forest` · `ESP32-S3` · `Arduino` · `emlearn` · `Water Quality` · `Edge AI` · `Embedded Systems` · `Classification` · `Scikit-learn` · `Optuna`

---

## Overview

This project implements a full TinyML pipeline for water quality analysis:

1. A Jupyter notebook trains a Random Forest on the [Water Potability dataset](https://www.kaggle.com/datasets/adityakadiwal/water-potability) and exports the model as a C header using **emlearn**.
2. An Arduino sketch embeds the model and 716 test samples directly in firmware, runs batch inference on the ESP32-S3, and prints classification metrics to the serial monitor.

No external sensors are needed to run the evaluation — the test data is compiled into the binary.

---

## How It Works

```
water_potability.csv
        │
        ▼
Water_Potability_RF.ipynb
  ├─ EDA & NaN imputation
  ├─ Ivanov class balancing
  ├─ Feature selection (6 features)
  ├─ StandardScaler + train/test split
  ├─ Optuna hyperparameter search
  ├─ RandomForestClassifier (sklearn)
  ├─ emlearn.convert() ──────────────► rfc_model.h
  └─ Export test arrays ─────────────► v3_rfc_water_emlearn.ino
                                               │
                                               ▼
                                    ESP32-S3 (Arduino)
                                      ├─ 716 samples in firmware
                                      ├─ rfc_model_predict()
                                      └─ Serial: metrics report
```

---

## Project Structure

| File | Description |
|---|---|
| `Water_Potability_RF.ipynb` | Full ML pipeline: EDA, balancing, Optuna tuning, emlearn export |
| `v3_rfc_water_emlearn.ino` | Arduino sketch: loads model + test data, runs inference, prints metrics |
| `rfc_model.h` | Auto-generated C model (emlearn inline format) |
| `eml_trees.h` | emlearn decision tree runtime |
| `eml_common.h` | emlearn common definitions |
| `eml_log.h` | emlearn logging utilities |
| `random_forest.joblib` | Saved sklearn model (for Python reuse) |
| `test_df.pkl` | Scaled test features (716 samples) |
| `test_expect.pkl` | Expected labels for test set |
| `wokwi.toml` | Wokwi simulation configuration |
| `diagram.json` | Wokwi circuit diagram (ESP32-S3 + serial monitor) |

---

## Water Quality Features

The final model uses **6 of the 9 original features** (selected to simulate parameters that can be obtained with electronic sensors):

| Feature | Description |
|---|---|
| `ph` | Acidity/alkalinity of water (0–14) |
| `Hardness` | Calcium and magnesium concentration (mg/L) |
| `Solids` | Total dissolved solids (ppm) |
| `Chloramines` | Chloramines concentration (ppm) |
| `Conductivity` | Electrical conductivity (μS/cm) |
| `Turbidity` | Light-scattering measure of water clarity (NTU) |

The label is `Potability` — `1` = potable, `0` = not potable.

> Features with missing values (`ph`, `Sulfate`, `Trihalomethanes`) were imputed with their column mean before training.

---

## Model Details

| Parameter | Value |
|---|---|
| Algorithm | Random Forest Classifier |
| `n_estimators` | 130 |
| `max_depth` | 10 |
| `max_features` | `sqrt` |
| `min_samples_split` | 4 |
| `min_samples_leaf` | 4 |
| `criterion` | `log_loss` |
| `class_weight` | `balanced` |
| Hyperparameter search | Optuna (30 trials, Pareto front: F1 vs model size) |
| Class balancing | Ivanov undersampling |
| Preprocessing | `StandardScaler` |
| Embedded via | `emlearn` (`method='inline'`) |

---

## Requirements

### Python (notebook)

```
pandas
numpy
matplotlib
seaborn
scikit-learn
optuna
emlearn
pycm
joblib
```

Install with:

```bash
pip install pandas numpy matplotlib seaborn scikit-learn optuna emlearn pycm joblib
```

You also need the dataset file `water_potability.csv` in the same directory as the notebook (available on [Kaggle](https://www.kaggle.com/datasets/adityakadiwal/water-potability)).

### Arduino / ESP32

- [Arduino IDE 2.x](https://www.arduino.cc/en/software) or `arduino-cli`
- ESP32 board package: **Espressif Systems — esp32** (install via Board Manager)
- Target board: **ESP32-S3 Dev Module** (or compatible)
- No additional Arduino libraries required — emlearn runtime headers are included in the repo

### Wokwi (optional simulation)

- [Wokwi CLI](https://docs.wokwi.com/wokwi-ci/getting-started)
- Pre-built binaries (`*.bin`, `*.elf`) are included for immediate simulation

---

## Getting Started

### 1. Clone the repository

```bash
git clone <repo-url>
cd water-potability-tinyml
```

### 2. Train the model and regenerate the C header

Open `Water_Potability_RF.ipynb` in Jupyter (or Google Colab) and run all cells. This will:

- Train the Random Forest with the tuned hyperparameters
- Generate `rfc_model.h` via `emlearn.convert()`
- Print the C arrays for the test data (paste into the `.ino` file if updating)
- Save `random_forest.joblib`, `test_df.pkl`, and `test_expect.pkl`

> **Note:** If you only want to flash and run the existing model, skip this step — all generated files are already committed.

### 3. Flash to ESP32-S3

#### Arduino IDE

1. Open `v3_rfc_water_emlearn.ino`
2. Select **Tools → Board → ESP32-S3 Dev Module**
3. Select the correct port under **Tools → Port**
4. Click **Upload**
5. Open **Tools → Serial Monitor** at baud rate **115200**

#### arduino-cli

```bash
arduino-cli compile --fqbn esp32:esp32:esp32s3 .
arduino-cli upload  --fqbn esp32:esp32:esp32s3 --port /dev/ttyUSB0 .
```

### 4. (Optional) Simulate with Wokwi

```bash
wokwi-cli simulate --timeout 60
```

The serial output will appear in the terminal.

---

## Expected Output

After flashing (or simulating), the serial monitor will print:

```
Begin
Total samples: 716
Time to run all inferences (ms): <value>
Total correct predictions: <value>
True Postives: <value>
True Negatives: <value>
False Positives: <value>
False Negatives: <value>
Accuracy: <value>
Precision: <value>
Recall: <value>
F1-Score: <value>
Informedness: <value>
Markedness: <value>
```

---

## Performance Metrics

Metrics computed on the embedded 716-sample test set (balanced, stratified split):

| Metric | Value |
|---|---|
| Accuracy | ~0.60 |
| Precision | ~0.61 |
| Recall | ~0.58 |
| F1-Score | ~0.59 |

These match the Python evaluation in the notebook (`accuracy_score`, `precision_score`, `recall_score`, `f1_score` from scikit-learn), confirming that the emlearn conversion preserves model behaviour exactly.

> The ~60% accuracy reflects the inherent difficulty of the water potability dataset (class imbalance, missing values, overlapping feature distributions). The model was optimized for F1-score and memory footprint rather than raw accuracy.

---

## Publication

This model was developed and used in the following peer-reviewed article:

> Emanuel Adler Medeiros Pereira, Jeferson Fernando da Silva Santos, Erick de Andrade Barboza,
> **An energy efficient TinyML model for a water potability classification problem**,
> *Sustainable Computing: Informatics and Systems*, Volume 43, 2024, 101010, ISSN 2210-5379.
> https://doi.org/10.1016/j.suscom.2024.101010

**Abstract:** Safe drinking water is an essential resource and a fundamental human right, but its access continues beyond billions of people, posing numerous health risks. A key obstacle in monitoring water quality is managing and analyzing extensive data. Machine learning models have become increasingly prevalent in water quality monitoring, aiding decision makers and safeguarding public health. An integrated system, which combines electronic sensors with a Machine Learning model, offers immediate feedback and can be implemented in any location. This type of system operates independently of an Internet connection and does not depend on data derived from chemical or laboratory analysis. The aim of this study is to develop an energy-efficient TinyML model to classify water potability that operates as an embedded system and relies solely on the data available through electronic sensing. When compared with a similar model functioning in the Cloud, the proposed model requires 51.2% less memory space, performs all inference tests approximately 99.95% faster, and consumes about 99.95% less energy. This increase in performance enables the classification model to run for years in devices that are very resource-constrained.

---

This model was also used in the following conference paper:

> Emanuel Pereira, Jeferson Santos, Erick Barboza,
> **Comparing TinyML Models and Libraries for On-Device Water Potability Classification**,
> *2024 XIV Brazilian Symposium on Computing Systems Engineering (SBESC)*, 2024, pp. 1–6, ISSN 2324-7894.
> https://doi.org/10.1109/SBESC65055.2024.10771818

**Abstract:** Water pollution, mainly caused by human activities that elevate harmful substance concentrations above ideal levels, threatens both the supply and quality of drinking water and also adversely affects economic development and environmental sustainability. Machine learning, combined with TinyML and the Internet of Things, is being used to predict drinking water classification, providing a promising alternative to traditional water sample monitoring. This study aims to compare various machine learning models and TinyML libraries to solve the classification problem of water potability. The Random Forest algorithm showed the best performance in Accuracy, Precision, Recall, and F1-Score, with Emlearn and Micromlgen libraries achieving the fastest inference time of 362 milliseconds. The multilayer perceptron model with the EmbML library used the least memory, with 283,113 bytes, and the Random Forest model with Micromlgen had the lowest energy consumption, using only 104.534 millijoules. This work can help researchers and professionals implement water potability classification systems and use TinyML in other classification problems as well.
