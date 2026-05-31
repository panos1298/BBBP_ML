# Blood-Brain Barrier Permeability Prediction (BBBP)

## Project Overview

This project develops a machine learning pipeline to predict blood–brain barrier (BBB) permeability using the BBBP dataset. BBB permeability is a key ADMET property in central nervous system (CNS) drug discovery, as only compounds capable of crossing the BBB can act on brain targets.

The pipeline covers the full workflow, from molecular preprocessing of raw SMILES representations to model evaluation and validation on an independent external dataset. The workflow is designed to reflect realistic QSAR modeling conditions, with an emphasis on chemical data quality and generalization performance.

---

## Datasets

- Training set: BBBP dataset (MoleculeNet) — 2058 compounds with binary BBB permeability labels
You can find the dataset at: https://moleculenet.org/datasets-1 

- External validation set: B3DB dataset — 5682 compounds after overlapping molecules with the training set were removed prior to evaluation
You can find the dataset at: https://huggingface.co/datasets/maomlab/B3DB?

---

## Molecular Preprocessing Pipeline

A comprehensive preprocessing workflow was applied using RDKit.

Steps Included
1. SMILES Parsing
SMILES strings were converted into RDKit molecule objects.

2. Salt Removal
Salts and counterions were removed using SaltRemover.

3. Largest Fragment Selection
For multi-fragment molecules, only the largest fragment was retained.

4. Molecule Sanitization
Invalid molecular structures were removed after RDKit sanitization.

5. Molecular Standardization
The following standardization steps were applied:
Functional group normalization
Reionization
Charge neutralization
Canonical tautomer generation

6. Duplicate Detection
Duplicates were identified using canonical SMILES.

7. Conflicting Label Removal
Molecules with inconsistent BBB labels were removed entirely.

8. Manual Data Quality Inspection
Several problematic entries were manually investigated and removed, including:
Incorrect SMILES representations
Corrupted structures after preprocessing
Dataset annotation errors

---

## Feature Engineering

### Molecular Descriptors

Five physicochemical descriptors were calculated:

Molecular Weight
TPSA (Topological Polar Surface Area)
LogP
Hydrogen Bond Donors
Rotatable Bonds

### Morgan Fingerprints

Morgan fingerprints were generated with:

Radius = 2
Fingerprint size = 2048 bits

These fingerprints were concatenated with the molecular descriptors to form the final feature matrix.

---

## Scaffold-Based Train/Test Split

Instead of random splitting, a Murcko scaffold split was used.

This approach ensures that structurally similar compounds do not appear simultaneously in both training and testing sets, producing a substantially more realistic estimate of model generalization for molecular machine learning tasks.

The dataset was split approximately:

80% training
20% testing

while preserving scaffold integrity.

---

## Models Evaluated

The following machine learning models were trained and optimized:

1. Logistic Regression 
2. Support Vector Classifier (RBF kernel)
3. Random Forest Classifier
4. Histogram Gradient Boosting Classifier

A ColumnTransformer pipeline was used to apply StandardScaler to physicochemical descriptors while passing fingerprint features unchanged.

---

## Hyperparameter Optimization

Hyperparameter tuning was performed using:

- GridSearchCV
- RandomizedSearchCV

Hyperparameter optimization was performed using GridSearchCV for Logistic Regression, while RandomizedSearchCV was applied for Support Vector Classification, Random Forest, and HistGradientBoostingClassifier, with the last two undergoing two rounds of hyperparameter refinement.

Primary optimization metric:

- ROC-AUC

Cross-validation strategy:

- 5-fold cross-validation

---

## Evaluation Metrics

Models were evaluated using:

- ROC-AUC
- PR-AUC
- Recall
- Precision
- F1-score
- Confusion Matrix
- ROC Curves
- Precision-Recall Curves

---

## Final Model Performance

Model	              ROC-AUC	PR-AUC	 Recall	  F1      Precision
Logistic Regression	  0.760	    0.942	 0.921	  0.907	  0.894
SVC	                  0.812	    0.958	 0.958	  0.924	  0.893
Random Forest	      0.814	    0.958	 0.970	  0.930	  0.894
HistGradientBoosting  0.760	    0.941	 0.961	  0.924	  0.890

Overall, the Support Vector Classifier and Random Forest models demonstrated the strongest performance across all evaluation metrics, with Random Forest emerging as the most robust model, achieving a strong balance between discriminative ability, recall, and precision, and showing consistent generalization under both scaffold-based and external validation settings

---

## Permutation Feature Importance

Permutation importance analysis on the final Random Forest model revealed that physicochemical descriptors dominate predictive performance, with H-Bond Donors, TPSA, LogP, Molecular Weight, and Rotatable Bonds ranking as the top features. This is consistent with established BBB permeability rules (e.g., Lipinski's Rule of Five, CNS MPO criteria).
Morgan fingerprint bits generally showed lower individual importance, contributing fine-grained structural information that complements the global descriptor-based signal.

---

## External Validation (B3DB, 5682 compounds)

Metric      Score
ROC-AUC     0.903
PR-AUC      0.927
Recall      0.940 
Precision   0.785
F1          0.855

The final Random Forest model demonstrated strong generalization to an independent external dataset of 5682 compounds, achieving a ROC-AUC of 0.903 and a PR-AUC of     0.927 — both notably higher than those obtained on the internal scaffold-based test set. The model maintained excellent recall (0.940), correctly identifying the large majority of BBB-permeable compounds, at the cost of moderate precision (0.785), reflecting a tendency toward false positives. This precision-recall trade-off is consistent with the model's behavior on the internal test set and is generally acceptable in virtual screening contexts, where sensitivity is prioritized over specificity.

---

## Python Libraries

- pandas
- numpy
- matplotlib
- scikit-learn
- RDKit
- datasets (HuggingFace)

---

## Project Structure

```text
├── BBBP.csv
├── blood_brain_barrier_penetration.ipynb
├── README.md
├── requirements.txt
└── figures/
```

---

## Limitations

- The scaffold-based split, while substantially more rigorous than random splitting, results in a smaller and chemically more diverse test set, which may increase variance in reported performance metrics.

- The moderate class imbalance (~75/25) affects negative-class prediction performance across all models and should be considered when interpreting recall- and precision-based metrics.

- The dataset contains approximately 2,000 compounds, limiting the complexity of models that can be reliably trained and increasing the risk of variance under scaffold-based evaluation.

- The external B3DB dataset exhibits a different class distribution (~59/41 BBB+/BBB−) compared to the BBBP training set (~75/25), which should be taken into account when directly comparing internal and external performance metrics.

---

## Future Improvements

Potential future extensions include:

- Nested cross-validation
- Threshold optimization
- Graph Neural Networks (GNNs)
- Additional molecular descriptors
- DeepChem integration
- Calibration analysis

------

## Installation

Clone the repository:

```bash
git clone https://github.com/panos1298/BBBP_ML.git
cd BBBP_ML
```

Install the required Python dependencies:

```bash
pip install -r requirements.txt 
```

---

## Running the Project


Launch Jupyter Notebook:
```bash
jupyter notebook
```

Open the notebook:

```
blood_brain_barrier_penetration.ipynb
```

Run the notebook cells sequentially to reproduce the preprocessing pipeline, model training, evaluation and analysis.

---

## Author

Panagiotis Koumpouris

Machine Learning and Chemoinformatics project focused on BBB permeability prediction using molecular descriptors, Morgan fingerprints, and scaffold-based validation strategies.