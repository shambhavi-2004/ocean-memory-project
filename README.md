# Ocean Memory Project — Machine Learning and Dynamical Modeling of Sea Surface Temperature Predictability

## 1. Introduction

The Ocean Memory Project investigates the question:  
**Can the physical state of the ocean today be used to predict its future state?**

Specifically, the project examines the **Sea Surface Temperature anomaly (SST_anom)** and explores the extent to which it exhibits “memory” — the persistence of past conditions influencing future variability.  
Using a sequence of data-driven and physics-informed models, this study seeks to:

1. Quantify the predictability of the next-month SST anomaly (`SST_next`).
2. Derive simple analytical relationships that govern this predictability.
3. Discover the underlying dynamical laws describing ocean memory.

The analysis uses regression, classification, symbolic regression, sparse dynamical modeling, and clustering to characterize both deterministic and stochastic components of ocean behavior.

---

## 2. Data Description

**Source:** Copernicus Marine Environment Monitoring Service (CMEMS)  
**Region:** Indian Ocean (60°E–100°E, 0°N–30°N)  
**Time period:** 1993–2024  
**Temporal resolution:** Monthly

After preprocessing and feature extraction from NetCDF files, the dataset included the following key physical variables:

| Variable | Description | Units |
|-----------|-------------|--------|
| `SST_anom` | Sea Surface Temperature anomaly | °C |
| `SSH_anom` | Sea Surface Height anomaly | meters |
| `MLD_anom` | Mixed Layer Depth anomaly | meters |
| `Current_speed` | Surface current speed | m/s |
| `Salinity_anom` | Sea surface salinity anomaly | PSU |
| `SST_next` | Next-month SST anomaly (target) | °C |

Preprocessing steps included missing-value removal, normalization, and time-based train/test splitting (1993–2010 for training; 2011–2024 for testing).

---

## 3. Methodology

The modeling framework was organized hierarchically, beginning with purely statistical approaches and progressing toward interpretable physical models.  
Each stage informed the next by revealing limitations or insights that guided methodological refinement.

### 3.1 Regression Analysis

Regression models were first used to evaluate the predictability of `SST_next` based on the current ocean state.

| Model | R² | RMSE (°C) | Interpretation |
|--------|----|------------|----------------|
| Linear Regression | 0.85 | 0.325 | Strong linear dependence |
| Random Forest Regressor | 0.76 | 0.42 | Moderate non-linear effects |
| Gradient Boosting Regressor | 0.73 | 0.46 | Captures smooth trends |
| Support Vector Regressor (RBF) | 0.70 | 0.49 | Non-linear response surfaces |

**Interpretation:**  
The linear model performed best, showing that the SST anomaly evolution is largely governed by **linear persistence**.  
This indicates that the ocean’s surface temperature retains strong memory on monthly scales.

---

### 3.2 Classification: Warm vs. Cold States

To test regime separability, the continuous target variable `SST_next` was converted into categorical states:
- **Warm:** SST_next > median
- **Cold:** SST_next ≤ median

| Classifier | Accuracy |
|-------------|-----------|
| Logistic Regression | 0.86 |
| Random Forest Classifier | 0.91 |
| Support Vector Machine | 0.88 |

**Interpretation:**  
Distinct warm and cold regimes exist and are **linearly separable** based on the physical state of the ocean.  
The high accuracy of tree-based classifiers confirms that certain patterns of temperature, height, and current speed consistently correspond to identifiable ocean states.

---

### 3.3 Symbolic Regression (PySR)

To discover a physically interpretable relationship, **symbolic regression** was applied using the PySR package.  
This method evolves mathematical expressions that best fit the data.

**Discovered equation:**
\[
SST_{next} = 0.74 \times SST_{anom} - 0.29 \times Current_{speed} + 27.59
\]

| Metric | Value |
|---------|--------|
| R² | 0.878 |
| RMSE | 0.308 °C |

**Interpretation:**  
This simple equation captures 87.8% of the variance in the data.  
It reveals that:
- SST anomalies persist with 74% of their previous magnitude (strong memory).
- Higher surface current speeds cause cooling through vertical mixing.
- The baseline equilibrium temperature is approximately 27.6°C.

This result demonstrates that the system is predominantly linear and deterministic, governed by a balance between persistence and mixing.

---

### 3.4 Dynamical Modeling (SINDy)

To uncover the differential law governing the *rate of change* of SST anomalies, the **Sparse Identification of Nonlinear Dynamics (SINDy)** framework was used.

**Identified dynamic equation:**
\[
\frac{d(SST_{anom})}{dt} = -0.108 \times SST_{anom} + 0.110 \times SSH_{anom} - 0.088 \times Current_{speed}
\]

**Interpretation of coefficients:**
- `-0.108 × SST_anom`: damping term representing the ocean’s relaxation toward equilibrium.  
- `+0.110 × SSH_anom`: positive feedback from heat storage in raised sea levels.  
- `-0.088 × Current_speed`: cooling effect from turbulent mixing.

**Memory timescale:**
\[
\tau = \frac{1}{0.108} \approx 9.26 \text{ months}
\]
This indicates that the ocean retains temperature memory for approximately nine months.

---

### 3.5 Clustering and Dimensionality Reduction

Unsupervised learning was used to identify natural groupings of ocean states using **K-Means clustering** and **Principal Component Analysis (PCA)**.

| Cluster | SST_anom | SSH_anom | Current_speed | Physical Interpretation |
|----------|-----------|-----------|----------------|--------------------------|
| 0 | High | High | Low | Stable warm conditions |
| 1 | Moderate | Moderate | Moderate | Transitional states |
| 2 | Low | Low | High | Cool, mixed regimes |

PCA reduced the data to two principal components, explaining over 80% of total variance.  
The dominant contributions were from `SST_anom` and `SSH_anom`, confirming that ocean heat content and surface elevation jointly define the main variability patterns.

---

### 3.6 Hybrid SINDy + GPR Ensemble Simulation

A hybrid modeling approach combined:
1. **SINDy** – providing a deterministic backbone equation.
2. **Gaussian Process Regression (GPR)** – capturing stochastic residuals representing atmospheric forcing.

**Findings:**
- The SINDy-only model explained ~50% of the rate-of-change variance.
- The hybrid model improved short-term predictive power to ~78%.
- However, in long-term integrations, the model diverged rapidly beyond the training period.

**Interpretation:**  
The divergence revealed a fundamental limit of predictability — the unmodeled atmospheric forcing behaves as **true stochastic noise**, which cannot be deterministically predicted from ocean-only variables.

This result is scientifically significant: it experimentally demonstrates that the **ocean memory system is damped and stochastic**, driven by both internal persistence and external unpredictable forcing.

---

## 4. Interactive Dashboard

A comprehensive **Streamlit dashboard (`app.py`)** was developed to visualize and interact with all experiments.

**Dashboard Sections:**
1. **Overview:** Dataset preview and feature correlations.  
2. **Regression Models:** Comparison of model performances and predictions.  
3. **Classification Models:** Warm/cold state classification and confidence visualization.  
4. **Symbolic Regression:** Interactive prediction using derived analytical formula.  
5. **SINDy Dynamics:** Visualization of ocean memory decay law.  
6. **Clustering & PCA:** Visualization of unsupervised regimes and principal components.

Each section allows the user to input physical values to make real-time predictions and explore model behavior interactively.

---

## 5. Results and Conclusions

- The **ocean exhibits strong short-term predictability** of SST anomalies due to its physical persistence.  
- The **memory timescale** was quantified at approximately **nine months** using SINDy.  
- Symbolic regression confirmed that a **simple linear model** captures the dominant processes.  
- Ensemble simulation revealed the **fundamental unpredictability** introduced by stochastic atmospheric forcing.  
- Clustering demonstrated that ocean states can be categorized into **distinct thermal regimes**, aligning with known oceanographic patterns.

---

## 6. Technologies and Tools

| Category | Tools and Libraries |
|-----------|--------------------|
| Data Handling | pandas, numpy, xarray |
| Machine Learning | scikit-learn, PySR, gplearn |
| Dynamical Modeling | pysindy |
| Visualization | matplotlib, seaborn, plotly |
| Dashboard Development | Streamlit |
| Clustering and Dimensionality Reduction | KMeans, PCA |

---

## 7. Key Contributions

1. Demonstrated that ocean surface temperature has a measurable and quantifiable memory on monthly to seasonal scales.  
2. Identified the simplest governing law for SST dynamics using symbolic regression and sparse modeling.  
3. Quantified the limit of predictability and established the role of stochastic atmospheric forcing.  
4. Developed an integrated, interactive dashboard for model visualization and user exploration.

---

## 8. Future Work

- Integrate **atmospheric reanalysis variables** (wind stress, air temperature, surface fluxes) to model external forcing.  
- Extend the hybrid SINDy–GPR framework to **spatio-temporal grids** for regional predictability studies.  
- Implement **ensemble forecasting and uncertainty quantification** in the dashboard.  
- Explore **transfer learning** to apply the trained models to other ocean basins.

---

## 9. Author and Acknowledgements

**Author:** Shambhavi Patil  
**GitHub:** [@shambhavi-2004](https://github.com/shambhavi-2004)

**Acknowledgements:**  
This work uses data from the Copernicus Marine Environment Monitoring Service (CMEMS).  
It draws conceptual inspiration from:
- Brunton, Proctor & Kutz (2016): *Discovering Governing Equations with Sparse Identification of Nonlinear Dynamics (SINDy)*  
- Cranmer et al. (2020): *PySR: Fast Symbolic Regression in Python*  

---

## 10. Summary

The Ocean Memory Project demonstrates that while the ocean surface exhibits **deterministic short-term memory**, its long-term evolution is shaped by **stochastic atmospheric forcing** that limits precise prediction.  
By combining interpretable machine learning and dynamical modeling, this work bridges the gap between **data-driven learning** and **physical understanding**, providing a framework for studying memory processes in Earth’s climate systems.
