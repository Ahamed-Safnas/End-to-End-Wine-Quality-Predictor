# 🍷 Wine Quality Predictor — End-to-End MLOps Pipeline

> End-to-end MLOps pipeline for wine quality prediction | ElasticNet regression on 11 physicochemical features | Modular pipeline stages (ingest, validate, transform, train, evaluate) | MLflow experiment tracking | Flask web UI | CI/CD via GitHub Actions + Docker + AWS ECR + EC2

---

## Workflows

1. Update config.yaml
2. Update schema.yaml
3. Update params.yaml
4. Update the entity
5. Update the configuration manager in src config
6. Update the components
7. Update the pipeline 
8. Update the main.py
9. Update the app.py

---

## 📌 Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Pipeline Stages](#pipeline-stages)
- [Experiment Tracking](#experiment-tracking)
- [Flask Web App](#flask-web-app)
- [CI/CD Deployment](#cicd-deployment)

---

## Overview

This project predicts the quality score of red wine based on 11 physicochemical input features. It is built as a **modular, production-style ML pipeline** with schema validation, centralized configuration, MLflow experiment tracking, and an automated CI/CD deployment pipeline to AWS.

**Input Features:**

| Feature | Type |
|---|---|
| Fixed Acidity | float64 |
| Volatile Acidity | float64 |
| Citric Acid | float64 |
| Residual Sugar | float64 |
| Chlorides | float64 |
| Free Sulfur Dioxide | float64 |
| Total Sulfur Dioxide | float64 |
| Density | float64 |
| pH | float64 |
| Sulphates | float64 |
| Alcohol | float64 |

**Target:** `quality` (integer score)

---

## Architecture

```
Remote Dataset (ZIP)
        |
        v
+------------------+
|  Data Ingestion   |  <- Download, unzip, store in artifacts/
+--------+---------+
         |
         v
+--------------------+
|  Data Validation    |  <- Schema check against schema.yaml
+--------+-----------+
         |
         v
+------------------------+
|  Data Transformation    |  <- Train/test split
+--------+---------------+
         |
         v
+------------------+
|  Model Trainer    |  <- ElasticNet (alpha=0.2, l1_ratio=0.1)
+--------+---------+
         |
         v
+--------------------+
|  Model Evaluation   |  <- RMSE, MAE, R2 logged to MLflow
+--------+-----------+
         |
         v
+------------------+
|  Flask Web App    |  <- /predict endpoint with HTML UI
+------------------+
```

---

## Tech Stack

| Category | Tools |
|---|---|
| **ML / Modeling** | scikit-learn, ElasticNet |
| **Experiment Tracking** | MLflow |
| **Web Framework** | Flask |
| **Containerization** | Docker |
| **Cloud Infrastructure** | AWS EC2, AWS ECR |
| **CI/CD** | GitHub Actions |
| **Config Management** | config.yaml, schema.yaml, params.yaml |
| **Environment** | Python 3.8, Conda |

---

## Project Structure

```
wine-quality-predictor-mlops/
|
+-- src/mlProject/
|   +-- pipeline/
|   |   +-- stage_01_data_ingestion.py
|   |   +-- stage_02_data_validation.py
|   |   +-- stage_03_data_transformation.py
|   |   +-- stage_04_model_trainer.py
|   |   +-- stage_05_model_evaluation.py
|   |   +-- prediction.py
|   +-- components/         # Core logic for each stage
|   +-- config/             # Configuration manager
|   +-- entity/             # Dataclasses for config entities
|   +-- utils/              # Common utility functions
|
+-- config/
|   +-- config.yaml         # All file paths and directories
|
+-- .github/workflows/
|   +-- cicd.yaml           # GitHub Actions CI/CD workflow
|
+-- templates/              # HTML templates for Flask UI
+-- artifacts/              # Auto-generated pipeline outputs (gitignored)
+-- app.py                  # Flask application
+-- main.py                 # Pipeline orchestrator
+-- params.yaml             # Model hyperparameters
+-- schema.yaml             # Data contract / column validation
+-- Dockerfile
+-- requirements.txt
+-- setup.py
+-- README.md
```

---

## Getting Started

### 1. Clone the Repository

```bash
git clone https://github.com/Ahamed-Safnas/wine-quality-predictor-mlops.git
cd wine-quality-predictor-mlops
```

### 2. Create and Activate Conda Environment

```bash
conda create -n mlproj python=3.8 -y
conda activate mlproj
```

### 3. Install Dependencies

```bash
pip install -r requirements.txt
```

### 4. Run the Full Training Pipeline

```bash
python main.py
```

### 5. Launch the Flask App

```bash
python app.py
```

Visit `http://localhost:8080` to use the prediction UI.

---

## Pipeline Stages

All stages are orchestrated via `main.py` and configured through `config/config.yaml`.

### Stage 1: Data Ingestion
Downloads the raw wine quality dataset from a remote URL, unzips it, and stores it under `artifacts/data_ingestion/`.

### Stage 2: Data Validation
Validates all columns and dtypes against `schema.yaml` before any transformation begins. A `status.txt` file is written indicating pass or fail.

### Stage 3: Data Transformation
Performs train/test split on the validated dataset. Outputs saved to `artifacts/data_transformation/`.

### Stage 4: Model Trainer
Trains an **ElasticNet** regression model using hyperparameters from `params.yaml`:

```yaml
ElasticNet:
  alpha: 0.2
  l1_ratio: 0.1
```

Saves the trained model as `model.joblib` under `artifacts/model_trainer/`.

### Stage 5: Model Evaluation
Evaluates the model on the test set and logs the following metrics to **MLflow**:

- RMSE (Root Mean Squared Error)
- MAE (Mean Absolute Error)
- R2 Score

Metrics are also saved locally to `artifacts/model_evaluation/metrics.json`.

---

## Experiment Tracking

MLflow is used to track all training runs and evaluation metrics. Each run logs hyperparameters and performance scores for easy comparison.

To view the MLflow UI locally:

```bash
mlflow ui
```

Then visit `http://localhost:5000`.

---

## Flask Web App

The Flask app exposes three routes:

| Route | Method | Description |
|---|---|---|
| `/` | GET | Home page |
| `/train` | GET | Triggers the full training pipeline |
| `/predict` | POST | Accepts 11 feature inputs and returns a quality prediction |

---

## CI/CD Deployment

The deployment pipeline is fully automated via **GitHub Actions** and triggers on every push to `main`.

### Flow

```
Code Push to main
       |
       v
GitHub Actions (CI)
  - Lint check
  - Unit tests
       |
       v
Build Docker Image
       |
       v
Push to AWS ECR
       |
       v
Deploy to AWS EC2 (self-hosted runner)
  - Pull image from ECR
  - Run container on port 8080
```

### AWS Setup Required

1. Create an IAM user with `AmazonEC2FullAccess` and `AmazonEC2ContainerRegistryFullAccess`
2. Create an ECR repository
3. Launch an EC2 instance (Ubuntu) and install Docker
4. Configure EC2 as a self-hosted GitHub Actions runner
5. Add the following GitHub Secrets:

```
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_REGION
AWS_ECR_LOGIN_URI
ECR_REPOSITORY_NAME
```

---

## License

MIT License — see [LICENSE](LICENSE) for details.

---

<p align="center">Built by <a href="https://github.com/Ahamed-Safnas">Ahamed Safnas</a></p>
