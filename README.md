# Comparative Analysis of Network Intrusion Detection System Using Semi-Supervised Machine Learning on the CIC-IDS2017 Dataset

# Problem
This project serves as a requirement for AIM's Post Graduate Diploma in Artificial Intelligence and Machine Learning.

Modern enterprise networks generate massive volumes of traffic, making manual monitoring by Security Operations Center (SOC) analysts impractical. Traditional rule-based intrusion detection systems rely on predefined signatures and static thresholds, which struggle to detect:

- previously unseen (zero-day) attacks,
- evolving attack patterns,
- subtle behavioral anomalies hidden within normal traffic.

This results in:

- missed intrusions (false negatives),
- excessive alerts (false positives),
- analyst fatigue and delayed incident response.

With these, development of an intelligent anomaly detection system is needed capable of automatically identifying suspicious network behavior from traffic data without relying on labeled attack signatures.

The system should assist SOC analysts by prioritizing anomalous traffic flows that are most likely to represent cyber intrusions.

## Objective
The task is to detect abnormal network traffic patterns using the CICIDS2017 dataset, which contains realistic benign and attack traffic across multiple intrusion scenarios.

Because real-world cybersecurity environments often lack reliable labels and must detect unknown attacks, the problem will be framed as unsupervised anomaly detection.

### Tasks
Build unsupervised machine learning model that:
1. Learn baseline behaviour of benign network traffic
2. Assign anomaly scores/reconstruction error to samples
3. Detect attack traffic with no/minimal labeling
4. Reduce noise/false alarms while maintaining precise detection capability

Aside from unsupervised model, we also build a supervised that serves as benchmark and also a fallback if unsupervised didn't have satisfactory results. 

### Models Explored
- Isolation Forest
- Autoencoder Neural Network
- XGBoost

## Success Metrics
Since CICIDS2017 is highly imbalanced, traditional accuracy is insufficient. Evaluation focuses on anomaly-detection-appropriate metrics:

### Primary Metrics
- PR-AUC: Metric for highly imbalanced dataset (like anomalies) that is agnostic on thresholds. This is good if business deems cost of false positives outweighs benefit of catching all intrusions
- Recall: Ability to detect attack among all data
- Precision: Measures alert quality (reducing false alarms)
- False Positive Rate: Frequency of false alarms

### Business KPI
|KPI|Impact|
|---|---|
|Alert Reduction|Less analyst overload|
|Top-K Detection Rate|Analyst able to prioritize attacks effectively|
|False Alert Cost|Direct Financial Impact|

# Exploratory Data Analysis (EDA) & Feature Engineering

To prepare the CIC-IDS2017 dataset for modeling, a rigorous data cleaning, feature selection, and engineering pipeline was implemented. Due to the massive size of the original dataset, a random sample of 30% was utilized to optimize model training and EDA efficiency.

## Data Preprocessing & Cleaning
The initial data inspection revealed the need for standard cleaning procedures to ensure model stability:
* **Whitespace Removal:** Leading and trailing whitespaces were stripped from all column headers to standardize feature names.
* **Duplicate Handling:** Duplicate rows were dropped to reduce redundancy and help flatten the instance distribution.
* **Identical Columns:** Column-wise comparisons were executed to identify and drop perfectly identical features.
* **Null and Infinite Values:** Missing (NaN) and infinite (Inf/-Inf) values were dropped, as they accounted for less than 1% of the total dataset.
* **Zero-Variance Features:** Columns containing only a single unique value were identified and removed since they offer no predictive power.

## Feature Selection & Correlation Analysis
To mitigate multicollinearity and reduce dimensionality without losing critical information:
* **Correlation Matrix:** A correlation heatmap was generated to identify highly correlated numerical feature pairs.
* **Mutual Information (MI) Scores:** Feature importance was calculated using Mutual Information scores to quantify how much each feature contributes to predicting an anomaly.
* **Strategic Dropping:** For feature pairs with an extreme correlation coefficient greater than 0.9999, the feature with the lower MI score was systematically dropped to eliminate redundancy while preserving the most informative variables.

## Domain-Specific Feature Engineering
To better capture the behavioral signatures of network attacks, several domain-specific features were engineered from the raw network flow metrics:
* **Bytes per Packet:** Calculated as `Total Length of Fwd Packets / Total Fwd Packets` to distinguish between tiny packet scans and large packet exfiltration.
* **Packet Rate Ratio:** Calculated as `Flow Packets/s / Flow Bytes/s` to detect volume-independent burst traffic.
* **Forward/Backward Packet Ratio & Direction Dominance:** Formulated to detect one-sided traffic characteristic of DDoS attacks and scans.
* **SYN & ACK Ratios:** Extracted to measure specific TCP flag responses relative to packet counts.
* **Log Transformations:** Applied `log1p` transformations to heavy-tailed metrics (`Flow Bytes/s`, `Flow Packets/s`, `Flow Duration`) to normalize their distributions.
* **Destination Port Distribution:** Converted the categorical `Destination Port` into a normalized frequency distribution.

## Novel Class Labeling (Zero-Day Preparation)
To rigorously test the models' zero-day detection capabilities, the target labels were mapped into a specific hierarchy for novelty detection testing:
* **Baseline (0):** Normal, benign network traffic (`BENIGN`).
* **Supervised/Known Attacks (1):** High-volume, distinct attacks used for training (`DoS Hulk`, `DDoS`, `PortScan`, `FTP-Patator`, `SSH-Patator`).
* **Novel/Benchmark Attacks (2):** Sophisticated, stealthy, or low-volume attacks explicitly excluded from training to test anomaly detection capabilities (`Infiltration`, `Bot`, `DoS slowloris`, `Heartbleed`, and various `Web Attacks`).

# Model Performance

The evaluation compared three model architectures—**Isolation Forest (IF)**, **Autoencoder (AE)**, and **XGBoost (XGB)**—across holistic testing and novel (zero-day) attack scenarios.

## Holistic Performance (Common + Novel Attacks)
In holistic testing, XGBoost served as the supervised benchmark and significantly outperformed the unsupervised models. Among the unsupervised approaches, the Autoencoder (AE 2) demonstrated higher precision and a lower false positive rate (FPR) than Isolation Forest.

| Model | Precision | Recall | F1-Score | ROC AUC | PR AUC |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **XGBoost** | 0.9939 | 0.9736 | 0.9836 | 0.9990 | 0.9974 |
| **AE 2** | 0.8852 | 0.5603 | 0.6863 | 0.8909 | 0.7337 |
| **AE 1** | 0.7888 | 0.5826 | 0.6702 | 0.8428 | 0.6738 |
| **IF 2** | 0.5823 | 0.5905 | 0.5864 | 0.8454 | 0.5407 |
| **IF 1** | 0.3532 | 0.6606 | 0.4603 | 0.7581 | 0.3364 |

*(Metrics sourced from the holistic testing results)*


## Novel Attack Testing
This phase isolated novel attacks to simulate a model encountering unknown threats not present in its training set.

* **Unsupervised Superiority:** The Autoencoder (AE 2) was notably more effective than Isolation Forest for novelty detection, achieving a PR AUC of 0.2663 compared to 0.0747 for IF 2.
* **Detection Logic:** Isolation Forest struggled with novel attacks because it failed to isolate subtle deviations from normal data, leading to low precision (0.12–0.16). Autoencoders were more effective as they flag data failing to fit the learned "Normal" reconstruction template.
* **Supervised Resilience:** Although its performance dropped from holistic levels, XGBoost remained the most effective model at identifying novel data as "Not Normal," achieving an F1-score of 0.6772.


## Business KPIs
Performance was translated into operational and financial impact to justify model selection for a Network Intrusion Detection System (NIDS).

* **Alert Reduction:** This measures the ability to filter network noise. XGBoost achieved a 99.16% reduction, while AE 2 achieved 98.86%.
* **Top-K Detection:** In a scenario where only 37% of daily alerts can be investigated, XGBoost reliably identified 99.07% of actual attacks, while AE 2 identified 90.26%.
* **False Alert Cost:** Wasted analyst resources due to high False Positive Rates (FPR) resulted in the following estimated annual costs:
    * **XGBoost:** ~₱97,905.51.
    * **AE 2:** ~₱430,920.68.


## Conclusion
**XGBoost** is recommended as the optimal choice for this NIDS implementation due to its high technical accuracy and low operational cost. Despite being a supervised model, it proved more effective at detecting novel attacks than the unsupervised alternatives. Among unsupervised options, the **Autoencoder (AE 2)** is the superior choice for its higher precision and improved reliability in zero-day scenarios.

# Ethical AI & Bias Auditing

To ensure the integrity, transparency, and operational fairness of the deployed models, an analysis was conducted covering model explainability, technical limitations, and strategies for bias detection.

## Model Explainability (SHAP Analysis)
To understand the decision-making process of the XGBoost classifier, a SHAP (SHapley Additive exPlanations) KernelExplainer was utilized. The SHAP beeswarm plot reveals that a small subset of features dominates the model’s predictions, while the majority contribute minimally. 

Top feature interpretations include:
* **Initial Window Bytes (Forward & Backward):** These are the most influential features, where both high and low values push the prediction toward "Benign" (negative). Standard window sizes (e.g., 8192 or 65535) are highly characteristic of clean handshakes, providing a strong benign signal for the model.
* **Bwd Packet Length Std:** Lower values for this feature have a significant negative SHAP impact. Low variability in packet length suggests a consistent, non-erratic flow, which the model correctly interprets as a sign of safe traffic.
* **Fwd IAT Min:** Data points for the minimum forward inter-arrival time are clustered between -0.01 and -0.02. The normal timing of these packets helps the model confirm that the traffic is not an attack.
* **Supporting Features:** Features like Average Packet Size and Bwd Header Length behave similarly, clustering near the -0.01 mark. These supporting features do not radically change predictions on their own, but they provide consistent evidence for the final classification.

## Addressing Model Limitations
Several strategies were implemented to address common machine learning limitations that could skew the model's reliability:
* **Class Imbalance:** The CICIDS2017 dataset is highly imbalanced because benign traffic heavily dominates the attack samples. This was mitigated by evaluating the models using metrics suitable for imbalanced data, such as PR-AUC, Precision, Recall, and F1-Score. Furthermore, the `scale_pos_weight` parameter was utilized in the XGBoost model to account for these class imbalances during training.
* **Data Leakage:** Data leakage can cause a model to overestimate its performance if it "sees" features from the future or features derived from the target variable. This was prevented by strictly dropping labels and engineered columns prior to training. Additionally, scalers and encoders were fitted exclusively on training data before being transformed onto the test and validation sets.
* **Overfitting:** Tree ensemble models like XGBoost are prone to memorizing high-dimensional training data. Overfitting was mitigated through model regularization by limiting hyperparameter values, such as max depth, min child weight, colsample bytree, and subsample. Stratified KFold was also used to ensure minority classes were properly represented in each fold, and various feature selection methods (e.g., variance analysis, correlation analysis, PCA) were applied during preprocessing to reduce dimensionality.

## Bias Detection & Fairness Audits
While traditional machine learning fairness focuses on sensitive human demographic groups (such as gender, race, or age), bias in an Intrusion Detection System (IDS) extends to operational unfairness. 

* **Defining IDS Bias:** Different subnets, network groups, or countries may generate traffic differently. If the IDS is more likely to falsely flag a specific group's traffic as malicious, it creates an operational bias.
* **Proposed Audit Framework:** Although it is not possible to test this with the currently available data, a framework for future fairness audits is proposed:
    * Define network groups based on traffic context, such as source IP, device type, or protocol.
    * Measure and track group-level evaluation metrics.
    * Apply fairness metric analogs to ensure an equal detection rate across all network groups.
    * Mitigate detected biases by oversampling or simulating rare/novel attacks in underrepresented groups, or by adjusting classification thresholds on a per-group basis.