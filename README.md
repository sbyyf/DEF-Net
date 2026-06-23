# Esophageal Radiotherapy Response Prediction

本项目利用 CT 影像、放疗剂量分布、食管掩膜及外部手工特征，预测食管癌放射治疗后的不良反应（RE）。包含数据预处理和模型训练两大部分。

## 环境依赖

- Python 3.8+
- PyTorch 1.12+
- MONAI, NiBabel, pandas, openpyxl, scikit‑learn, matplotlib, tqdm, scipy

```bash
pip install torch monai nibabel pandas openpyxl scikit-learn matplotlib tqdm scipy
