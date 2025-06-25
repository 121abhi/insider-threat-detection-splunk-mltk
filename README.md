# 🔐 Insider Threat Detection using Splunk MLTK

## 🚀 Overview

This project aims to detect **insider threats** based on **user behavioral anomalies** using **unsupervised machine learning** in **Splunk MLTK**. We use the `DensityFunction` algorithm to model normal behavior and flag deviations across three behavioral indicators:

* `activity_count`
* `avg_volume`
* `hour_of_day`

This solution was built for the **Splunk Hackathon - Track 4 (MLTK Challenge)**.

---

## 📁 Project Structure

```
splunk-insider-threat-app/
│
├── README.md                         # This file
├── lookups/
│   ├── insider_data.csv              # Training dataset
│   └── insider_test_data.csv         # Test dataset for dashboard
│
├── models/
│   ├── threat_model_1.pmml           # Model for activity_count
│   ├── threat_model_2.pmml           # Model for avg_volume
│   └── threat_model_3.pmml           # Model for hour_of_day
│
├── dashboards/
│   └── insider_threat_dashboard.xml  # Dashboard definition (Splunk XML)
│
├── documentation/
│   ├── Solution_Document.pdf         # 2-page solution writeup
│   └── architecture.png              # Optional: data flow diagram
```

---

## 📊 Machine Learning Models

We trained 3 models using **DensityFunction**:

| Model Name               | Field Analyzed   | Model File            |
| ------------------------ | ---------------- | --------------------- |
| Threat Drift Detection   | `activity_count` | threat\_model\_1.pmml |
| Threat Drift Detection 2 | `avg_volume`     | threat\_model\_2.pmml |
| Threat Drift Detection 3 | `hour_of_day`    | threat\_model\_3.pmml |

All models use the following parameters:

* `Distribution`: Auto (Wasserstein distance)
* `Threshold`: `0.0001` for anomaly detection
* `Split Method`: All data (no supervised label)
* `Algorithm`: `DensityFunction` (unsupervised)

---

## 🧪 How to Use All Models on Test Data

1. **Upload your test dataset (`insider_test_data.csv`) as a lookup.**

2. **Run this SPL to apply all 3 models**:

```spl
| inputlookup insider_test_data.csv
| eval hour_of_day = strftime(strptime(timestamp, "%Y-%m-%d %H:%M:%S"), "%H")
| stats count AS activity_count avg(volume) AS avg_volume BY username, activity_type, hour_of_day, location, status
| apply "Threat Drift Detection"
| rename "Threat Drift Detection".isOutlier AS activity_count_outlier
| apply "Threat Drift Detection avg_volume"
| rename "Threat Drift Detection avg_volume".isOutlier AS avg_volume_outlier
| apply "Threat Drift Detection hours_of_day"
| rename "Threat Drift Detection hours_of_day".isOutlier AS hour_outlier
| eval total_score = activity_count_outlier + avg_volume_outlier + hour_outlier
| eval threat_level = case(total_score >= 2, "High", total_score == 1, "Medium", true(), "Normal")
| table username activity_type location status total_score threat_level
```

### ✅ Output Columns:

* `activity_count_outlier`: 1 if anomaly in activity count
* `avg_volume_outlier`: 1 if anomaly in average volume
* `hour_outlier`: 1 if unusual time of access
* `total_score`: Sum of all anomaly flags (0–3)
* `threat_level`: High / Medium / Normal

---

## 📉 Dashboard

You can import the `insider_threat_dashboard.xml` into Splunk:

### Panels include:

* High Threat Users
* Threat Scores by User
* Outlier Trends Over Time
* Activity Type Risk Summary

---

## ⚙️ Setup Instructions

### 1. Upload the datasets

```bash
Settings > Lookups > Lookup table files > Add New
```

Upload:

* `insider_data.csv` (for training)
* `insider_test_data.csv` (for testing)

### 2. Train Models

If not already trained, you can:

* Navigate to **Machine Learning Toolkit > Experiments**
* Use the SPL below to extract features:

  ```spl
  | inputlookup insider_data.csv
  | eval hour_of_day = strftime(strptime(timestamp, "%Y-%m-%d %H:%M:%S"), "%H")
  | stats count AS activity_count avg(volume) AS avg_volume BY username, activity_type, hour_of_day, location, status
  ```
* Select algorithm: `DensityFunction`
* Train for each field separately (activity\_count, avg\_volume, hour\_of\_day)
* Save each model (give meaningful names like `Threat Drift Detection`)

### 3. Test the Model

Use the **combined SPL** from the section above.

---

## 📈 Results

* Total anomalies detected: ✅ See dashboard
* Threat levels assigned: ✅ High / Medium / Normal
* Real-time ready: ✅ Just connect to a live stream instead of lookup

---

## 🛠️ Future Work

* Integrate user roles & device data
* Alerting system for High threats
* Real-time streaming integration
* Model drift monitoring & retraining

---

## 📋 Dependencies

* Splunk Enterprise 9+
* Splunk Machine Learning Toolkit App
* Permissions to create Lookups, Models, and Dashboards

---

## 📌 Authors & Acknowledgments

* Author: Abhijeet Basant
* Special Thanks: Splunk Hackathon team and mentors!

