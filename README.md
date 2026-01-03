# 🧬 AI Model for Predicting Drug Effects Using Single-Cell Transcriptomic Data
대규모 세포–약물 Perturbation 데이터(Tahoe-100M)를 기반으로 약물 처리로 유도되는 유전자 발현 변화(ΔExpression)를 학습하고,
  
- **Forward task**: 약물 + 세포주 → 유전자 발현 변화 예측  
- **Inverse task**: 원하는 발현 변화를 가장 잘 재현하는 약물 **Ranking / Retrieval**
  
을 동시에 다루는 Transformer 기반 딥러닝 연구 프로젝트입니다.
<br/>
<br/>

## ✨ Key Contributions

- **Cell-aware Drug Retrieval**: 동일 약물이라도 세포주에 따라 반응이 달라진다는 점을 명시적으로 모델링
<br/>

- **Dual Perspective Evaluation**: 회귀(ΔExpression 예측) + 랭킹(Retrieval) 지표를 동시에 평가
<br/>

- **Scalable Design**: Parquet 기반 대용량 perturbation 데이터 직접 로딩 및 학습
<br/>

- **Representation Alignment**: 유전자 발현 변화 공간과 약물 SMILES 표현 공간의 정렬 실험
<br/>
<br/>

## 📁 Repository Structure
본 레포지터리의 구조는 다음과 같습니다.
  
```
drug_ranking-main/
├── f_p/
│   └── f_p_smalltargets.ipynb
├── f_r/
│   └── f_r_onalldata_withcellline.ipynb
└── making_data/
    ├── analysis1.ipynb
    ├── tahoe_counts_per_cell_line.csv
    ├── tahoe_counts_per_drug.csv
    └── tahoe_counts_per_drug_cell_line.csv
```
<br/>
<br/>

## 🧪 Dataset & Preprocessing

### Data Source
- **Tahoe-100M** 약물 반응 (Perturbation) 데이터셋 (Parquet 형식)
- 각 샘플은 `(drug, cell line, gene)`에 대한 반응 측정값에 해당합니다.
<br/>

### Baseline Normalization
- 모든 발현값은 **ΔExpression (발현 변화량)**으로 변환되었습니다.
- 기준이 되는 베이스라인은 각 세포주별 **DMSO-treated control**으로 정의합니다.
<br/>

### Imbalance Analysis (`making_data/analysis1.ipynb`)
불균형한 데이터셋을 다음 세 가지 수준에서 구체적으로 분석했습니다.
  
- 약물 수준 (Drug-level)
- 세포주 수준 (Cell-line-level)
- 약물, 세포주 쌍 수준 (Drug, Cell-line pair-level)
  
분석 결과 Long-tail 분포가 관찰되었으며, 이를 기반으로 다음과 같이 적용했습니다.
- 최소 샘플 수 임계값 설정 (Thresholding)
- 안정적인 학습을 위한 쌍 (Pair) 단위 필터링
<br/>
<br/>

## 🧠 Methods

### Problem Formulation

Let:
- \( d \): drug
- \( c \): cell line
- \( x \in \mathbb{R}^G \): observed gene expression change

We aim to learn a model \( f(d, c) \rightarrow \hat{x} \) such that:

- \( \hat{x} \approx x \) (forward prediction)
- Drugs can be **ranked** by similarity between \( \hat{x} \) and a query signature

---

### Model Architecture (f_r)

**Input Sequence**
```
[CLS] [DRUG] [CELL] g₁ g₂ ... gₙ
```

- `gᵢ`: gene token embedding + projected ΔExpression value
- `[DRUG]`: SMILES-based drug embedding
- `[CELL]`: pretrained cell line embedding

**Encoder**
- Transformer encoder (Cell2Sentence-style)
- Positional embeddings applied

**Output**
- `CLS` token → MLP head → predicted ΔExpression vector

---

### Fast Prototyping Model (f_p)

- Focused on **small target gene sets**
- Trained with:
  - Target gene matching loss
  - CLIP-like contrastive loss between:
    - Drug-induced expression representation
    - SMILES embedding

This stage is used for:
- Architecture sanity check
- Retrieval metric validation

---

### Loss Functions

#### Forward Regression Loss
- Mean Squared Error (MSE)

#### Ranking / Alignment Loss
- Cosine similarity based loss
- Contrastive / ranking loss (InfoNCE-style)

#### Total Loss (example)
```
L = λ_mse · L_mse + λ_rank · L_rank + λ_align · L_align
```

Warm-up strategy is used where ranking loss weight is gradually increased.
<br/>
<br/>

## 🧪 Experiments

### Experimental Setup

- **Train / Validation / Test split**
  - Per (drug, cell line) pair
- **Evaluation**
  - Global metrics
  - Per-cell-line stratified metrics
  - Per-drug stratified metrics

### Tasks

#### 1) Forward Prediction
- Predict ΔExpression given (drug, cell)

#### 2) Inverse Retrieval
- Given a query ΔExpression signature:
  - Rank candidate drugs by similarity

---

### Metrics

#### Regression
- MSE
- MAE
- Cosine similarity
- Pearson correlation
- Spearman correlation

#### Ranking / Retrieval
- Precision@K
- Recall@K
- NDCG@K
- mAP@K

---

### Key Observations

- Incorporating **cell line tokens** significantly improves ranking stability
- Warm-up before applying ranking loss improves convergence
- Learned gene embeddings form structured manifolds even without pathway supervision

---

## ▶️ How to Run

1. Analyze data imbalance
```
making_data/analysis1.ipynb
```

2. Fast prototyping
```
f_p/f_p_smalltargets.ipynb
```

3. Full retrieval & ranking
```
f_r/f_r_onalldata_withcellline.ipynb
```

---

## 🛠 Requirements

```bash
pip install torch numpy pandas pyarrow scanpy scipy scikit-learn matplotlib tqdm
```

---

## 🧩 Notes

- (Drug, Cell) imbalance is severe → filtering is critical
- SMILES embeddings **must align** with drug metadata ordering
- Current implementation is notebook-based for research flexibility

---

## 🔮 Future Work

- Script-based training pipeline
- Zero-shot cell line generalization
- Pathway-aware evaluation
- Downstream disease signature reversal experiments
