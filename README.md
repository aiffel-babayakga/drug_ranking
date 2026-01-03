# 🧬 AI Model for Predicting Drug Effects Using Single-Cell Transcriptomic Data
본 레포지터리는 대규모 세포–약물 Perturbation 데이터(Tahoe-100M)를 기반으로  
약물 처리로 유도되는 유전자 발현 변화(ΔExpression)를 학습하고,
  
- **Forward task**: 약물 + 세포주 → 유전자 발현 변화 예측  
- **Inverse task**: 원하는 발현 변화를 가장 잘 재현하는 약물 **Ranking / Retrieval**
  
을 동시에 다루는 **Transformer 기반 딥러닝 연구 프로젝트**입니다.
<br/>
<br/>

## ✨ Key Contributions
- **Cell-aware Drug Retrieval**: 동일한 약물이라도 세포주에 따라 반응이 달라진다는 점을 명시적으로 모델링
  
- **Dual Perspective Evaluation**: ΔExpression 회귀 성능과 약물 Retrieval 성능을 동시에 평가
  
- **Scalable Design**: Parquet 기반 대규모 perturbation 데이터를 직접 로딩하여 학습 가능
  
- **Representation Alignment**: 유전자 발현 변화 공간과 약물 SMILES 표현 공간 간 정렬 실험 수행
<br/>
<br/>

## 📁 Repository Structure
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
- **Tahoe-100M** single-cell perturbation dataset (Parquet format)
- 각 샘플은 `(drug, cell line, gene)` 단위의 발현 반응 정보를 포함
<br/>

---

### Baseline Normalization
- 모든 발현값은 **ΔExpression** (발현 변화량)으로 변환
- 기준 베이스라인은 각 세포주별 **DMSO-treated control**
<br/>

---

### Imbalance Analysis (`making_data/analysis1.ipynb`)
데이터 불균형을 다음 세 가지 수준에서 분석합니다.
  
- Drug-level
- Cell-line-level
- Drug–Cell-line pair-level
  
분석 결과, 강한 **Long-tail 분포**가 관찰되었으며 이를 바탕으로:
  
- 최소 샘플 수 기준 임계값 설정
- 학습 안정성을 위한 pair 단위 필터링
  
을 적용했습니다.
<br/>
<br/>

## 🧠 Methods
본 연구는 두 가지 주요 모듈로 구성됩니다.
  
### 1) Forward Prediction (Response Modeling)
- 입력: `(drug, cell line)` 쌍
- 출력: 약물 처리 후 **ΔExpression 벡터**
- Transformer encoder를 사용하여 유전자 토큰 시퀀스를 모델링
- 세포주 정보를 명시적인 토큰으로 주입하여 cell-specific response를 학습
  
### 2) Inverse Retrieval (Drug Ranking)
- 입력: 쿼리 ΔExpression signature
- 후보 약물에 대해 예측된 반응과의 유사도를 계산
- 유사도 기반으로 약물 랭킹 산출
- Retrieval 안정성을 위해 ranking-aware loss를 함께 사용
<br/>
<br/>

## 🧪 Experiments

### Experimental Setup
- **Data Split**
  - 약물–세포주 쌍 기준 Train / Validation / Test 분할
  
- **Evaluation Strategy**
  - 전체(Global) 성능 지표
  - 세포주별 계층화 지표
  - 약물별 계층화 지표
<br/>

---

### Tasks

#### 1) Forward Prediction
- 주어진 약물–세포주 쌍에 대해 유전자 발현 변화 예측
  
#### 2) Inverse Retrieval
- 쿼리 ΔExpression signature에 대해:
  - 예측된 반응과의 유사도 계산
  - 후보 약물 순위 산출
<br/>

---

### Metrics

#### Regression Metrics
- MSE
- MAE
- Cosine Similarity
- Pearson Correlation
- Spearman Correlation
  
#### Ranking / Retrieval Metrics
- Precision@K
- Recall@K
- NDCG@K
- mAP@K
<br/>

---

### Key Observations
- 세포주 토큰을 명시적으로 사용하는 것이 retrieval 안정성을 크게 향상시킴
- 랭킹 손실을 점진적으로 도입하면 학습 수렴이 더 부드러워짐
- 학습된 유전자 임베딩은 명시적 pathway supervision 없이도
  구조화된 다양체(manifold)를 형성함
<br/>
<br/>

## ▶️ How to Run

1. 데이터 불균형 분석
```
making_data/analysis1.ipynb
```

2. Fast prototyping (Inverse task)
```
f_p/f_p_smalltargets.ipynb
```

3. Full forward prediction & retrieval
```
f_r/f_r_onalldata_withcellline.ipynb
```
<br/>
<br/>

## 🛠 Requirements

```
pip install torch numpy pandas pyarrow scanpy scipy scikit-learn matplotlib tqdm
```
<br/>
<br/>

## 🧩 Notes

- (약물, 세포주) 데이터 불균형이 매우 심하므로 필터링이 필수적입니다.
- SMILES 임베딩은 약물 메타데이터의 정렬 순서와 반드시 일치해야 합니다.
<br/>
<br/>
