# PointArena Solution: Molmo2-8B + Segmentation Hint

A solution for PointArena point-grounding evaluation, built on top of the original PointArena project.  
This repo contains the modified evaluator (`molmo_evaluator_withseg.py`) and result.

## Table of Contents
- [1. Overview](#1-overview)
- [2. Repository Scope](#2-repository-scope)
- [3. Environment](#3-environment)
- [4. Reproduction (From Original PointArena Clone)](#4-reproduction-from-original-pointarena-clone)
- [5. Method](#5-method)
- [6. Evaluation Protocol](#6-evaluation-protocol)
- [7. Results](#7-results)

## 1. Overview
This solution modifies the Molmo evaluator to improve pointing robustness by adding a segmentation-based visual hint image and stronger coordinate parsing logic.

Core idea:
- Input to Molmo includes both the original image and a Mask2Former-generated segmentation visualization.
- The model output is parsed with multi-stage fallback rules for point extraction.
- Success is determined by mask hit (and exact count matching for counting tasks).

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

### 3.2 Python Dependencies
Install with:

```bash
pip install -r requirements.txt
```

`requirements.txt` includes key packages such as:
- `torch==2.6.0+cu118`
- `transformers==4.57.1`
- `accelerate==0.27.2`
- `pillow`, `numpy`, `python-dotenv`, etc.

### 3.3 Hardware (Recommended)
- GPU: >= 24 GB VRAM recommended for Molmo2-8B full precision
- CPU RAM: >= 32 GB
- Multi-GPU is supported by this evaluator (`device_map="auto"`)
- If single-GPU OOM occurs, code includes a 4-bit quantization fallback path

## 4. Reproduction (From Original PointArena Clone)

### Step 1. Clone original PointArena

```bash
git clone <POINTARENA_ORIGINAL_REPO_URL>
cd pointarena
```

### Step 2. Prepare PointArena data and model assets
Follow the original PointArena instructions to ensure the following are available at repository root:
- `data.json`
- `images/`
- `masks/`
- `pixmo_metadata.csv`
- `models/Molmo2-8B` (if using local Molmo weights)
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

Windows PowerShell example:

```powershell
Copy-Item "<YOUR_SOLUTION_REPO_PATH>\molmo_evaluator_withseg.py" ".\molmo_evaluator_withseg.py"
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

PowerShell:

```powershell
$env:SAVED_MODELS_DIR = ".\\models"
```

### Step 7. Run evaluation

```bash
python molmo_evaluator_withseg.py --model Molmo2-8B --type molmo --no-resume
```

Resume from existing partial results:

```bash
python molmo_evaluator_withseg.py --model Molmo2-8B --type molmo --resume
```

### Step 8. Check outputs
- Main result JSON:
  - `static_results/results_molmo_allenai_Molmo2-8B_simple_prompt.json`
- Point overlay visualizations:
  - `point_on_mask/`

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

## 6. Evaluation Protocol
- Dataset iterator reads from `data.json`
- Image file is resolved from `images/<category>/<image_filename>`
- Mask file is loaded from `masks/<mask_filename>`
- Intermediate results are auto-saved every 100 processed samples
- Final report includes `total`, `success`, `failure`, and per-sample details

## 7. Results
From `static_results/results_molmo_allenai_Molmo2-8B_simple_prompt.json` in this repo:

- Total: `982`
- Success: `739`
- Failure: `243`
- Success Rate: **75.25%**

Note:
- Reproduced score may vary slightly across hardware, dependency versions, precision/quantization mode, and model checkpoints.


