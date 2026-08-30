---
created: 2026-08-30
purpose: Generative AI and Machine Learning fundamentals for Infosys interview
---
# GenAI & ML Fundamentals - Infosys Interview

## 1. Machine Learning Basics

### Types of ML

| Type | Description | Example | Your Project |
|------|-------------|---------|--------------|
| **Supervised** | Labeled data (input -> output) | Spam detection, price prediction | Case routing, Meal prediction |
| **Unsupervised** | No labels, find patterns | Clustering, anomaly detection | User grouping |
| **Reinforcement** | Learn via reward/penalty | Game playing, robotics | Not used |

### Supervised Learning Subtypes

| Subtype | Target | Algorithm | Example |
|---------|--------|-----------|---------|
| **Classification** | Discrete class | Logistic Regression, SVM, Random Forest | Case type prediction |
| **Regression** | Continuous value | Linear Regression, Ridge, Lasso | Price prediction |

---

## 2. Key Algorithms

### Linear Regression
**Use**: Predicting continuous values (house prices, sales).

```python
from sklearn.linear_model import LinearRegression

model = LinearRegression()
model.fit(X_train, y_train)
predictions = model.predict(X_test)
```

**Assumptions**: Linear relationship, independent errors, homoscedasticity.

### Logistic Regression
**Use**: Binary classification (spam/not spam, pass/fail).

```python
from sklearn.linear_model import LogisticRegression

model = LogisticRegression()
model.fit(X_train, y_train)
probabilities = model.predict_proba(X_test)  # [not_spam_prob, spam_prob]
```

**Interview Answer**: Despite the name, it's a classification algorithm. Uses sigmoid function to map outputs to probabilities between 0 and 1.

### Decision Tree
**Use**: Classification and regression with interpretable rules.

```python
from sklearn.tree import DecisionTreeClassifier

model = DecisionTreeClassifier(max_depth=5)
model.fit(X_train, y_train)
```

**Pros**: Interpretable, handles mixed data types.
**Cons**: Prone to overfitting, unstable.

### Random Forest
**Use**: Ensemble of decision trees, robust classification.

```python
from sklearn.ensemble import RandomForestClassifier

model = RandomForestClassifier(n_estimators=100, random_state=42)
model.fit(X_train, y_train)
predictions = model.predict(X_test)
```

**How it works**: Creates multiple decision trees on random subsets of data and features. Final prediction = majority vote (classification) or average (regression).

**Your Context**: "In PGPulse, I used Random Forest for meal prediction. It handles the mix of categorical (day of week, weather) and numerical (historical attendance) features well."

**Interview Answer**: "Random Forest reduces overfitting by creating many trees on random subsets and averaging their predictions. It's like asking 100 experts and taking the majority vote."

### K-Nearest Neighbors (KNN)
**Use**: Simple classification based on similarity.

```python
from sklearn.neighbors import KNeighborsClassifier

model = KNeighborsClassifier(n_neighbors=5)
model.fit(X_train, y_train)
```

**How it works**: Finds K closest training examples and assigns the most common class.

### Support Vector Machine (SVM)
**Use**: Classification with maximum margin separation.

```python
from sklearn.svm import SVC

model = SVC(kernel='rbf')
model.fit(X_train, y_train)
```

---

## 3. Model Evaluation

### Classification Metrics

| Metric | Formula | When to Use |
|--------|---------|-------------|
| **Accuracy** | (TP+TN)/(TP+TN+FP+FN) | Balanced classes |
| **Precision** | TP/(TP+FP) | Cost of false positive is high (spam filter) |
| **Recall** | TP/(TP+FN) | Cost of false negative is high (cancer detection) |
| **F1 Score** | 2*(Precision*Recall)/(Precision+Recall) | Imbalanced classes |

### Confusion Matrix
```
                Predicted
                Positive  Negative
Actual Positive   TP        FN
Actual Negative   FP        TN
```

### Precision vs Recall Trade-off
- **High Precision**: Few false positives (not many legit emails go to spam)
- **High Recall**: Few false negatives (not many spam emails go to inbox)

**Your Context**: "For case routing in LawPrix, I'd optimize for precision - you don't want to route a case to the wrong lawyer. For face recognition in PGPulse, recall matters more - you want to correctly identify all residents."

### Regression Metrics
| Metric | Description |
|--------|-------------|
| **MAE** | Mean Absolute Error (average of absolute differences) |
| **MSE** | Mean Squared Error (penalizes large errors more) |
| **RMSE** | Square root of MSE (same unit as target) |
| **R-squared** | Proportion of variance explained (0-1) |

---

## 4. Overfitting & Underfitting

### Overfitting (High Variance)
- Model memorizes training data, fails on new data
- Signs: Training accuracy >> Test accuracy
- **Solutions**: Regularization, more data, simpler model, cross-validation, dropout

### Underfitting (High Bias)
- Model too simple, can't capture patterns
- Signs: Both training and test accuracy are low
- **Solutions**: More complex model, more features, less regularization

### Bias-Variance Tradeoff
```
Total Error = Bias^2 + Variance + Irreducible Error

Simple Model: High Bias, Low Variance
Complex Model: Low Bias, High Variance
```

**Interview Answer**: "Overfitting is like memorizing answers for an exam without understanding - you ace practice tests but fail the real exam. Underfitting is like not studying at all."

---

## 5. Feature Engineering

### Techniques
| Technique | Description | Example |
|-----------|-------------|---------|
| **Normalization** | Scale to [0,1] range | Min-Max scaling |
| **Standardization** | Mean=0, Std=1 | Z-score normalization |
| **Encoding** | Convert categories to numbers | One-hot encoding, Label encoding |
| **Feature Selection** | Remove irrelevant features | Correlation analysis, PCA |
| **Feature Creation** | Create new features from existing | Extract day from date |

### Why Feature Engineering Matters
"Garbage in, garbage out." Good features improve model performance more than algorithm tuning.

---

## 6. Generative AI & LLMs

### What is a Large Language Model (LLM)?
Neural network trained on massive text data to understand and generate human-like text. Examples: GPT, Claude, LLaMA, Nemotron.

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Tokenization** | Breaking text into tokens (words/subwords) |
| **Embedding** | Converting tokens to dense vector representations |
| **Attention** | Mechanism to focus on relevant parts of input |
| **Context Window** | Maximum tokens model can process |
| **Temperature** | Controls randomness (0 = deterministic, 1 = creative) |

### Transformer Architecture
```
Input -> Tokenizer -> Embedding -> Transformer Layers -> Output
                                    |
                                    v
                              Self-Attention
                              (how each word relates to every other word)
```

### RAG (Retrieval-Augmented Generation)
**Use**: Ground LLM responses in external, up-to-date knowledge.

```
User Query -> Retrieve relevant documents from vector DB
           -> Combine query + documents as context
           -> LLM generates answer grounded in retrieved info
```

**Your Context**: "In LawPrix, I use RAG for case assessment. Case documents get embedded and stored. When assessing a new case, I retrieve similar cases and pass them as context to the LLM, so summaries are grounded in actual case material."

**Interview Answer**: "RAG solves the hallucination problem by giving the LLM specific documents to reference. Instead of generating from general knowledge, it generates from retrieved evidence."

### Embeddings
- Dense vector representations of text (e.g., 512-dimensional)
- Similar texts have similar embeddings
- Used for: semantic search, clustering, recommendation

**Your Context**: "In PGPulse, InsightFace generates 512-dimensional facial embeddings. I compare embeddings using cosine similarity to match faces. In LawPrix, I embed case documents for RAG retrieval."

### Cosine Similarity vs Euclidean Distance
```python
# Cosine Similarity (used in your projects)
cosine_sim = dot(a, b) / (norm(a) * norm(b))
# Range: -1 to 1 (1 = identical, 0 = orthogonal, -1 = opposite)
# Good for: high-dimensional text/face embeddings

# Euclidean Distance
euclidean_dist = sqrt(sum((a - b)^2))
# Range: 0 to infinity (0 = identical)
# Good for: lower-dimensional data
```

**Why Cosine Similarity for Embeddings**: Embeddings are high-dimensional. Cosine measures angle (direction) not magnitude, making it more robust to variations in embedding magnitude.

### Prompt Engineering
**Techniques**:
- **Zero-shot**: No examples, just the task description
- **Few-shot**: Include examples in the prompt
- **Chain-of-thought**: Ask model to explain reasoning step by step
- **Structured output**: Request specific JSON format

**Your Context**: "In LawPrix, I use structured prompting - asking the LLM to return case assessment as JSON with specific fields: summary, key_facts, urgency_level, and confidence_score."

### Fine-tuning vs RAG
| Aspect | Fine-tuning | RAG |
|--------|-------------|-----|
| When to use | domain-specific language/style | up-to-date or proprietary knowledge |
| Cost | High (GPU hours, data prep) | Low (vector DB + API calls) |
| Updates | Retrain required | Update documents |
| Hallucination | Can still hallucinate | Grounded in retrieved docs |

---

## 7. Scikit-learn Pipeline

```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import cross_val_score

# Create pipeline
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('classifier', RandomForestClassifier(n_estimators=100))
])

# Cross-validation
scores = cross_val_score(pipeline, X, y, cv=5)
print(f"Accuracy: {scores.mean():.2f} (+/- {scores.std():.2f})")
```

**Your Context**: "In LawPrix, the case routing is a scikit-learn pipeline that first scales features and then applies a classifier. This ensures consistent preprocessing during training and inference."

---

## 8. ML Interview Quick Answers

**Q: What is the difference between classification and regression?**
A: Classification predicts discrete classes (spam/not spam). Regression predicts continuous values (house prices). Random Forest can do both.

**Q: What is cross-validation?**
A: Technique to assess model performance by splitting data into K folds, training on K-1 folds, testing on remaining fold, and rotating. Gives more reliable performance estimate than single train/test split.

**Q: What is regularization?**
A: Technique to prevent overfitting by adding penalty term to loss function. L1 (Lasso) adds absolute value of coefficients, L2 (Ridge) adds squared coefficients. Both shrink coefficients toward zero.

**Q: How do you handle imbalanced datasets?**
A: Techniques: oversampling minority class (SMOTE), undersampling majority class, class weights, different metrics (F1 instead of accuracy), ensemble methods.

**Q: What is a confusion matrix?**
A: Table showing True Positives, True Negatives, False Positives, False Negatives. Used to calculate precision, recall, F1-score. Reveals where model makes errors.

**Q: What is the bias-variance tradeoff?**
A: High bias = underfitting (too simple). High variance = overfitting (too complex). Goal is finding the sweet spot that minimizes total error.

---

> **For Infosys**: They may ask basic ML concepts (supervised vs unsupervised, overfitting). Your projects give you an edge - be ready to explain how you applied these concepts in LawPrix (RAG, case routing) and PGPulse (face recognition, meal prediction).
