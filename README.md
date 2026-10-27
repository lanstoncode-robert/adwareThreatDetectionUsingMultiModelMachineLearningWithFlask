# Network Intrusion & Malware Attack Detection System

This repository contains a machine learning-based network intrusion detection system designed to classify network traffic behavior and detect malicious attacks (such as adware, malware, or network intrusions). The project includes a full dataset preprocessing and training pipeline, alongside a live Flask web application interface (`app2.py`) that utilizes an ensemble-weighted voting strategy for traffic analysis.

---

## 🚀 Project Overview & Benchmark Dataset

The model exploration and deployment architecture is anchored entirely on the public benchmark **CICAndMal2017 Dataset** (Android Malware Dataset curated by the Canadian Institute for Cybersecurity). This dataset represents comprehensive multi-state network flow logs captured using `CICFlowMeter-V3` from real runtime installations across 42 distinct malicious families—specifically capturing behavior inside critical threat clusters like Adware, Ransomware, Scareware, and SMS Malware.

The project is divided into three primary structural modules:
1. **Data Preprocessing & Cleaning Pipeline:** Consolidates multiple standalone network capture logs, removes redundant/static attributes, converts abstract markers (like IP and timestamps) into standard numeric coordinates, and formats labels.
2. **Model Training & Evaluation (`.ipynb`):** Fits multiple machine learning architectures against the normalized metrics to discover the strongest detection thresholds.
3. **Web Application Interface (`app2.py`):** Runs a local deployment dashboard where you can manually feed in a flow log's 12 prominent features to obtain a real-time risk classification.

---

## 🧹 Data Preprocessing & Cleaning Pipeline

Before model construction, raw network flow dumps extracted from the **CICAndMal2017 Dataset** are processed within the initial stages of the pipeline to resolve column structural offsets and make text records natively compatible with Scikit-Learn. The explicit data sequence is as follows:

### 1. Multi-Source Log Merging
The network capture engine outputs individual `.csv` trace logs grouped by malware/adware signatures (`ad-dowgin`, `ewind`, `feiwo`, `gooligan`, `kemoge`, etc.). 
* Shapes are checked globally to catch column tracking misalignments (e.g., source file 2 containing an extra empty `Unnamed: 85` tracking metric which is dynamically dropped).
* All files are stacked row-wise via an unindexed spatial concentration matrix (`pd.concat(..., axis=0)`) into a single global evaluation ledger.

### 2. Feature Filtering & Dimensions Reduction
The system filters out static system variables, target leak items, and non-generalizable network identifiers from the original 80+ features found in CICAndMal2017. 26 descriptive headers are dropped entirely:
* **Structural Identifiers:** `Flow ID`, `Destination IP` (prevents model overfitting to specific machines).
* **Statistical Extremes (Min/Max boundaries):** `Fwd Packet Length Max`, `Fwd Packet Length Min`, `Bwd Packet Length Max`, `Bwd Packet Length Min`, `Flow IAT Max`, `Flow IAT Min`, `Fwd IAT Max`, `Fwd IAT Min`, `Bwd IAT Max`, `Bwd IAT Min`, `Min Packet Length`, `Max Packet Length`, `Active Max`, `Active Min`, `Idle Max`, `Idle Min`.
* **Zero-Variance/Constant Logs:** `Packet Length Variance`, `Fwd Header Length`, `Fwd Avg Bytes/Bulk`, `Fwd Avg Packets/Bulk`, `Fwd Avg Bulk Rate`, `Bwd Avg Bytes/Bulk`, `Bwd Avg Packets/Bulk`.
* **Target Leak:** The reference string target `Label` is split off entirely to prevent self-referencing.

### 3. Structural Field Encoding
To preserve the predictive signaling value hidden in categorical data fields, text columns are mapped into continuous numeric values:
* **Source IP Conversion:** Structural IPv4 configurations (e.g., `192.168.1.1`) are resolved through programmatic subnet parsing (`ipaddress.IPv4Address`) and systematically stored as unique base-10 numerical integers.
* **Timestamp Normalization:** The standard object transaction timestamps are parsed into global `datetime` sequences, shifted relative to the Unix Epoch scale, and transformed to uniform floating-point precision seconds (`Timestamp / 10**9`).
* **Label Quantization:** Target descriptions are encoded into fixed integer arrays via `LabelEncoder()`, converting textual classifications into a sequence map (e.g., mapping `BENIGN` to `0` and distinct threat patterns to active integers).

---

## 📊 Key Features for Detection

Following the dimensionality cleanup from the original **CICAndMal2017 Dataset**, the final dataset down-samples tracking properties to these **12 optimal diagnostic flow inputs**:
1. `Bwd Packet Length Min`
2. `Bwd Packet Length Std`
3. `Flow IAT Max`
4. `Fwd IAT Std`
5. `Bwd IAT Total`
6. `Bwd Packets/s`
7. `Min Packet Length`
8. `PSH Flag Count`
9. `ACK Flag Count`
10. `URG Flag Count`
11. `Init_Win_bytes_forward`
12. `min_seg_size_forward`

---

## 🔧 Prerequisites & Installation

### 1. Clone the Repository
```bash
git clone <repository_url>
cd <repository_folder>
```

### 2. Set Up a Virtual Environment (Recommended)
```bash
# Windows
python -m venv venv
venv\Scripts\activate

# macOS/Linux
python3 -m venv venv
source venv/bin/activate
```

### 3. Install Dependencies
```bash
pip install flask pandas scikit-learn joblib
```

---

## 🏃 Execution Guide

### Step 1: Prepare the Cleaned Dataset
Ensure your preprocessed dataset containing the clean columns extracted from the CICAndMal2017 files is placed in the root directory under the filename:
* **`cleaned_dataset.csv`**

### Step 2: Model Training & Serialization (Optional)
If you want to re-train the models from scratch, run the compilation cells within your Jupyter Notebook. It will evaluate model validation boundaries and serialize the objects into independent `.joblib` model binaries:
* `random_forest_model.joblib`
* `decision_tree_model.joblib`
* `svm_model.joblib`
* `knn_model.joblib`

### Step 3: Run the Web Dashboard
Execute the server routing controller file locally:
```bash
python app2.py
```

### Step 4: Run Traffic Classification Inference
1. Navigate your web browser to: `http://127.0.0`
2. Populate the interface fields (Value 1 through Value 12) with your live network metrics.
3. Click **Predict**. The system will combine metrics from your model ecosystem using a performance-weighted consensus approach to report an **Attack** or **BENIGN** outcome.
