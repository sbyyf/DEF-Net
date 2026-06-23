环境配置
Python 3.8+

PyTorch 1.12+

MONAI、NiBabel、pandas、scikit‑learn、matplotlib、tqdm 等

安装示例：

bash
pip install torch monai nibabel pandas openpyxl scikit-learn matplotlib tqdm scipy
数据预处理
运行 preprocess.py 将原始 NIfTI 数据转换为统一的 numpy 格式，并进行重采样、CT 标准化、掩膜二值化。

bash
python preprocess.py \
    --raw_root /path/to/raw_cases \
    --split_dir /path/to/splits \
    --dataset_code MyDataset \
    --save_root /path/to/processed \
    --label_xlsx /path/to/labels.xlsx \
    --target_spacing 1.0 1.0 3.0
参数说明

raw_root：每个子文件夹为一个病例，内含 ct.nii.gz、dose.nii.gz、eso.nii.gz（脚本会自动检测文件名）。

split_dir：包含 train.txt、val.txt、test.txt，每行一个病例文件夹名。若不提供，全部数据视为测试集。

label_xlsx：Excel 文件，必须有 patientname 和 RE 两列。

save_root & dataset_code：处理后的数据保存在 {save_root}/{dataset_code}/train|val|test/病例ID/。

模型训练
训练脚本支持多种模式：微调 ResNet、仅训练融合分类器、只训练 ResNet。以下示例为推荐的微调模式。

bash
python train.py \
    --gpu "0,1" \
    --data_root /path/to/processed \
    --train_dataset_codes MyDataset \
    --val_dataset_codes MyDataset \
    --val_dataset_codes_2 MyDataset2 \
    --modality both \
    --pretrained_model_path /path/to/pretrained_resnet.pth \
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
常用参数

data_root：预处理步骤的 save_root。

train_dataset_codes / val_dataset_codes：与预处理时的 dataset_code 一致。

pretrained_model_path：预训练 ResNet 权重（可选）。若无，可去掉相关开关从零训练。

external_features_files：外部特征 CSV（第一列为患者 ID，后续列为特征值），可省略并设 --use_external_features False。

batch_size、lr、num_epochs 根据显存和数据量调整。

fine_tune_resnet：仅训练 ResNet 的后几层 + 融合分类器，适合迁移学习。

训练日志、最佳模型及预测结果保存在 work_dir 下，训练过程预览图在 preview_dir。

输出文件
best_model_val1.pth、best_model_val2.pth、best_model_combined.pth：各验证集/综合最优模型权重。

training_results.csv：每个 epoch 的指标记录。

best_*_predictions_epoch_*.csv：最优模型对训练集、验证集的预测详情。

注意事项
训练数据存在类别不平衡，默认使用 Focal Loss 或带 pos_weight 的 BCE。

若没有食管 mask，预处理脚本会自动生成全零掩膜。

外部特征文件需保证患者 ID 与数据集中的病例 ID 一致。

多 GPU 训练时设置 --gpu "0,1,2,3" 并调整 batch_size 为单卡倍数。

引用
本项目基于食管癌放疗多模态数据，更多细节请参考相关论文。
