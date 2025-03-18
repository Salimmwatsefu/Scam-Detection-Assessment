# Swahili SMS Scam Detection Model

This repository contains the implementation of a Swahili SMS Scam Detection Model using NLP techniques. The goal is to classify SMS messages as `'scam'` or `'trust'` (legitimate) using the Swahili SMS Detection Dataset.

## Dataset
The dataset used is the "Swahili SMS Detection Dataset" by Henry Dioniz, sourced from Kaggle:  
[https://www.kaggle.com/datasets/henrydioniz/swahili-sms-detection-dataset](https://www.kaggle.com/datasets/henrydioniz/swahili-sms-detection-dataset).  
- Stored in `data/bongo_scam.csv`.  
- License details are in `data/LICENSE.txt`.

## 1. Data Exploration & Preprocessing

### 1.1 Loading the Dataset and Initial Inspection
**Approach**:  
The dataset was loaded using Pandas to examine its structure, column names, class distribution, and to check for missing values.

**Results**:  
- **Sample Data**: The dataset includes columns `'Category'` (labels: `'scam'` or `'trust'`) and `'Sms'` (message text). Example entries show `'scam'` messages with commands and `'trust'` messages with polite tones.  
- **Columns**: `['Category', 'Sms']`.  
- **Class Distribution**: 1000 `'scam'` messages (66%) and 508 `'trust'` messages (34%), indicating a class imbalance.  
- **Missing Values**: None found, ensuring data completeness.

### 1.2 Text Preprocessing
**Approach**:  
The `'Sms'` column was preprocessed by converting text to lowercase, removing punctuation and numbers, tokenizing into words, and filtering out a basic set of Swahili stopwords. A new column, `'cleaned_sms'`, was created with the processed text.

**Results**:  
- **Sample Preprocessed Data**: Original messages like "IYO PESA ITUME KWENYE NAMBA HII 0657538690" became "iyo pesa itume kwenye namba jina italeta magom...", with numbers and punctuation removed and stopwords filtered.  
- Preprocessing preserved key terms like "pesa" (money) and "namba" (number) while reducing noise.

### 1.3 Exploratory Visualizations
**Approach**:  
A word cloud was generated for `'scam'` messages to visualize frequent terms, and a bar plot was created to show the top 10 words in `'scam'` messages.

**Results**:  
- **Word Cloud**: Saved as `scam_wordcloud.png` in `reports/visualizations/`. Prominent terms include "namba," "jina," and "pesa."  
- **Top Words Bar Plot**: Saved as `scam_top_words.png`. Top words are "namba" (461), "jina" (460), "kwenye" (378), "iyo" (247), "pesa" (204), and others, highlighting scam-related vocabulary.

### 1.4 Linguistic Patterns in Scam Messages
**Approach**:  
Top unigrams, bigrams, and sample `'scam'` messages were analyzed to identify three unique linguistic patterns distinguishing `'scam'` from `'trust'` messages.

**Results**:  
- **Top Words in Scam**: "namba" (461), "jina" (460), "kwenye" (378), "iyo" (247), "pesa" (204), "piga" (152), "hela" (135), "nitumie" (130), "yako" (124), "ela" (117).  
- **Top Bigrams in Scam**: "kwenye namba" (128), "namba jina" (115), "nitumie kwenye" (105), "jina litakuja" (95), "iyo hela" (83), etc.  
- **Patterns Identified**:  
  1. **Imperative Requests for Money Transfer**: Commands like "nitumie" (send me) and "piga" (call) with money terms "pesa" and "hela."  
  2. **Instructions Involving Phone Numbers**: Frequent use of "namba" and "kwenye," often as "kwenye namba."  
  3. **Bait with Vague Promises**: Use of "jina" in phrases like "jina litakuja" or lures like "tiba asili."

