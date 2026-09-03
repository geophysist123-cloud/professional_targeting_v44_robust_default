# Professional Geophysical Targeting Dashboard

A Streamlit-based decision-support dashboard for integrating magnetic and gravity geophysical evidence into ranked exploration targets.

## Overview

This application combines magnetic and gravity datasets, normalizes their anomaly strength, applies configurable evidence weights, accounts for data availability through Data Confidence, evaluates magnetic–gravity agreement through Concordance, and produces a final Target Score and priority classification.

The application is designed for **exploration screening and decision support**. It does not replace geological interpretation, quality control, or detailed geophysical modeling.

## Main Workflow

```text
Magnetic / Gravity Data
          |
          v
     Normalization
          |
          v
   Normalized Scores
          |
          v
       Weighting
          |
          v
   Data Confidence
          |
          v
     Contribution
          |
          +------------------+
          |                  |
          v                  v
   Core Evidence       Concordance
          |                  |
          +--------+---------+
                   |
                   v
        85% Evidence + 15%
          Concordance
                   |
                   v
            Target Score
                   |
                   v
          Target Priority
```

## Normalization

The default and recommended normalization method is:

**Robust Z-score — Recommended**

Robust Z-score uses the median and Median Absolute Deviation (MAD), making the normalization less sensitive to extreme outliers than a conventional mean/standard-deviation Z-score.

The application also supports:

- Percentile
- Winsorized Min-Max

Robust Z-score is selected by default in the application.

## Magnetic and Gravity Weighting

The default evidence weights are:

- **Magnetic: 60%**
- **Gravity: 40%**

The Magnetic weight can be changed in the application. Gravity is automatically calculated as:

```text
Gravity Weight = 100% - Magnetic Weight
```

The weights control how strongly each available normalized dataset contributes to the evidence score.

## Data Confidence

Data Confidence explicitly reflects which geophysical datasets are available for a target.

| Magnetic | Gravity | Data Confidence |
|---|---|---:|
| Available | Available | 100% |
| Available | Missing | 60% |
| Missing | Available | 40% |
| Missing | Missing | No Score |

This prevents a target supported by only one dataset from being treated as equivalent to a target supported by both datasets.

When both datasets are missing, the target receives **No Score** rather than an artificial zero-confidence score.

## Contribution

For each available dataset, the contribution is based on:

```text
Normalized Score × Dataset Weight × Confidence Factor
```

For example, with:

```text
Magnetic Score = 90
Magnetic Weight = 60%
Confidence = 100%
```

the Magnetic contribution is:

```text
90 × 0.60 × 1.00 = 54
```

Gravity is calculated using the same principle.

The individual Magnetic and Gravity contributions make the final target ranking more transparent and auditable.

## Concordance

Concordance measures how closely the normalized Magnetic and Gravity scores agree.

When both datasets are available:

```text
Concordance = 100 - |Magnetic Score - Gravity Score|
```

The result is constrained to the 0–100 range.

Example:

```text
Magnetic Score = 90
Gravity Score  = 85

Concordance = 100 - |90 - 85|
            = 95
```

A high Concordance indicates that both geophysical signals produce similarly strong normalized evidence.

If only one dataset is available, Concordance is reported as **No Score**, because agreement between two independent signals cannot be evaluated from a single signal.

## Final Target Score

When both Magnetic and Gravity are available, the final score combines:

- **85% Evidence**
- **15% Concordance**

Conceptually:

```text
Target Score =
    0.85 × Core Evidence
  + 0.15 × Concordance
```

This keeps the actual geophysical evidence as the dominant component while allowing multi-physics agreement to strengthen or weaken the final ranking.

For targets with only one available dataset, the score is based on the confidence-adjusted available evidence rather than inventing a Concordance value.

## Priority Classification

The dashboard classifies targets using the Target Score:

| Target Score | Priority |
|---:|---|
| ≥ 80 | VERY HIGH |
| ≥ 65 | HIGH |
| ≥ 50 | MEDIUM |
| < 50 | LOW |

## Inputs

The application supports magnetic and gravity data and can work with common tabular/geospatial formats supported by the application.

Typical fields include:

- X / longitude
- Y / latitude
- Magnetic anomaly
- Gravity anomaly

The application also provides data validation and quality-control information.

## Outputs

The dashboard provides exploration-targeting information including:

- Target Score
- Target Priority
- Magnetic normalized score
- Gravity normalized score
- Magnetic contribution
- Gravity contribution
- Data Confidence
- Concordance
- Target coordinates
- Supporting data-quality information
- Interactive map visualization

## Installation

Use Python 3.11 or 3.12 for the most predictable local environment.

Create a virtual environment:

```bash
python -m venv .venv
```

Activate it on Windows:

```bash
.venv\Scripts\activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Run the application:

```bash
streamlit run app.py
```

If the Python file has a different name, replace `app.py` with that filename.

## Streamlit Cloud Deployment

Recommended repository structure:

```text
professional_targeting_v4_robust_default/
│
├── app.py
├── requirements.txt
├── README.md
└── other project files
```

In Streamlit Cloud configure:

```text
Repository: your GitHub repository
Branch: main
Main file path: app.py
```

Then deploy/reboot the application.

The `requirements.txt` file must be located in the repository root so that Streamlit Cloud can install the required packages.

## Required Packages

The project dependencies are listed in `requirements.txt`, including:

- Streamlit
- Pandas
- OpenPyXL
- Folium
- Streamlit-Folium
- PyProj
- GeoPandas
- ReportLab

## Important Notes for Streamlit Cloud

If the application reports:

```text
ModuleNotFoundError: No module named 'folium'
```

check that:

1. `folium` appears in `requirements.txt`.
2. `requirements.txt` is in the repository root.
3. The deployed Streamlit application points to the correct branch.
4. The selected Main file is the correct Python file.
5. A new deployment/reboot is performed after committing changes.

The dependency installation log should show Folium and the other required packages during deployment.

## Development Container

A `.devcontainer/devcontainer.json` file may be added if a reproducible VS Code development environment is desired.

The development container is optional and is not required for Streamlit Cloud deployment.

## Project Purpose

The dashboard is intended to provide a transparent and reproducible framework for:

- integrating magnetic and gravity evidence,
- ranking exploration targets,
- identifying high-confidence multi-physics targets,
- exposing missing-data limitations,
- supporting early-stage exploration decisions.

The resulting targets should be reviewed alongside geology, structure, petrophysical information, acquisition quality, processing parameters, and other exploration evidence before operational decisions are made.

## License

Add the project's chosen license here if the repository will be distributed publicly.
