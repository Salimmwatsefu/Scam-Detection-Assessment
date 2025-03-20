# Swahili SMS Scam Detection Model

This repository contains the implementation of a Swahili SMS Scam Detection Model using NLP techniques. The goal is to classify SMS messages as `'scam'` or `'trust'` (legitimate) using the Swahili SMS Detection Dataset.

## Dataset
The dataset used is the "Swahili SMS Detection Dataset" by Henry Dioniz, sourced from Kaggle:  
[https://www.kaggle.com/datasets/henrydioniz/swahili-sms-detection-dataset](https://www.kaggle.com/datasets/henrydioniz/swahili-sms-detection-dataset).  
- Stored in `data/bongo_scam.csv`.  
- License details are in `data/LICENSE.txt`.

## 1. Data Exploration & Preprocessing

### 1.1 Loading the Dataset and Initial Inspection
**Approach** 
The dataset was loaded using Pandas to examine its structure, column names, class distribution, and to check for missing values.

**Results**
- **Sample Data**: The dataset includes columns `'Category'` (labels: `'scam'` or `'trust'`) and `'Sms'` (message text). Example entries show `'scam'` messages with commands and `'trust'` messages with polite tones.  
- **Columns**: `['Category', 'Sms']`.  
- **Class Distribution**: 1000 `'scam'` messages (66%) and 508 `'trust'` messages (34%), indicating a class imbalance.  
- **Missing Values**: None found, ensuring data completeness.

### 1.2 Text Preprocessing
**Approach**  
The `'Sms'` column was preprocessed by converting text to lowercase, removing punctuation and numbers, tokenizing into words, and filtering out a basic set of Swahili stopwords. A new column, `'cleaned_sms'`, was created with the processed text.

**Results**
- **Sample Preprocessed Data**: Original messages like "IYO PESA ITUME KWENYE NAMBA HII 0657538690" became "iyo pesa itume kwenye namba jina italeta magom...", with numbers and punctuation removed and stopwords filtered.  
- Preprocessing preserved key terms like "pesa" (money) and "namba" (number) while reducing noise.

### 1.3 Exploratory Visualizations
**Approach**  
A word cloud was generated for `'scam'` messages to visualize frequent terms, and a bar plot was created to show the top 10 words in `'scam'` messages.

**Results**  
- **Word Cloud**: Saved as `scam_wordcloud.png` in `reports/visualizations/`. Prominent terms include "namba," "jina," and "pesa."  
- **Top Words Bar Plot**: Saved as `scam_top_words.png`. Top words are "namba" (461), "jina" (460), "kwenye" (378), "iyo" (247), "pesa" (204), and others, highlighting scam-related vocabulary.

### 1.4 Linguistic Patterns in Scam Messages
**Approach**  
Top unigrams, bigrams, and sample `'scam'` messages were analyzed to identify three unique linguistic patterns distinguishing `'scam'` from `'trust'` messages.

**Results** 
- **Top Words in Scam**: "namba" (461), "jina" (460), "kwenye" (378), "iyo" (247), "pesa" (204), "piga" (152), "hela" (135), "nitumie" (130), "yako" (124), "ela" (117).  
- **Top Bigrams in Scam**: "kwenye namba" (128), "namba jina" (115), "nitumie kwenye" (105), "jina litakuja" (95), "iyo hela" (83), etc.  
- **Patterns Identified**:  
  1. **Imperative Requests for Money Transfer**: Commands like "nitumie" (send me) and "piga" (call) with money terms "pesa" and "hela."  
  2. **Instructions Involving Phone Numbers**: Frequent use of "namba" and "kwenye," often as "kwenye namba."  
  3. **Bait with Vague Promises**: Use of "jina" in phrases like "jina litakuja" or lures like "tiba asili."


## 2. Model Development
 ## 2.1 Baseline Model: TF-IDF + Logistic Regression 
### Approach


#### Implementation
- **Data Preparation**: Loaded `processed_bongo_scam.csv` (1508 samples) and mapped labels (`trust`: 0, `scam`: 1). Split into 80% training (~1206 samples) and 20% testing (~302 samples) using `train_test_split` with `random_state=42`.
- **Feature Extraction**: Applied `TfidfVectorizer` (`max_features=5000`) to convert cleaned SMS texts into numerical features, capturing word importance.
- **Model Training**: Trained a `LogisticRegression` model with `class_weight='balanced'` to handle class imbalance (`trust` < `scam`) and `max_iter=1000` for convergence.
- **Hyperparameter Optimization**: Used `GridSearchCV` to tune the model, optimizing for F1-score (specific parameters tuned not shown but included in `baseline_model.ipynb`).

#### Results
- **F1-Score**: Achieved a perfect F1-score of 1.0000 on the test set (~302 samples), indicating flawless classification.
- **Confusion Matrix**: Visualized in `baseline_confusion_matrix.png`:
  - True `trust` → Predicted `trust`: 91
  - True `trust` → Predicted `scam`: 0
  - True `scam` → Predicted `trust`: 0
  - True `scam` → Predicted `scam`: 211
- **Analysis**: The perfect F1-score suggests potential data leakage (e.g., duplicates in the dataset) or overfitting due to high `max_features`. While the model performed exceptionally on the test set, its generalization to new data may be limited.

#### Setbacks and Solutions
- **Class Imbalance**: The dataset had more `scam` (211) than `trust` (91) samples in the test set. Addressed by setting `class_weight='balanced'` in Logistic Regression, ensuring the model didn’t favor the majority class.
- **Hardware Constraints**: Although the baseline model trained quickly (~seconds) and fit within 1 GB RAM, this constraint became a significant challenge for the transformer model (detailed below).

## Future Work
- **Data Validation**: Investigate data leakage in `processed_bongo_scam.csv` to ensure realistic performance.
- **Feature Engineering**: Experiment with lower `max_features` or n-grams to reduce overfitting.
- **Model Alternatives**: Test other lightweight models for comparison.


## 2.2 Transformer Model: AfroXLMR
### Approach

#### Implementation
- **Data Preparation**: Used a small 10% chunk of the data (~150 samples: ~120 to train, ~30 to test) because the full data (1508 samples) took too long to train. Split the data 80% for training and 20% for testing.
- **Model Choice**: Picked `Davlan/afro-xlmr-base` because it’s pre-trained on African languages like Swahili, making it better for our SMS messages than mBERT (which doesn’t know Swahili well).
- **Text Setup**: Turned SMS messages into numbers using a tokenizer, keeping them short (max length 50 words) to save space, since SMS messages aren’t long.
- **Training**: Trained AfroXLMR to read 8 messages at a time (batch size 8) for 1 full round (1 epoch), which took ~15 steps and ~22 minutes. Saved the results like a chart to see how well it did.

#### Results
- **F1-Score**: Got a score of 0.7083 (about 71% correct) on the ~30 test messages, meaning it’s pretty good at spotting scams but not perfect.
- **Confusion Matrix**: Saved as `transformer_confusion_matrix.png`:
  - Out of 14 real `trust` messages, it guessed 0 as `trust` and 14 as `scam` (all wrong).
  - Out of 17 real `scam` messages, it guessed 17 as `scam` and 0 as `trust` (all right).
- **What This Means**: The model guessed everything as `scam`, so it caught all the scams but missed all the safe messages. This might be because we used a small amount of data, and there were more `scam` messages (17) than `trust` (14) in the test set.

#### Setbacks and Solutions
- **RAM Problem**: My laptop only had 1 GB of free RAM, which is tiny. AfroXLMR is big (~500 MB) and needed more space (~3 GB total) to train, so it kept crashing.
- **Swap Solution**: I added 8 GB of swap space, which is like borrowing extra space from my laptop’s storage (206 GB free). This let me train AfroXLMR without crashing, but it was slow (~22 minutes) because the swap isn’t as fast as the desk (RAM).
- **Time Problem**: Training the full data (1508 samples) for 3 rounds was going to take ~102 hours (over 4 days!), so I used only 10% of the data to make it faster (~22 minutes).

## Future Work
- **More RAM**: Get a laptop with more RAM (like 4-8 GB free) to train faster without swap.
- **Full Data**: Use all 1508 samples to get better results, maybe an F1-score of ~0.8-0.9.
- **More Rounds**: Train for 2-3 rounds (epochs) to help AfroXLMR learn better.
- **Explain Predictions**: Add tools to show why AfroXLMR made its guesses, so users can trust it more.


## Explainability & Insights

### Influential Features for Scam Detection
- **Visualization**: Using the baseline model, I found the top 10 words that most strongly predict `scam` messages (visualized in `top_scam_words.png`). Key words include `freemason`, `karibu`, `biashara`, `pesa`, and `namba`.
- **What It Means**: These words show what scams often talk about—fake groups (`freemason`), friendly greetings (`karibu`), business offers (`biashara`), and requests for money (`pesa`) or phone numbers (`namba`).

### False Positives Analysis
- **Baseline**: There were no false positives (0 `trust` messages marked as `scam`), probably because the model overlearned the data or the dataset was too easy.
- **AfroXLMR**: All 14 `trust` messages were wrongly marked as `scam`. This happened because I only used a small dataset (~150 samples), so the model didn’t see enough `trust` examples to learn their patterns. Also, some `trust` messages (like "nipigie baada saa moja tafadhali") might use words like "tafadhali" that scams also use, confusing the model.

### Scam Message Patterns in Swahili SMS
- **Patterns**: Scams often use friendly words (`karibu`, `ndoto`), promise big things (`biashara`), and ask for money (`pesa`) or personal info (`namba`, `airtel`).
- **How They Work**: These messages sound urgent and pushy, asking users to act fast (e.g., "naomba unitumie iyo hela kwenye namba...").

### Recommendations for Tanzanian Telecom Companies
1. **Block Suspicious Messages**: Stop SMS with scam words like `freemason`, `ndoto`, or `unitumie`, or messages asking for money or numbers, so they don’t reach users.
2. **Warn Users**: Send alerts to users when they get a suspicious SMS, teaching them how to spot scams and stay safe.




