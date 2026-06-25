# 混凝土缺陷检测实验代码

本目录包含论文中混凝土缺陷检测实验所需的代码和数据集，包括 YOLOv8s 基线模型、本文提出的 FDEM、TAAM、CSAFM 模块、消融实验配置、训练脚本、评估脚本和数据集划分。

## 环境配置

建议使用 Python 3.10。完整复现 300 epoch 训练实验建议使用 CUDA GPU。

```bash
pip install -r requirements.txt
```

本项目基于 Ultralytics YOLO。公开预训练权重不包含在本目录中，例如 `yolov8s.pt`、YOLOv5/YOLOv10/YOLO11 和 RT-DETR 相关权重。若机器可以联网，Ultralytics 会自动下载这些权重；若离线复现，需要提前将对应 `.pt` 权重文件放到项目根目录，或在脚本参数中改为本地权重路径。

## 数据集

数据集已经包含在本目录中：

```text
data/construction_defect/images/train
data/construction_defect/images/val
data/construction_defect/images/test
data/construction_defect/labels/train
data/construction_defect/labels/val
data/construction_defect/labels/test
```

数据集配置文件为 `data/data.yaml`。当前划分包含 4,583 张训练图像、229 张验证图像和 228 张测试图像，图像与标签数量一一对应。

## 快速检查

正式训练前建议先运行：

```bash
python scripts/test_modules.py
python scripts/test_csafm_identity.py
```

这两个脚本用于检查自定义模块是否可以被正确构建，以及 CSAFM 的初始化行为是否符合预期。

## 主要训练命令

训练本文完整模型：

```bash
python scripts/train_custom.py --model models/yolov8s_custom.yaml --name full_model --device 0 --epochs 300 --batch 16 --patience 50
```

训练 YOLOv8s 基线模型：

```bash
python scripts/train_custom.py --model models/yolov8s_baseline.yaml --name baseline --device 0 --epochs 300 --batch 16 --patience 50
```

运行完整消融实验和对比实验队列：

```bash
python scripts/run_ablation_queue.py --device 0 --epochs 300 --batch 16 --patience 50
```

训练输出会保存在：

```text
runs/detect/<实验名称>
```

## 评估与结果汇总

在测试集上评估训练好的模型：

```bash
python scripts/eval.py --weights runs/detect/full_model/weights/best.pt --split test
```

生成消融实验汇总表：

```bash
python scripts/aggregate_results.py --use-best
```

输出文件会保存在：

```text
results/ablation_summary.csv
results/ablation_summary.md
results/ablation_summary.tex
```

生成逐类别分析：

```bash
python scripts/per_class_analysis.py
```

测试推理速度：

```bash
python scripts/benchmark_fps.py --device 0 --precision both
```

生成预测可视化和对比图：

```bash
python scripts/visualize.py --weights runs/detect/full_model/weights/best.pt --baseline-weights runs/detect/baseline/weights/best.pt --mode all
```

## 可复现性说明

本目录已经包含复现论文主实验所需的核心代码、模型配置和数据集划分。对方拿到本目录后，在正确安装依赖并准备好预训练权重的情况下，可以从头训练并复现实验流程。

需要注意的是，最终数值可能会受到 GPU 型号、CUDA/cuDNN 版本、PyTorch 版本、Ultralytics 版本和随机性的影响，因此不同机器上的 mAP 可能存在小幅波动。若要求严格复现论文中的具体数值，建议使用与原实验一致的依赖版本和硬件环境。

本目录默认不包含已经训练完成的实验输出。如果希望对方不重新训练、直接核对论文中的表格和指标，还需要额外提供：

```text
runs/detect/<实验名称>/weights/best.pt
runs/detect/<实验名称>/results.csv
results/ablation_summary.*
results/fps_benchmark.csv
```
