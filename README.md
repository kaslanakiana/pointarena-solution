# PointArena Solution: Molmo2-8B + Segmentation Hint

A solution for PointArena point-grounding evaluation.  
This repo contains the modified evaluator (`molmo_evaluator_withseg.py`) and result.

## Table of Contents
- [1. Overview](#1-overview)
- [2. Repository Scope](#2-repository-scope)
- [3. Environment](#3-environment)
- [4. Reproduction (From Original PointArena Clone)](#4-reproduction-from-original-pointarena-clone)
- [5. Method](#5-method)
- [6. Results](#6-results)

## 1. Overview
This solution is built around Molmo2's multi-image capability: we feed both the original image and an additional segmentation-based visual hint image into the model to improve point grounding reliability.

## 2. Repository Scope
This repository is a **solution patch repo**, not a full standalone benchmark package.

Included:
- `molmo_evaluator_withseg.py`: modified evaluator
- `requirements.txt`: tested dependency set
- `static_results/results_molmo_allenai_Molmo2-8B_simple_prompt.json`: evaluation result file

Not included:
- Full PointArena codebase and assets
- Benchmark data (`data.json`, `images/`, `masks/`, `pixmo_metadata.csv`)
- Model weights (`models/Molmo2-8B`, `mask2former-swin-large-coco-panoptic`)

## 3. Environment

### 3.1 Software
- Python 3.10+ (recommended: 3.10 or 3.11)
- CUDA-enabled PyTorch environment for efficient inference
- Linux/Windows supported

### 3.2 Hardware
- GPU: >= 24 GB VRAM recommended for Molmo2-8B full precision

## 4. Reproduction

### Step 1. Clone original PointArena

```bash
git clone <https://github.com/pointarena/pointarena>
cd pointarena
```

### Step 2. Prepare PointArena data and model assets
Follow the original PointArena instructions to ensure the following are available at repository root:
- `data.json`
- `images/`
- `masks/`
- `pixmo_metadata.csv`
- `models/Molmo2-8B` 
- `mask2former-swin-large-coco-panoptic/`

### Step 3. Clone this solution repo
Clone this solution repository anywhere local:

```bash
git clone <YOUR_SOLUTION_REPO_URL>
```

### Step 4. Apply the evaluator patch
Copy solution evaluator into your PointArena root (rename as needed):

```bash
cp <YOUR_SOLUTION_REPO_PATH>/molmo_evaluator_withseg.py ./molmo_evaluator_withseg.py
```

### Step 5. Install dependencies
Inside original `pointarena` root:

```bash
pip install -r <YOUR_SOLUTION_REPO_PATH>/requirements.txt
```

### Step 6. Configure environment variables (optional but recommended)
If using local models, set:

```bash
export SAVED_MODELS_DIR=./models
```

### Step 7. Run evaluation

```bash
python molmo_evaluator_withseg.py --model Molmo2-8B --type molmo
```

## 5. Method

### 5.1 Dual-image prompting
For each sample, evaluator builds a two-image input:
1. Original image
2. Mask2Former panoptic segmentation visualization (`call_mask2former`)

The second image acts as a detection/structure hint for point grounding.

### 5.2 Category-aware behavior
- `counting`: allows multiple points and enforces predicted point count == ground-truth count
- non-counting categories: keeps only one point
- `steerable`: draws provided reference point(s) on image before inference (`pixmo_metadata.csv`)

### 5.3 Robust coordinate extraction
Prediction parsing uses multi-stage fallback:
1. Parse `<points coords="...">` style outputs
2. Parse bracketed coordinate arrays with regex
3. Parse JSON-like structures
4. Last-resort numeric extraction

### 5.4 Success criterion
A sample is successful when:
- predicted point(s) are inside the target binary mask (`is_point_in_mask`), and
- for counting tasks, predicted count matches expected count

## 6. Results
From `static_results/results_molmo_allenai_Molmo2-8B_simple_prompt.json` in this repo:

- Total: `982`
- Success: `739`
- Failure: `243`
- Success Rate: **75.25%**

Note:
- Reproduced score may vary slightly across hardware, dependency versions, precision/quantization mode, and model checkpoints.


