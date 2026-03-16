# Predictive Alerting for Cloud Metrics

## 1. Project Overview

This project implements a predictive alerting system designed to anticipate cloud service incidents. The goal is to predict the probability of an incident occurring within a future horizon (**H**) based on a historical window of metrics (**W**).

## 2. Problem Formulation

I treated the task as a **binary classification problem** on time-series data using a **sliding window** approach:

- **Window (W):** 60 steps (1 hour of history).
- **Horizon (H):** 15 steps (15-minute prediction lead time).
- **Target (y):** Binary label, where `1` indicates at least one incident point within the next H steps.

## 3. Dataset

Due to the sensitivity of real cloud metrics, I generated a **synthetic dataset** that simulates typical CloudWatch patterns:

- Daily seasonality (circadian patterns).
- Gaussian noise.
- Injected anomalies (random spikes and sustained threshold breaches).

## 4. Modeling Choices

- **Model:** `RandomForestClassifier` with `class_weight='balanced'`.
- **Reasoning:** Random Forest is robust to noise and non-linear patterns, making it an excellent baseline for skewed distributions common in cloud metrics.
- **Preprocessing:** The sliding window transforms the time-series into a tabular format suitable for supervised learning.

## 5. Evaluation & Results

Given the imbalanced nature of incidents (~8% positive class), I optimized the model using the **Precision-Recall Curve** rather than accuracy.

- **Recall Target:** I dynamically adjusted the probability decision threshold (lowered to 0.02) to achieve the target of **~80% Recall**, as specified in the project requirements.
- **Results on Test Set:** - **Recall:** 0.79 (79% of upcoming incidents successfully anticipated).
  - **Precision:** 0.07 (Demonstrating a steep precision-recall trade-off; capturing 80% of anomalies in this noisy setup results in a high false-positive rate).
  - **PR-AUC:** 0.450

_Discussion:_ The low precision highlights the inherent difficulty of predicting rare anomalies based solely on a single noisy metric window. While the system successfully alerts early, future improvements would require multivariate feature engineering or recurrent architectures (e.g., LSTMs) to filter out false positives.

## 6. Implementation in a Real System

For a production AWS environment, the implementation would consist of:

1. **Streaming Inference:** A Lambda function triggered by CloudWatch Events every minute to run the model on the latest W steps.
2. **Alerting:** Triggering SNS notifications or PagerDuty if the predicted probability exceeds the optimized threshold.
3. **Automated Retraining:** A scheduled job (SageMaker or Lambda) to retrain the model daily using the most recent data stored in S3 to adapt to shifting metric patterns.

---

## How to Run

1. Install dependencies: `pip install -r requirements.txt`
2. Open `solution.ipynb` in PyCharm or Jupyter and run all cells.
