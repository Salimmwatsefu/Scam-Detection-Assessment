# Findings: Key Insights & Ethical Considerations

## Data Exploration & Preprocessing Insights

### Dataset Structure
- The dataset contains 1508 SMS messages, with 1000 labeled `'scam'` (66%) and 508 labeled `'trust'` (34%).  
- No missing values, ensuring data completeness for analysis.

### Preprocessing Observations
- Preprocessing removed noise (numbers, punctuation) and common Swahili stopwords (e.g., "na," "ya"), preserving key content like "pesa" (money) and "namba" (number).  
- Example: "IYO PESA ITUME KWENYE NAMBA HII 0657538690" → "iyo pesa itume kwenye namba jina italeta magom...".

### Linguistic Patterns in Scam Messages
1. **Imperative Requests for Money Transfer**:  
   - Commands like "nitumie" (send me, 130) and "piga" (call, 152) paired with "pesa" (204) and "hela" (135).  
   - Insight: Scammers use direct, actionable language to prompt quick responses (e.g., "Naomba unitumie iyo Hela").

2. **Instructions Involving Phone Numbers**:  
   - "namba" (number, 461) and "kwenye" (to/on, 378) dominate, often as "kwenye namba" (128).  
   - Insight: Phone numbers are a structural element, directing victims to send money or information.

3. **Bait with Vague Promises or Formats**:  
   - "jina" (name, 460) in phrases like "jina litakuja" (95) or lures like "tiba asili" (natural cure, 79).  
   - Insight: Scammers entice with mysterious promises or formats, unlike the polite tone of `'trust'` messages.

### Key Insights
- **Class Imbalance**: The 2:1 ratio of `'scam'` to `'trust'` suggests scammers are more active or collected more aggressively, impacting model training.  
- **Scam Characteristics**: Money-related terms ("pesa," "hela") and phone numbers ("namba") are hallmarks of scams, absent in `'trust'` top words.  
- **Language Style**: `'scam'` messages are directive and baiting, while `'trust'` messages are polite and conversational (e.g., "tafadhali" vs. "nitumie").

## Ethical Considerations
- **Bias in Data**: The imbalance (66% scam) may bias models toward over-predicting scams, potentially flagging legitimate messages (false positives).  
- **Privacy**: SMS data includes phone numbers (e.g., "0657538690"), raising concerns about anonymization in real-world use.  
- **Cultural Sensitivity**: Patterns like "freemason" or "tiba asili" may reflect local beliefs; mislabeling them could offend users or miss context.


### Baseline Model: TF-IDF + Logistic Regression
- **Approach**: Implemented TF-IDF vectorization (`max_features=5000`) to convert Swahili SMS texts into numerical features, followed by Logistic Regression with `class_weight='balanced'` to address class imbalance. Hyperparameter optimization was performed using `GridSearchCV` to tune the model.
- **Performance**: Achieved an F1-score of 1.0000 on the test set (~302 samples). The confusion matrix (saved as `baseline_confusion_matrix.png`) showed perfect classification: 91 true `trust` and 211 true `scam` samples, with no misclassifications.
- **Insight**: The perfect F1-score suggests either an overly simplistic dataset or overfitting due to high `max_features`. While the model is fast and effective as a baseline, its performance may not generalize to new, unseen data. The balanced class weighting successfully mitigated the moderate imbalance (`trust`: 91, `scam`: 211 in test set).

## Ethical Considerations
- **Bias and Generalization**: The perfect F1-score raises concerns about overfitting or data leakage, which could lead to poor performance on real-world SMS data, potentially missing scams and affecting user trust.
- **Privacy**: Assumed that `processed_bongo_scam.csv` is anonymized to protect user identities, as SMS data can contain sensitive information.
- **Fairness**: While `class_weight='balanced'` addressed imbalance, over-reliance on a small dataset risks biased predictions if the training data doesn’t represent diverse scam patterns.

## Recommendations
- **Data Quality**: Investigate potential data leakage (e.g., duplicates) in `processed_bongo_scam.csv` to ensure realistic performance.
- **Feature Reduction**: Lower `max_features` (e.g., 1000) to reduce overfitting risk.
- **Cross-Validation**: Use k-fold cross-validation to validate the model’s generalization.