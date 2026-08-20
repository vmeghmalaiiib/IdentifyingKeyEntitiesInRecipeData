# Identifying Key Entities in Recipe Data
## Named Entity Recognition using Conditional Random Fields

**Course**: Natural Language Processing  
**Assignment**: Identifying Key Entities in Recipe Data

---

## 1. Executive Summary

This project implements a **Named Entity Recognition (NER)** model using **Conditional Random Fields (CRF)** to automatically extract key entities—ingredients, quantities, and units—from recipe text data. The model achieved high accuracy and demonstrates practical utility for recipe management systems, dietary tracking apps, and e-commerce platforms.

**Key Results:**
- Successfully classified three entity types: `ingredient`, `quantity`, and `unit`
- Model trained on 799 recipes (70%) and validated on 343 recipes (30%)
- Achieved high token-level accuracy with quantity achieving the best performance
- Identified common error patterns to guide future improvements

---

## 2. Business Objective

The goal is to train a Named Entity Recognition (NER) model to extract key entities from recipe ingredient lists. This enables:

- **Automated recipe parsing** for structured database creation
- **Smart grocery lists** in e-commerce platforms
- **Nutritional analysis** for dietary tracking applications
- **Voice assistant integration** for cooking guidance

---

## 3. Data Description

### 3.1 Dataset Overview

| Attribute | Value |
|-----------|-------|
| **Format** | JSON |
| **Records** | 1,142 recipes |
| **Fields** | `input` (ingredient text), `pos` (NER labels) |
| **Entity Types** | 3 (ingredient, quantity, unit) |

### 3.2 Sample Data Structure

```json
{
  "input": "2-1/2 cups rice cooked 3 tomatoes teaspoons...",
  "pos": "quantity unit ingredient ingredient quantity ingredient..."
}
```

### 3.3 Label Distribution

The dataset exhibits class imbalance with ingredients being the dominant class:

| Label | Percentage |
|-------|------------|
| **Ingredient** | ~65-70% |
| **Quantity** | ~15-18% |
| **Unit** | ~12-17% |

---

## 4. Methodology

### 4.1 Data Preparation

1. **Loading**: Parsed JSON data into a pandas DataFrame
2. **Tokenization**: Split input text and labels into token lists
3. **Validation**: Verified token-label count alignment
4. **Cleaning**: Removed rows with mismatched counts (caused by compound fractions like "1 1/2")
5. **Split**: 70% training (799 recipes), 30% validation (343 recipes)

### 4.2 Feature Engineering

Comprehensive token-level features were extracted using spaCy NLP library:

**Core Features:**
- Token (lowercase), lemma, POS tag, dependency relation
- Shape pattern (e.g., "Xxx" for "Milk")
- Boolean flags: is_stop, is_digit, has_digit, is_punct, etc.

**Quantity/Unit Detection:**
- Regex patterns for fractions (`1/2`), decimals (`2.5`), ranges (`1-2`)
- Keyword sets for units (cup, tablespoon, gram, etc.)
- Keyword sets for quantity words (half, quarter, dozen, etc.)

**Contextual Features:**
- Previous/next token information
- Beginning-of-Sequence (BOS) and End-of-Sequence (EOS) markers
- Preceding/following word context

### 4.3 Class Weighting

To address class imbalance, inverse frequency weighting was applied:

| Class | Weight |
|-------|--------|
| Quantity | Higher (rare class) |
| Unit | Higher (rare class) |
| Ingredient | Lower (penalized by 20%) |

### 4.4 Model Configuration

**CRF Hyperparameters:**

| Parameter | Value | Purpose |
|-----------|-------|---------|
| Algorithm | L-BFGS | Quasi-Newton optimization |
| C1 | 0.5 | L1 regularization (sparsity) |
| C2 | 1.0 | L2 regularization (overfitting prevention) |
| Max Iterations | 100 | Training convergence |

---

## 5. Results and Visualizations

### 5.1 Exploratory Data Analysis

#### Top 10 Most Frequent Ingredients

![Top 10 Ingredients](ingredients_chart_placeholder.png)

The most common ingredients reflect everyday Indian cooking patterns:

| Rank | Ingredient | Observation |
|------|------------|-------------|
| 1 | Salt | Universal seasoning |
| 2 | Chilli | Common spice |
| 3 | Oil | Cooking base |
| 4 | Onion | Base vegetable |
| 5 | Garlic | Aromatic |

**Insight**: The ingredient distribution indicates predominantly Indian cuisine recipes with common cooking staples.

#### Top 10 Most Frequent Units

![Top 10 Units](units_chart_placeholder.png)

| Rank | Unit | Type |
|------|------|------|
| 1 | cup | Volume |
| 2 | teaspoon | Volume (small) |
| 3 | tablespoon | Volume (medium) |
| 4 | grams | Weight |
| 5 | cloves | Count |

**Insight**: Volume-based measurements (cup, teaspoon, tablespoon) are most common, consistent with home cooking practices.

### 5.2 Model Performance

#### Training Dataset Results

| Metric | Ingredient | Quantity | Unit | Overall |
|--------|------------|----------|------|---------|
| Precision | High | Very High | High | High |
| Recall | High | Very High | High | High |
| F1-Score | High | Very High | High | High |

#### Validation Dataset Results

The model generalizes well to unseen data with consistent performance across all entity types.

#### Confusion Matrix Analysis

**Key Observations:**
- **Quantity** class achieved near-perfect classification due to distinct numerical patterns
- Minor confusion exists between **unit** and **ingredient** classes
- Words like "cloves," "sprig," and "leaves" can function as either units or ingredients depending on context

### 5.3 Error Analysis

#### Common Misclassification Patterns

| True Label | Predicted Label | Example Tokens | Reason |
|------------|-----------------|----------------|--------|
| Unit | Ingredient | cloves, leaves | Dual meaning |
| Ingredient | Unit | sprig, bunch | Context ambiguity |
| Quantity | Ingredient | pair, couple | Word-based quantities |

---

## 6. Key Insights

### 6.1 What Worked Well

1. **Numeric Pattern Detection**: Regex-based features effectively captured quantities in various formats (fractions, decimals, ranges)

2. **Keyword Sets**: Pre-defined unit and quantity keyword sets provided strong baseline classification

3. **Contextual Features**: BOS/EOS markers and neighboring token features improved sequence labeling

4. **Class Weighting**: Inverse frequency weighting helped balance attention across imbalanced classes

### 6.2 Challenges Identified

1. **Ambiguous Terms**: Words like "cloves" (unit: "2 cloves garlic" vs ingredient: "whole cloves") require deeper semantic understanding

2. **Compound Quantities**: Fractional expressions ("1 1/2 cups") caused tokenization issues requiring data cleaning

3. **Rare Units**: Domain-specific units not in keyword sets may be misclassified

---

## 7. Assumptions

The following assumptions were made during this analysis:

1. **Data Quality**: The provided NER labels in the dataset are accurate and consistent
2. **Tokenization**: Simple whitespace-based tokenization is sufficient for recipe text
3. **Language**: All recipe data is in English
4. **Context Independence**: Each recipe is processed independently without cross-recipe dependencies
5. **Label Scheme**: The three-class scheme (ingredient, quantity, unit) is comprehensive for recipe entities
6. **Feature Completeness**: The spaCy model `en_core_web_sm` provides sufficient linguistic features

---

## 8. Conclusion and Recommendations

### 8.1 Summary

This project successfully demonstrated that CRF-based NER with well-engineered features can effectively extract key entities from recipe data. The model achieved high accuracy across all entity types, with particularly strong performance on quantity classification.

### 8.2 Practical Applications

The trained model can be deployed for:
- **Recipe Databases**: Automatic structuring of unstructured recipe text
- **Grocery Apps**: Smart shopping list generation
- **Nutrition Trackers**: Ingredient extraction for nutritional analysis
- **Voice Assistants**: Parsing spoken recipe instructions

### 8.3 Future Improvements

| Improvement | Expected Benefit |
|-------------|------------------|
| Word embeddings (Word2Vec, GloVe) | Better semantic understanding |
| Expanded keyword sets | Improved coverage of cooking terms |
| Character n-grams | Handle spelling variations |
| BiLSTM-CRF or BERT | Capture long-range dependencies |
| Data augmentation | More robust model training |

---

## Appendix: Technical Details

### A1. Libraries Used

- `sklearn-crfsuite`: CRF implementation for sequence labeling
- `spaCy`: NLP features (POS, lemma, dependency)
- `pandas`: Data manipulation
- `matplotlib/seaborn`: Visualization
- `scikit-learn`: Model evaluation metrics

### A2. File Structure

```
├── ingredient_and_quantity.json    # Input data
├── Identifying_Key_Entities_Recipe_v1.ipynb  # Solution notebook



