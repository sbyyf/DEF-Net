# 胸部放疗放射性食管炎预测

本项目利用 CT 影像、放疗剂量分布、食管掩膜及外部手工特征，预测胸部肿瘤放射治疗后放射性食管炎（RE）的发生风险。包含数据预处理和模型训练两大部分。

## 环境依赖

- Python 3.8+
- PyTorch 1.12+
- MONAI, NiBabel, pandas, openpyxl, scikit‑learn, matplotlib, tqdm, scipy

```bash
pip install torch monai nibabel pandas openpyxl scikit-learn matplotlib tqdm scipy
```

## 数据预处理

将原始 NIfTI 数据转换为统一的 numpy 格式，并进行重采样、CT 标准化、掩膜二值化。

```bash
python preprocess.py \
    --raw_root /path/to/raw_cases \
    --split_dir /path/to/splits \
    --dataset_code MyDataset \
    --save_root /path/to/processed \
    --label_xlsx /path/to/labels.xlsx \
    --target_spacing 1.0 1.0 3.0
```

**参数说明**  
- `raw_root`：每个子文件夹为一个病例，内含 CT、剂量、eso 文件（脚本自动检测文件名）。  
- `split_dir`：包含 `train.txt`、`val.txt`、`test.txt`，每行一个病例文件夹名；不提供则全归测试集。  
- `label_xlsx`：包含 `patientname` 和 `RE` 列的 Excel 文件。  
- `save_root` & `dataset_code`：处理结果保存在 `{save_root}/{dataset_code}/train|val|test/病例ID/`。

## 模型训练

训练脚本支持多种模式：微调 ResNet、仅训练融合分类器、只训练 ResNet。推荐使用微调模式：

```bash
python train.py \
    --gpu "0,1" \
    --data_root /path/to/processed \
    --train_dataset_codes MyDataset \
    --val_dataset_codes MyDataset \
    --val_dataset_codes_2 MyDataset2 \
    --modality both \
    --pretrained_model_path /path/to/pretrained.pth \
    --load_resnet_weights --load_vit_weights \
    --fine_tune_resnet \
    --unfreeze_layers layer3_layer4 \
    --unfreeze_classifier --unfreeze_bn \
    --use_external_features \
    --external_features_files /path/to/features.csv \
    --loss_type focal --focal_alpha 0.22 --focal_gamma 1.0 \
    --bce_pos_weight 3.145 --label_smoothing 0.01 \
    --lr 5e-4 --batch_size 4 --num_epochs 40 \
    --amp --freeze_vit \
    --work_dir ./work_dir --preview_dir ./previews
```

**关键参数**  
- `data_root`：预处理输出的 `save_root`。  
- `train_dataset_codes` / `val_dataset_codes`：与预处理时的 `dataset_code` 一致。  
- `pretrained_model_path`：预训练权重（可选）；若无，去掉 `--load_resnet_weights` 等从头训练。  
- `external_features_files`：手工特征 CSV（第一列为患者 ID，后续列为特征值）；不需要可设 `--use_external_features False`。  
- 根据显存调整 `batch_size`、`lr` 和 `num_epochs`。

## 输出文件

训练在 `work_dir` 下生成：
- `best_model_val1.pth`、`best_model_val2.pth`、`best_model_combined.pth` —— 各验证集/综合最优模型  
- `training_results.csv` —— 每 epoch 的指标  
- `best_*_predictions_epoch_*.csv` —— 最优模型的详细预测  

训练过程预览图保存在 `preview_dir` 中。

## 注意事项

- 存在类别不平衡，默认使用 Focal Loss 或带 `pos_weight` 的 BCE。  
- 无食管 mask 时预处理会自动生成全零掩膜。  
- 外部特征文件的患者 ID 需与数据集中病例 ID 一致。  
- 多 GPU 训练请用 `--gpu "0,1,2,3"` 并保持 `batch_size` 为单卡的整数倍。

## 引用

本项目基于胸部放疗多模态数据预测放射性食管炎，更多细节请参考相关论文。
```
