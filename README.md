# Wildfire Ignition Modelling and Spatial Bias Analysis Using a WUI Pressure Index

This repository contains the processed geospatial datasets, modelling workflow, and figure outputs associated with a study of wildfire ignition prediction and spatial model bias across the Tamil Nadu–Karnataka region of India.

The study proposes a **Wildland–Urban Interface Pressure Index (WUI-PI)** to represent the co-occurrence of anthropogenic pressure and combustible vegetation. Machine-learning models are evaluated using conventional predictive metrics and geographically independent spatial validation to examine model generalization and the spatial distribution of prediction errors.

## Overview

Wildfire ignition is influenced by climatic, environmental, and anthropogenic factors. Conventional wildfire prediction models commonly represent these variables as independent predictors and are frequently evaluated using global performance metrics.

This study investigates two related problems:

1. How interactions between human pressure and fuel availability can be represented within wildfire ignition modelling.
2. Whether strong global predictive performance is retained when models are evaluated across geographically separated regions.

The proposed WUI-PI combines anthropogenic and fuel-related components using a multiplicative interaction. The modelling framework further evaluates spatial generalization through a west-to-east geographic hold-out and maps model overprediction and underprediction regions.

## Study Area

The analysis covers the **Tamil Nadu–Karnataka region in southern India**.

All predictor layers were processed to a common spatial grid at approximately **500 m spatial resolution**. Pixels containing missing values in any predictor layer were excluded before model development.

## Repository Structure

```text
.
├── figures/
├── GEE Scripts.txt
├── LST_TN_KA.tif
├── NDVI_TN_KA.tif
├── VPD_TN_KA_FIXED.tif
├── aspect_TN_KA.tif
├── builtup_density_TN_KA.tif
├── fire_labels2.tif
├── forest.tif
├── fragmentation.tif
├── ntl2.tif
├── road_density.tif
├── wildfire_ignition_wui_pi_analysis.ipynb
├── LICENSE
└── README.md
```

## Data Layers

| File | Description |
|---|---|
| `LST_TN_KA.tif` | Land Surface Temperature |
| `NDVI_TN_KA.tif` | Normalized Difference Vegetation Index |
| `VPD_TN_KA_FIXED.tif` | Vapor Pressure Deficit |
| `aspect_TN_KA.tif` | Terrain aspect |
| `builtup_density_TN_KA.tif` | Built-up density |
| `fire_labels2.tif` | Binary wildfire occurrence labels |
| `forest.tif` | Forest-related raster layer |
| `fragmentation.tif` | Landscape fragmentation |
| `ntl2.tif` | Nighttime light intensity |
| `road_density.tif` | Road density |

Selected remote-sensing variables were prepared using Google Earth Engine. The corresponding processing scripts are provided in `GEE Scripts.txt`.

## WUI Pressure Index

The proposed **Wildland–Urban Interface Pressure Index (WUI-PI)** represents the interaction between anthropogenic pressure and fuel availability.

### Anthropogenic Component

Human pressure is represented using:

- Nighttime light intensity
- Road density
- Built-up density

These variables are normalized before being combined into the anthropogenic pressure component.

### Fuel Component

Fuel-related conditions are represented using vegetation and landscape characteristics, including NDVI-derived information and fragmentation-related variables.

### WUI-PI Formulation

The anthropogenic and fuel components are combined using a multiplicative interaction:

```text
WUI-PI = Human Pressure × Fuel Pressure
```

The multiplicative formulation emphasizes locations where anthropogenic pressure and combustible vegetation occur simultaneously. Consequently, a high value in only one component is insufficient to produce a high WUI-PI value.

## Methodological Workflow

The analysis follows the workflow below:

1. Raster loading and alignment
2. Removal of pixels containing missing predictor values
3. Generation of fire and non-fire samples
4. Normalization of selected predictor variables
5. Construction of anthropogenic pressure
6. Construction of fuel pressure
7. Calculation of WUI-PI
8. Feature engineering
9. Machine-learning model training
10. Global model evaluation
11. Geographic spatial validation
12. Repeated validation
13. Spatial bias diagnosis
14. Feature sensitivity analysis

## Machine-Learning Models

Three classification models are evaluated:

- Random Forest
- XGBoost
- Logistic Regression

Random Forest is used as the principal model for spatial validation, spatial bias diagnosis, and feature sensitivity analysis.

## Model Evaluation

Model performance is evaluated using multiple complementary measures:

- Receiver Operating Characteristic Area Under the Curve (ROC-AUC)
- Precision–Recall AUC (PR-AUC)
- Precision
- Recall
- F1-score
- Confusion matrix
- Top-10% fire occurrence rate
- Spatial AUC

The use of multiple metrics is particularly important because wildfire pixels represent a small proportion of the complete raster dataset.

## Spatial Validation

Geographic transferability is evaluated using a fixed spatial hold-out strategy.

The study region is divided geographically into western and eastern portions:

```text
Western region → Training
Eastern region → Testing
```

The model is trained using pixels from the western portion of the study area and evaluated using geographically separated pixels from the eastern portion.

This procedure is used to assess model performance in a spatially independent region rather than relying exclusively on randomly partitioned samples.

## Repeated Validation

Repeated validation is used to assess the sensitivity of model performance to random non-fire background sampling.

Ten independent random seeds are used:

```text
42, 52, 62, 72, 82, 92, 102, 112, 122, 132
```

For each repetition:

1. Non-fire pixels are randomly resampled.
2. A 10:1 non-fire-to-fire sampling ratio is applied.
3. The feature configuration is retained.
4. The Random Forest configuration is held constant.
5. Global model performance is calculated.
6. The fixed west-to-east spatial validation procedure is applied.
7. Spatial performance is recorded.

The following metrics are collected for each repetition:

- Global ROC-AUC
- PR-AUC
- Operating threshold
- Top-10% threshold
- Top-10% fire occurrence rate
- Spatial AUC
- Spatial Top-10% fire occurrence rate

Mean values and standard deviations are subsequently calculated across the ten experiments.

The complete repeated-validation implementation is included at the end of `wildfire_ignition_wui_pi_analysis.ipynb`.

## Spatial Bias Diagnosis

Model prediction errors are examined spatially using overprediction and underprediction maps.

### Overprediction

Overprediction regions represent locations where the model assigns elevated wildfire probability despite the absence of an observed wildfire label.

### Underprediction

Underprediction regions represent observed wildfire locations that receive insufficient predicted wildfire probability.

These error classes are mapped to examine whether prediction errors exhibit geographic structure across the study region.

The resulting maps are interpreted as spatial patterns of model error and are not treated as direct evidence of causal relationships between individual environmental variables and wildfire ignition.

## Feature Sensitivity Analysis

Two complementary feature importance approaches are used.

### Impurity-Based Feature Importance

Random Forest impurity-based importance measures the contribution of individual predictor variables to reductions in node impurity during model training.

### Permutation Feature Importance

Permutation importance measures the reduction in model performance after randomly permuting an individual predictor variable.

The two approaches are compared to evaluate the consistency of influential predictors across complementary importance estimation methods.

## Figures

The `figures/` directory contains the principal visual outputs generated for the study, including:

- Study area and wildfire ignition distribution
- WUI-PI conceptual framework
- Feature engineering workflow
- Methodological workflow
- Spatial distribution of WUI-PI
- Confusion matrix
- ROC curves
- Global and spatial AUC comparison
- Overprediction and underprediction maps
- Random Forest feature importance
- Permutation feature importance
- Comparison of feature importance methods

## Reproducing the Analysis

### 1. Clone the Repository

```bash
git clone https://github.com/RelativelyBurberry/wildfire-wuipi
cd wildfire-wuipi
```

### 2. Install Required Python Packages

The analysis uses the following principal Python libraries:

```bash
pip install numpy pandas matplotlib rasterio scikit-learn xgboost
```

### 3. Open the Analysis Notebook

```bash
jupyter notebook wildfire_ignition_wui_pi_analysis.ipynb
```

### 4. Execute the Notebook

Run the notebook cells sequentially to:

1. Load the raster datasets.
2. Align and mask valid pixels.
3. Construct the modelling dataset.
4. Generate fire and non-fire samples.
5. Construct the WUI-PI.
6. Generate engineered predictor variables.
7. Train the machine-learning models.
8. Evaluate global model performance.
9. Perform geographic spatial validation.
10. Run repeated validation.
11. Generate spatial bias maps.
12. Calculate feature importance.

Raster files should remain in the repository root unless the file paths in the notebook are modified.

## Reproducibility Notes

The repository contains the processed raster layers and modelling workflow required to reproduce the principal analyses presented in the study.

Repeated validation is reproduced directly from the main analysis notebook using the ten specified random seeds. The geographic west-to-east partition remains fixed across repeated experiments.

Minor numerical differences may occur because of differences in software versions or computational environments.

## Supplementary Material

This repository serves as the computational supplementary material for the study. It provides:

- Processed raster datasets
- Google Earth Engine processing scripts
- Data preprocessing workflow
- Feature engineering implementation
- WUI-PI construction
- Machine-learning model training workflow
- Global and spatial evaluation procedures
- Repeated validation implementation
- Spatial bias analysis
- Feature sensitivity analysis
- Figure outputs

## Citation

If you use this repository or the proposed WUI-PI framework, please cite the associated publication.

The complete citation and DOI will be added following final publication metadata assignment.

## License

This repository is distributed under the terms specified in the `LICENSE` file.
