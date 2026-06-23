# Esophageal Radiotherapy Response Prediction

本项目利用 CT 影像、放疗剂量分布、食管掩膜及外部手工特征，预测食管癌放射治疗后的不良反应（RE）。包含数据预处理和模型训练两大部分。

## 环境依赖

- Python 3.8+
- PyTorch 1.12+
- MONAI, NiBabel, pandas, openpyxl, scikit‑learn, matplotlib, tqdm, scipy

```bash
pip install torch monai nibabel pandas openpyxl scikit-learn matplotlib tqdm scipy
```
# 数据预处理

运行 preprocess.py 将原始 NIfTI 数据转换为统一的 numpy 格式，并进行重采样、CT 标准化、掩膜二值化。

```bash
python preprocess.py \
    --raw_root /path/to/raw_cases \
    --split_dir /path/to/splits \
    --dataset_code MyDataset \
    --save_root /path/to/processed \
    --label_xlsx /path/to/labels.xlsx \
    --target_spacing 1.0 1.0 3.0
```

#  参数说明

- raw_root：每个子文件夹为一个病例，内含 ct.nii.gz、dose.nii.gz、eso.nii.gz（脚本会自动检测文件名）。

- split_dir：包含 train.txt、val.txt、test.txt，每行一个病例文件夹名。若不提供，全部数据视为测试集。

- label_xlsx：Excel 文件，必须有 patientname 和 RE 两列。

- save_root & dataset_code：处理后的数据保存在 {save_root}/{dataset_code}/train|val|test/病例ID/
