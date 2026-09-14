# Model Explainability with SHAP and LIME

## 1. Overview

Fraud detection models often make decisions using many interacting transaction features. Explainability techniques help analysts and users understand **why** a transaction was classified as fraudulent or legitimate.

This document describes two popular approaches:

- **SHAP** — SHapley Additive exPlanations
- **LIME** — Local Interpretable Model-agnostic Explanations

In FraudShield, SHAP is implemented for the Random Forest fraud detection model. LIME is a useful complementary technique, particularly for models such as the LSTM that are more difficult to explain directly.

## 2. Why Explainability Matters

Explainability supports:

- Fraud analyst investigation
- Customer-service explanations
- Model debugging and validation
- Feature importance analysis
- Regulatory and audit requirements
- Detection of unintended correlations or biased behavior

An explanation describes model behavior. It does not prove that a feature caused fraud.

## 3. SHAP

### 3.1 Definition

SHAP is based on Shapley values from cooperative game theory. Each feature is treated as a contributor to the final prediction.

Conceptually:

```text
Final prediction =
    Base prediction
    + contribution of feature 1
    + contribution of feature 2
    + ...
```

For an individual transaction:

- A positive SHAP value increases the fraud prediction.
- A negative SHAP value decreases the fraud prediction.
- A larger absolute SHAP value indicates a stronger influence.

### 3.2 SHAP Implementation in FraudShield

The project creates a tree-based explainer for the trained Random Forest model:

```python
explainer = shap.TreeExplainer(rf_model)
shap_values = explainer.shap_values(X_test)
```

For binary classification, the fraud-class SHAP values are selected:

```python
if isinstance(shap_values, list):
    shap_values_fraud = shap_values[1]
else:
    shap_values_fraud = shap_values
```

The implementation is available in:

- [`fraud_model.py`](<e:/Portfolio 101/Portfolio Projects/FraudShield-AI-Powered-Fraud-Detection-System-main/FraudShield-AI-Powered-Fraud-Detection-System-main/fraud_model.py>)
- [`shap_analysis.py`](<e:/Portfolio 101/Portfolio Projects/FraudShield-AI-Powered-Fraud-Detection-System-main/FraudShield-AI-Powered-Fraud-Detection-System-main/shap_analysis.py>)

### 3.3 SHAP Visualizations

#### Summary Bar Plot

Displays global feature importance using the average absolute SHAP value. Features at the top have the greatest overall influence on fraud predictions.

#### Summary Scatter Plot

Shows the distribution of SHAP values for each feature:

- Red points represent high feature values.
- Blue points represent low feature values.
- Values on the right increase fraud risk.
- Values on the left decrease fraud risk.

#### Dependence Plot

Shows how the value of one feature affects its SHAP contribution. It can also reveal interactions with other features.

#### Force Plot

Explains one prediction by showing which features push the model toward fraud or legitimate classification.

#### Waterfall Plot

Provides a step-by-step breakdown from the model's base value to the final prediction.

#### Decision Plot

Displays how features progressively move predictions from the base value toward their final outputs.

### 3.4 Example SHAP Interpretation

```text
Base fraud probability: 0.20
High transaction amount: +0.35
Unusual device:          +0.18
Low transaction velocity: -0.05
Final fraud probability: 0.68
```

The transaction is classified as high risk primarily because of its high amount and unusual device. Low transaction velocity slightly reduces the risk.

## 4. LIME

### 4.1 Definition

LIME means **Local Interpretable Model-agnostic Explanations**. It explains one prediction by approximating the complex model near a specific transaction with a simpler interpretable model.

### 4.2 LIME Process

For a selected transaction, LIME:

1. Creates many slightly modified versions of the transaction.
2. Obtains predictions for those modified samples.
3. Gives greater weight to samples closer to the original transaction.
4. Fits a simple local model, such as a linear model.
5. Uses the local model's coefficients as the explanation.

Example:

```text
amount = high              +0.31
device = unfamiliar        +0.22
location = unusual         +0.15
account_age = high         -0.08
```

The signs indicate whether the feature supports or opposes the fraud prediction.

### 4.3 LIME in FraudShield

LIME is not currently implemented in the project. It could be added as a complementary explainer for:

- LSTM predictions
- SVM predictions
- A unified prediction API
- Models for which a specialized SHAP explainer is unavailable

## 5. SHAP and LIME Comparison

| Aspect | SHAP | LIME |
|---|---|---|
| Main purpose | Local and global explanations | Mainly local explanations |
| Theoretical basis | Game theory | Local surrogate model |
| Model support | Broad, with specialized explainers | Model-agnostic |
| Stability | Generally more consistent | May vary between runs |
| Global feature analysis | Strong | Limited |
| Computational cost | Can be expensive | Can be expensive |
| FraudShield status | Implemented | Not currently implemented |

## 6. Recommended Usage

FraudShield should use the methods as follows:

- Use **SHAP** for global feature importance and detailed Random Forest explanations.
- Use **LIME** for local explanations of models such as the LSTM or SVM.
- Compare explanations from both methods when validating important decisions.
- Present explanations as supporting evidence for analysts, not as causal proof.

## 7. Limitations

### SHAP Limitations

- Exact Shapley-value calculations can be computationally expensive.
- Explanations depend on the selected background data.
- A feature contribution does not establish causation.
- Different model output formats require careful handling.

### LIME Limitations

- Explanations are local and may not represent global model behavior.
- Results can change depending on random perturbations.
- The local surrogate may not accurately approximate the original model.
- Discretization choices can affect the explanation.

## 8. Conclusion

SHAP and LIME make FraudShield's predictions easier to inspect and communicate. SHAP provides the project's current explanation framework, including global importance, local prediction breakdowns, feature dependence, and decision-path visualizations. LIME offers a useful model-agnostic complement for models that are more difficult to explain directly.

Together, these techniques can improve fraud investigation, model governance, debugging, and user trust while preserving the predictive capabilities of machine learning.
