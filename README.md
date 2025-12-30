# 🧬 Drug Ranking & Retrieval (Babayakga)

대규모 **세포–약물 Perturbation 데이터(Tahoe-100M, Parquet)**를 기반으로  
- (Forward) **약물 처리 후 유전자 발현 변화(ΔExpression)를 예측**하고
- (Inverse) **원하는 ΔExpression을 유도하는 약물을 Ranking/Retrieval**하는  
Transformer 기반 딥러닝 실험 레포지토리입니다.
 
> 이 레포는 **파이썬 패키지 형태가 아니라 Notebook 중심**으로 구성되어 있으며,  
> 데이터 불균형 분석(EDA) → 소규모 타깃 기반 Fast Prototyping(`f_p`) → 전체 유전자 대상 Retrieval/Ranking(`f_r`) 흐름으로 진행됩니다.
<br/>
<br/>

## ✨ What’s inside

### 1. 데이터 불균형(imbalance) 분석
`making_data/analysis1.ipynb`에서 아래 3종 통계 CSV를 기반으로 **Long-tail / super-class 분포**를 확인합니다.
  
- `tahoe_counts_per_drug.csv`
- `tahoe_counts_per_cell_line.csv`
- `tahoe_counts_per_drug_cell_line.csv`
  
또한 threshold를 바꿔가며 drug 최소 샘플 수 필터링, (drug, cell_line) pair 최소 샘플 수 필터링을 했을 때  
**커버리지가 얼마나 줄어드는지** 확인하는 코드가 포함되어 있습니다.
<br/>

### 2. `f_p`: Fast Prototyping (Small Targets)
`f_p/f_p_smalltargets.ipynb`  
- Parquet에서 perturbation 샘플을 읽어 **ΔExpression 기반 representation**을 만들고
- 약물의 **타깃 유전자 벡터(멀티라벨/타깃 기반)**를 맞추는 학습을 합니다.
- 또한 약물의 **SMILES 임베딩과 CLIP-like alignment loss**를 함께 사용합니다.
  
**Loss 구성**
- cosine / alignment loss
- BCE loss
- ranking loss
  
**평가 지표**
- Hit@K
- Recall / Precision@K
- mAP@K
- NDCG@K
<br/>

### 3. `f_r`: Fast Retrieval / Ranking (All genes + Cell line token)
`f_r/f_r_onalldata_withcellline.ipynb` : Transformer Encoder로 **[CLS][DRUG][CELL] + gene tokens** 입력을 처리합니다.
  
- `[DRUG]` 토큰 위치에 SMILES 임베딩 주입
- `[CELL]` 토큰 위치에 cell line embedding 주입
- 출력 `CLS` representation → ΔExpression 회귀
  
**평가**
- Regression: MSE, MAE, Pearson/Spearman
- Ranking: Precision@K, Recall@K, NDCG@K
<br/>
<br/>

## 📁 Repository Structure
```text
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

## 🛠 Requirements
```
pip install torch numpy pandas pyarrow scanpy scipy scikit-learn matplotlib tqdm
```
<br/>
<br/>

## ▶️ How to Run
1. 데이터 분포 확인  
   `making_data/analysis1.ipynb`
  
2. Fast Prototyping  
   `f_p/f_p_smalltargets.ipynb`
  
3. Full Retrieval / Ranking  
   `f_r/f_r_onalldata_withcellline.ipynb`
<br/>
<br/>

## 📊 Metrics
- Regression: MSE, MAE, Cosine, Pearson, Spearman
- Ranking: Precision@K, Recall@K, NDCG@K, mAP@K
<br/>
<br/>

## 🧩 Notes
- Tahoe-100M은 (drug, cell) imbalance가 매우 큼
- SMILES embedding은 drug_metadata row 정렬과 반드시 일치해야 함
- f_r 모델은 warmup 이후 ranking loss 적용
<br/>
<br/>
