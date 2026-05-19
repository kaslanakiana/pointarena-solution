# PointArena Competition Solution (Molmo + Detection Hint)

本仓库为 PointArena 比赛提交方案，核心改动文件为 `molmo_evaluator_det.py`。  
方法基于 Molmo2-8B，并引入检测提示图（Mask2Former 生成）辅助 point 预测。

## 1. Competition
- Task: 视觉语言 point grounding（根据文本在图像中输出点坐标）
- Metric: Success Rate（点是否落在目标 mask 内；counting 类别还要求点数匹配）
- Project base: PointArena

## 2. Environment

### 2.1 Software
- OS: Linux / Windows (均可)
- Python: 3.10+（建议 3.10 或 3.11）
- PyTorch: 与 CUDA 匹配版本
- Transformers: 支持 `AutoModelForImageTextToText` 的版本

### 2.2 Install
```bash
conda create -n pointarena_det python=3.10 -y
conda activate pointarena_det
pip install -r requirements.txt
