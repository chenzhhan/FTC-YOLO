The complete code will be made public after the paper is accepted.

# Experimental Code for Concrete Defect Detection

This directory contains the code and dataset required for the concrete defect detection experiments reported in the paper, including the YOLOv8s baseline model, the proposed FDEM, TAAM, and CSAFM modules, ablation experiment configurations, training scripts, evaluation scripts, and dataset splits.

## Environment Setup

Python 3.10 is recommended. A CUDA-enabled GPU is recommended for fully reproducing the 300-epoch training experiments.

```bash
pip install -r requirements.txt
```

This project is based on Ultralytics YOLO. Publicly available pretrained weights are not included in this directory, such as `yolov8s.pt`, YOLOv5/YOLOv10/YOLO11 weights, and RT-DETR-related weights. If the machine has internet access, Ultralytics will automatically download these weights. For offline reproduction, the corresponding `.pt` weight files should be placed in the project root directory in advance, or the script arguments should be modified to use local weight paths.

## Dataset

The dataset is already included in this directory:

```text
data/construction_defect/images/train
data/construction_defect/images/val
data/construction_defect/images/test
data/construction_defect/labels/train
data/construction_defect/labels/val
data/construction_defect/labels/test
```

The dataset configuration file is `data/data.yaml`. The current split contains 4,583 training images, 229 validation images, and 228 test images, with a one-to-one correspondence between images and labels.

## Quick Checks

Before formal training, it is recommended to run:

```bash
python scripts/test_modules.py
python scripts/test_csafm_identity.py
```

These two scripts are used to check whether the custom modules can be correctly constructed and whether the initialization behavior of CSAFM is as expected.

## Main Training Commands

Train the full proposed model:

```bash
python scripts/train_custom.py --model models/yolov8s_custom.yaml --name full_model --device 0 --epochs 300 --batch 16 --patience 50
```

Train the YOLOv8s baseline model:

```bash
python scripts/train_custom.py --model models/yolov8s_baseline.yaml --name baseline --device 0 --epochs 300 --batch 16 --patience 50
```

Run the complete queue of ablation experiments and comparative experiments:

```bash
python scripts/run_ablation_queue.py --device 0 --epochs 300 --batch 16 --patience 50
```

Training outputs will be saved in:

```text
runs/detect/<实验名称>
```

## Evaluation and Result Summary

Evaluate a trained model on the test set:

```bash
python scripts/eval.py --weights runs/detect/full_model/weights/best.pt --split test
```

Generate the ablation experiment summary table:

```bash
python scripts/aggregate_results.py --use-best
```

The output files will be saved in:

```text
results/ablation_summary.csv
results/ablation_summary.md
results/ablation_summary.tex
```

Generate the per-class analysis:

```bash
python scripts/per_class_analysis.py
```

Benchmark the inference speed:

```bash
python scripts/benchmark_fps.py --device 0 --precision both
```

Generate prediction visualizations and comparison figures:

```bash
python scripts/visualize.py --weights runs/detect/full_model/weights/best.pt --baseline-weights runs/detect/baseline/weights/best.pt --mode all
```

## Reproducibility Notes

This directory already contains the core code, model configurations, and dataset splits required to reproduce the main experiments in the paper. After the dependencies are correctly installed and the pretrained weights are prepared, users can train the models from scratch and reproduce the experimental workflow.

It should be noted that the final numerical results may be affected by the GPU model, CUDA/cuDNN version, PyTorch version, Ultralytics version, and randomness. Therefore, the mAP values may show slight fluctuations across different machines. If strict reproduction of the exact numerical results reported in the paper is required, it is recommended to use the same dependency versions and hardware environment as those used in the original experiments.

By default, this directory does not include completed training outputs. If users are expected to directly verify the tables and metrics reported in the paper without retraining the models, the following files should be additionally provided:

```text
runs/detect/<实验名称>/weights/best.pt
runs/detect/<实验名称>/results.csv
results/ablation_summary.*
results/fps_benchmark.csv
```

