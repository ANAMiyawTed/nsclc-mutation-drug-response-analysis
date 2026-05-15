
# NSCLC Mutation–Drug Response Analysis

## 概要

本リポジトリでは、非小細胞肺がん（NSCLC）細胞株を対象に、  
**EGFR / KRAS / TP53 の変異と薬剤感受性の関係**を探索的に解析した。

DepMap の細胞株・変異データと GDSC2 の薬剤感受性データを用い、  
まず薬剤–遺伝子ペアの候補を統計的に確認した。  
その後、特に **KRAS 変異と MEK 阻害薬感受性**に着目し、  
変異情報のみを使った簡単な予測モデルでも同様の傾向がみられるかを確認した。


## 解析の流れ

### 1. 変異と薬剤感受性の探索解析
`01_Lung_analysis.ipynb`

- DepMap の Model データから NSCLC 細胞株を抽出
- EGFR / KRAS / TP53 の変異有無を二値化
- GDSC2 の薬剤感受性データと結合
- LN_IC50 のばらつきが大きい薬剤を対象に絞り込み
- Mann–Whitney U test と Cliff's delta を用いて、変異群と野生型群を比較
- 多重検定に対して FDR 補正を実施
- KRAS 変異と MEK 阻害薬（Trametinib, PD0325901）に注目して追加確認

### 2. 変異情報のみを用いたベースライン予測
`02_ML.ipynb`

- EGFR / KRAS / TP53 の変異有無を説明変数として使用
- LN_IC50 を中央値で二分し、高感受性群 / 低感受性群を作成
- Logistic Regression と Random Forest を比較
- 5-fold cross-validation で AUC を評価
- 各変異の寄与を係数・変数重要度から確認

---

## 主な結果

- 30 薬剤 × 3 遺伝子の計 90 ペアを検定したが、  
  **FDR 補正後に有意となったペアは確認されなかった。**

- 一方で、探索的な結果として、
  - **Trametinib × KRAS**
  - **PD0325901 × KRAS**

  では、KRAS 変異群の LN_IC50 が野生型群より低い傾向がみられた。

- 変異情報のみを用いた予測モデルでは、
  - Trametinib: AUC 約 0.59–0.65
  - PD0325901: AUC 約 0.58–0.59

  となり、無作為予測をやや上回る程度の性能であった。

- KRAS 変異は両薬剤で正の係数を示し、  
  統計解析でみられた感受性傾向と同じ方向の結果が確認された。

---

## 使用データ

本解析では以下の公開データを使用した。

- **DepMap**
  - `Model.csv`
  - `OmicsSomaticMutations.csv`
- **GDSC2**
  - `GDSC2_fitted_dose_response_27Oct23.xlsx`

ファイルサイズの都合上、raw data は本リポジトリに含めていない。  
再現する場合は各データを取得し、以下のように配置する。

```text
data/
└── raw/
    ├── Model.csv
    ├── OmicsSomaticMutations.csv
    └── GDSC2_fitted_dose_response_27Oct23.xlsx
```

---

## リポジトリ構成

```
├── notebooks/
│   ├── 01_Lung_analysis.ipynb
│   └─ 02_ML.ipynb
│
├── data/results
|    ├── raw/        # raw data（GitHub には含めていない）
│   ├── processed_data.pkl
│   └── ml_results.pkl
│
├── README.md
└── .gitignore
```

---

## 実行順序

1. raw data を `data/raw/` に配置
2. `01_Lung_analysis.ipynb` を実行
3. `data/results/processed_data.pkl` が保存される
4. `02_ML.ipynb` を実行
5. `data/results/ml_results.pkl` が保存される

---

## 限界点

- 本解析は探索的なものであり、確証的なバイオマーカー検証を目的としたものではない
- FDR 補正後に有意な薬剤–遺伝子ペアは確認されなかった
- 一部の変異群ではサンプル数が限られている
- 細胞株データに基づく解析であり、患者での薬剤反応を直接示すものではない
- 予測モデルでは 3 遺伝子の変異情報のみを使用しており、発現量やコピー数変化などは考慮していない

---

## 今後の課題

今回の解析では変異情報のみを扱ったが、  
今後は発現量、コピー数変化、経路活性などを加え、  
薬剤感受性をより多面的に捉えられるかを検討したい。

