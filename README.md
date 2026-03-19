# Incident Prediction via Time-Series Classification

## 📌 Project Overview
This repository contains a solution for predicting future system incidents based on historical time-series metrics. Instead of relying on static thresholds, this project frames the alerting mechanism as a **Machine Learning Binary Classification** problem using a sliding-window approach.

## 🧠 Problem Formulation
To transform a continuous time-series into a supervised learning problem, I utilized a **sliding-window** architecture:
* **W (Past Window) = 60 steps:** The model looks at the previous 60 data points of CPU usage to understand the recent trend and context.
* **H (Future Horizon) = 10 steps:** The target variable is boolean (`1` or `0`). It predicts whether an incident (anomaly/spike) will occur anywhere within the next 10 time steps.

By structuring the data this way, we convert 1D signals into a 2D tabular dataset, where each row represents a historical snapshot and its corresponding short-term future outcome.

## 🛠️ Model Selection: Random Forest
For this specific task, I selected a **Random Forest Classifier**. 
**Why?**
1. **Handles Non-Linearity:** It naturally captures complex, non-linear relationships within the sliding window without requiring heavy feature scaling (unlike Neural Networks or Logistic Regression).
2. **Robust to Noise:** Random forests reduce variance and are highly robust against the noisy nature of synthetic or real-world CPU metrics.
3. **Class Imbalance:** By utilizing the `class_weight='balanced'` parameter, the model penalizes misses on the minority class (incidents), which is critical since anomalies are rare.

## 📊 Evaluation Strategy
In alerting systems, **Accuracy is a misleading metric** because incidents are extremely rare. A model that simply predicts "No Incident" 100% of the time could achieve 99% accuracy but would be operationally useless.

Therefore, the evaluation focuses on:
* **Precision:** To prevent "Alert Fatigue." If Precision is low, engineers will receive too many False Positives and start ignoring the system.
* **Recall:** To ensure critical incidents are caught. A high recall means the model successfully identified most of the actual anomalies.
* **Custom Alert Threshold:** Instead of using the default 0.5 probability, the model is evaluated at a `0.70` confidence threshold. An alert is only fired if the model is 70% sure an incident is imminent.

## 🚧 Limitations
1. **Feature Engineering:** The current model only uses raw values. In a production scenario, adding rolling statistics (e.g., rolling mean, rolling standard deviation, or rate of change) within the window $W$ would significantly improve the model's predictive power.
2. **Lookahead Bias Risk:** When dealing with real data, extreme care must be taken during the train-test split to ensure no future data leaks into the training set (which is why chronological splitting was used here).

## 🚀 Adaptation to a Real Alerting System
To adapt this proof-of-concept into a production-grade alerting pipeline (e.g., for Site Reliability Engineering):
1. **Streaming Ingestion:** Use **Apache Kafka** or AWS Kinesis to stream real-time server metrics.
2. **Feature Store / Windowing:** A stream-processing engine (like Apache Flink) would continuously buffer the last $W$ minutes of data into memory.
3. **Real-Time Inference:** The buffered array is passed to the deployed Random Forest API.
4. **Actionable Alerts:** If the `predict_proba()` exceeds the defined threshold, an event is pushed to an incident management tool (like **PagerDuty** or Opsgenie) to notify the on-call engineer *before* the system actually crashes.

## 💻 How to Run
1. Clone this repository.
2. Ensure you have the required libraries: `pip install numpy pandas scikit-learn matplotlib seaborn`
3. Open and run the `JetBrains.ipynb` Jupyter Notebook.