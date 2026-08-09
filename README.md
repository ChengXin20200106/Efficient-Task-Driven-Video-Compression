# Efficient Task-Driven Video Compression via Lightweight Inter-Frame Compression and Fidelity-Aware Joint Optimization

Official project repository for the manuscript:

> **Efficient Task-Driven Video Compression via Lightweight Inter-Frame Compression and Fidelity-Aware Joint Optimization**

**Xin Cheng, Lei Yang, Rui Li**

School of Information Science and Engineering, Hunan University, China

---

## Overview

This work presents an efficient task-driven video compression framework for **human–machine vision**, supporting both high-fidelity signal reconstruction for human perception and semantic analysis for machine vision through a **unified bitstream**.

To improve the efficiency of inter-frame coding, we redesign four major functional stages of the compression backbone:

- **RDFR-ME**: Residual Deformable Flow Refinement for Motion Estimation
- **FWCR-MC**: Flow-Warp and Contextual Refinement for Motion Compensation
- **MS-RC**: Multi-Scale Residual Coding
- **ILFE**: In-Loop Feature Enhancement

In addition, we employ a fidelity-aware joint optimization strategy to coordinate bitrate, signal fidelity, and downstream task performance. During joint training, a Charbonnier-based reconstruction objective is used together with uncertainty-based adaptive objective weighting.

The complete framework is evaluated on four representative downstream machine-vision tasks:

- Action recognition
- Semantic segmentation
- Object detection
- Instance segmentation

---

## Repository Status

> **Current status:** The experimental configuration and evaluation protocol are provided in this repository.

| Component | Status |
|---|:---:|
| Experimental configuration | ✅ Available |
| Evaluation protocol | ✅ Available |
| Dataset and task settings | ✅ Available |
| Training protocol | ✅ Available |
| Source code | ⏳ Upon acceptance |
| Trained checkpoints | ⏳ Upon acceptance |

**The source code and trained checkpoints will be released upon acceptance.**

---

## Method Overview

The proposed framework follows a unified-bitstream human–machine video compression paradigm.

For each inter frame, motion and residual information are encoded and transmitted through the same compression pipeline. The reconstructed frame is subsequently delivered to both the human-vision and machine-vision backends.

The inter-frame compression backbone consists of four successive stages.

### 1. RDFR-ME: Residual Deformable Flow Refinement for Motion Estimation

RDFR-ME adopts a coarse-to-refined motion-estimation strategy.

SPyNet first estimates a coarse optical-flow field. Instead of predicting a complete motion field from scratch, the coarse flow serves as the dominant displacement prior, while a lightweight deformable refinement branch predicts only local residual motion corrections.

RDFR-ME contains three main components:

- **MFE**: Motion Feature Extraction
- **MCW**: Motion Channel Weighting
- **FDRR**: Flow-Guided Deformable Residual Refinement

---

### 2. FWCR-MC: Flow-Warp and Contextual Refinement for Motion Compensation

FWCR-MC replaces computationally intensive multi-frame feature aggregation with a compact single-reference prediction–correction structure.

The previous reconstructed feature is first aligned using flow-guided feature-domain bilinear warping. A lightweight contextual refinement branch then predicts residual corrections for the local prediction errors remaining after warping.

This design avoids:

- Multi-frame feature concatenation
- Multiple-reference feature storage
- 3D spatio-temporal aggregation

while retaining an explicit flow-guided temporal prediction path.

---

### 3. MS-RC: Multi-Scale Residual Coding

MS-RC introduces scale-aware residual processing around the retained entropy-coding core.

It consists of:

- **MRE**: Multi-Scale Residual Enhancer
- Latent analysis, quantization, entropy coding, and synthesis
- **RRD**: Residual Refinement Decoder

MRE enhances the residual representation before latent analysis using complementary receptive fields, while RRD reconstructs and refines the entropy-decoded residual feature.

The design retains a **single entropy-coded latent stream** and does not introduce additional scale-specific coded branches.

---

### 4. ILFE: In-Loop Feature Enhancement

ILFE directly fuses the intermediate reconstructed feature with the previously reconstructed feature inside the coding loop.

The fused representation is refined using residual processing and channel recalibration. The enhanced feature is then used for:

1. Current-frame reconstruction
2. The reference feature for subsequent predictive coding

Compared with explicit reference-matching approaches, ILFE avoids patch unfolding, explicit similarity matching, matched-feature transfer, and feature reorganization.

---

## Task-Driven Optimization

The training procedure contains three stages.

### Stage I: Compression Backbone Pretraining

The video compression backbone is pretrained using a conventional rate–distortion objective:

$$
\mathcal{L}_{RD}=R+\lambda D
$$

where $R$ denotes the coding rate, $D$ denotes the reconstruction distortion, and $\lambda$ controls the rate–distortion trade-off.

Both PSNR-oriented and MS-SSIM-oriented models are trained.

---

### Stage II: Downstream Task Pretraining

The downstream task networks are independently pretrained to obtain task-aware initialization.

| Task | Dataset | Network | Metric |
|---|---|---|---|
| Action recognition | UCF101 | R3D | Top-1 Accuracy |
| Semantic segmentation | Cityscapes | DeepLabV3+ | mIoU |
| Object detection | Cityscapes | Faster R-CNN + FPN | Box mAP |
| Instance segmentation | Cityscapes | Mask R-CNN + FPN | Mask mAP |

---

### Stage III: Joint Optimization

The pretrained compression backbone and downstream task network are jointly optimized.

The joint objective follows an uncertainty-based adaptive weighting formulation:

$$
\mathcal{L}_{joint}
=
\frac{1}{2s_r^2}\mathcal{L}_{RD}
+
\log(1+s_r^2)
+
\frac{1}{2s_t^2}\mathcal{L}_{task}
+
\log(1+s_t^2)
$$

where $s_r$ and $s_t$ are learnable scale parameters associated with the rate–distortion objective and downstream task objective, respectively.

During Stage III, the reconstruction distortion is modeled using the Charbonnier loss:

$$
\mathcal{L}_{Charb}(\hat{x},x)
=
\frac{1}{N}
\sum_p
\sqrt{(\hat{x}_p-x_p)^2+\epsilon^2}
$$

The corresponding rate–distortion objective is:

$$
\mathcal{L}_{RD}
=
R+\lambda_{charb}\mathcal{L}_{Charb}
$$

The Charbonnier formulation reduces the excessive influence of relatively large reconstruction errors on shared-parameter updates and provides a more robust reconstruction constraint during joint optimization.

---

# Experimental Configuration

## Hardware

All experiments are conducted using:

- **GPU:** NVIDIA RTX 3090, 24 GB
- **CPU:** Intel Xeon Platinum 8358P @ 2.60 GHz

Unless otherwise specified, the experiments are performed using a single GPU.

---

## Compression Training Dataset

### Vimeo-90K

The proposed video compression backbone is pretrained on **Vimeo-90K**.

- Training clips: **91,701**
- Random crop size: **256 × 256**

Vimeo-90K is used for compression-backbone pretraining.

---

## Rate–Distortion Evaluation Datasets

The compression backbone is evaluated on three standard video compression benchmarks.

### HEVC Test Sequences

| Class | Resolution |
|---|---:|
| Class B | 1920 × 1080 |
| Class C | 832 × 480 |
| Class D | 416 × 240 |
| Class E | 1280 × 720 |

### UVG

- Resolution: **1920 × 1080**

### MCL-JCV

- Resolution: **1920 × 1080**

These datasets cover different resolutions and content characteristics and are used to evaluate the generalization ability of the compression backbone.

---

## Downstream Task Evaluation

Four representative machine-vision tasks are evaluated.

### Action Recognition

- **Dataset:** UCF101
- **Network:** R3D
- **Metric:** Top-1 Accuracy

### Semantic Segmentation

- **Dataset:** Cityscapes
- **Network:** DeepLabV3+
- **Metric:** mIoU

### Object Detection

- **Dataset:** Cityscapes
- **Network:** Faster R-CNN with FPN
- **Metric:** Box mAP

### Instance Segmentation

- **Dataset:** Cityscapes
- **Network:** Mask R-CNN with FPN
- **Metric:** Mask mAP

---

# Training Configuration

## Compression Backbone

Four rate points are trained for the proposed compression backbone.

### PSNR-Oriented Models

For MSE-based rate–distortion optimization:

| Rate Point | $\lambda$ |
|:---:|---:|
| 1 | 512 |
| 2 | 1024 |
| 3 | 2048 |
| 4 | 4096 |

### MS-SSIM-Oriented Models

For MS-SSIM-based rate–distortion optimization:

| Rate Point | $\lambda$ |
|:---:|---:|
| 1 | 16 |
| 2 | 32 |
| 3 | 64 |
| 4 | 128 |

### Optimization

- **Optimizer:** Adam
- **Total epochs:** 50

| Epochs | Learning Rate |
|---|---:|
| 1–30 | $1\times10^{-4}$ |
| 31–40 | $5\times10^{-5}$ |
| 41–50 | $1\times10^{-5}$ |

---

## Downstream Task Networks

The downstream task networks are independently pretrained before joint optimization.

- **Optimizer:** Adam
- **Learning rate:** $1\times10^{-5}$
- **Training epochs:** 30

---

## Joint Training

After pretraining the compression backbone and downstream task network independently, the complete framework is jointly optimized.

- **Optimizer:** SGD
- **Learning rate:** $1\times10^{-5}$
- **Training epochs:** 50

For the Charbonnier reconstruction loss:

$$
\epsilon=1\times10^{-3}
$$

The four rate points use:

$$
\lambda_{charb}=[4096,\ 2048,\ 1024,\ 512]
$$

These values are kept fixed across all downstream tasks.

The learnable scale parameters are initialized as:

$$
s_r=1.0,\qquad s_t=1.0
$$

---

# GOP Configuration

The GOP settings used during evaluation are:

| Dataset | GOP Size |
|---|:---:|
| HEVC Classes B/C/D/E | 10 |
| UVG | 12 |
| MCL-JCV | 12 |
| UCF101 | 12 |
| Cityscapes | 12 |

The first frame or image of each GOP is encoded as an intra frame using **`cheng2020anchor` from CompressAI**.

The remaining samples are encoded using the evaluated inter-frame video compression backbone.

---

# Evaluation Protocol

## Coding Rate

The coding rate is reported in **bits per pixel (BPP)**.

The total coding rate consists of the bits required for:

- Motion information
- Residual information

---

## Reconstruction Quality

Signal fidelity is evaluated using:

### PSNR

Peak Signal-to-Noise Ratio measures pixel-level reconstruction fidelity.

### MS-SSIM

Multi-Scale Structural Similarity measures structural reconstruction quality.

---

## Rate–Distortion Evaluation

Rate–distortion performance is evaluated using:

- **BD-PSNR**
- **BD-MS-SSIM**
- **BD-BR / BD-Rate**

For the reported BD-metric comparisons, **x265 with the `veryslow` preset** is used as the anchor.

Lower BD-Rate indicates greater bitrate savings at comparable quality, while higher BD-PSNR and BD-MS-SSIM indicate better reconstruction quality at comparable bitrate.

---

## Rate–Task Evaluation

Task performance is evaluated at different coding rates measured in BPP.

The following metrics are used:

| Task | Metric |
|---|---|
| Action recognition | Top-1 Accuracy |
| Semantic segmentation | mIoU |
| Object detection | Box mAP |
| Instance segmentation | Mask mAP |

Rate–task performance is additionally summarized using **BD-Task** and the corresponding **task-oriented BD-Rate**.

---

## Computational Complexity

Backbone-level computational efficiency is evaluated using:

- Parameter count
- FLOPs
- Encoding time
- Decoding time

The complexity comparison is performed for the **video compression backbone only** on **1080p videos**.

Runtime experiments use the same hardware platform:

- Intel Xeon Platinum 8358P @ 2.60 GHz
- NVIDIA RTX 3090

---

# Compared Methods

## Video Compression Backbone

The compression backbone is compared with representative conventional and learned video compression methods, including:

- x265 (`veryslow`)
- HM-16.20
- VTM-13.20
- DVC
- DVCPro
- FVC
- SPME (FVC*)
- TDVC
- DMVC
- HDCVC
- STFE-VC (RES)

For computational efficiency analysis, additional methods with available complexity results are also included.

---

## Task-Driven Human–Machine Compression

For end-to-end human–machine video compression, the proposed method is primarily compared with **TDVC**.

To separate the effect of compression-backbone redesign from task-driven joint optimization, four configurations are reported:

- **VCB (TDVC)**: TDVC compression backbone without task-driven joint optimization
- **TDVC**: complete TDVC framework
- **VCB (Ours)**: proposed compression backbone without Stage-III task-driven joint optimization
- **Ours**: complete proposed framework

---

# Main Results

## Rate–Distortion Performance

Using x265 (`veryslow`) as the anchor, the proposed compression backbone achieves the following average performance over HEVC Classes B/C/D/E, UVG, and MCL-JCV.

### PSNR-Oriented Evaluation

| Metric | Result |
|---|---:|
| BD-PSNR | **+1.4345 dB** |
| BD-Rate | **−36.41%** |

### MS-SSIM-Oriented Evaluation

| Metric | Result |
|---|---:|
| BD-MS-SSIM | **+0.0140** |
| BD-Rate | **−62.24%** |

---

## Computational Efficiency

For 1080p video compression, the proposed backbone requires:

| Metric | Ours |
|---|---:|
| FLOPs | **10.73 T** |
| Parameters | **24.88 M** |
| Encoding time | **0.37 s** |
| Decoding time | **0.28 s** |

Compared with the TDVC compression backbone under the same experimental conditions, the proposed backbone reduces:

| Metric | Reduction |
|---|---:|
| FLOPs | **32.0%** |
| Parameters | **5.2%** |
| Encoding time | **31.5%** |
| Decoding time | **31.7%** |

---

## End-to-End Human–Machine Performance

Averaged over the four downstream tasks, compared with TDVC, the complete proposed framework improves:

| Metric | Improvement |
|---|---:|
| BD-PSNR | **+0.809 dB** |
| BD-MS-SSIM | **+0.0049** |
| BD-Task | **+0.537 pp** |

It additionally provides bitrate savings of:

| Equivalent Performance | Additional Bitrate Saving |
|---|---:|
| PSNR | **12.10 percentage points** |
| MS-SSIM | **23.83 percentage points** |
| Task performance | **4.69 percentage points** |

---

# Ablation Studies

## Structural Modules

The contributions of the four proposed inter-frame coding stages are evaluated using a stage-replacement protocol:

- RDFR-ME
- FWCR-MC
- MS-RC
- ILFE

Each proposed stage is individually replaced with its corresponding component from the TDVC compression backbone, while the other three proposed stages remain unchanged.

On UVG, the complete model achieves:

| Metric | Full Model |
|---|---:|
| BD-Rate (PSNR) | **−41.59%** |
| BD-PSNR | **+1.3848 dB** |
| BD-Rate (MS-SSIM) | **−51.25%** |
| BD-MS-SSIM | **+0.0088** |
| Parameters | **24.88 M** |

The stage-replacement experiments evaluate the contribution of each complete functional stage rather than attempting to isolate every individual convolution, attention block, or residual block.

---

## Reconstruction Objective

The effect of replacing MSE with the Charbonnier loss during Stage-III joint optimization is evaluated while keeping the compression backbone and adaptive objective-weighting strategy fixed.

### Low-Rate Regime

| Loss | BPP | PSNR (dB) | MS-SSIM | Top-1 (%) |
|---|---:|---:|---:|---:|
| MSE | 0.0465 | 32.14 | 0.9874 | 82.95 |
| Charbonnier | **0.0460** | 32.10 | **0.9896** | **83.80** |

### High-Rate Regime

| Loss | BPP | PSNR (dB) | MS-SSIM | Top-1 (%) |
|---|---:|---:|---:|---:|
| MSE | 0.1699 | 37.82 | 0.9962 | 85.90 |
| Charbonnier | **0.1690** | 37.80 | **0.9983** | **86.50** |

These results indicate that the Charbonnier reconstruction objective provides a more favorable empirical trade-off among bitrate efficiency, structural reconstruction quality, and downstream task performance in the evaluated joint-training setting.

---

# Reproducibility

To facilitate reproducibility, the current repository documents:

- Training datasets
- Evaluation datasets
- Downstream task networks
- Evaluation metrics
- Rate points
- Rate–distortion coefficients
- Optimizers
- Learning-rate schedules
- Training epochs
- GOP settings
- Intra-frame coding configuration
- Joint-training configuration
- Hardware platform
- Evaluation scope

The source code, executable scripts, and trained checkpoints will be added upon acceptance.

---

# Planned Code Release

Upon acceptance, this repository will be updated with the implementation and trained models.

The planned release will include:

- Training code
- Evaluation code
- Configuration files
- Pretrained compression models
- Trained task-driven models
- Dataset preparation instructions
- Reproduction scripts

A possible repository structure after the full release is:

```text
Efficient-Task-Driven-Video-Compression/
├── README.md
├── configs/
│   ├── compression/
│   └── downstream_tasks/
├── models/
│   ├── rdfr_me/
│   ├── fwcr_mc/
│   ├── ms_rc/
│   ├── ilfe/
│   └── codec/
├── tasks/
│   ├── action_recognition/
│   ├── semantic_segmentation/
│   ├── object_detection/
│   └── instance_segmentation/
├── scripts/
│   ├── train/
│   └── eval/
├── checkpoints/
├── requirements.txt
└── LICENSE
```

---

# Dataset Availability

The datasets used in this work are publicly available from their respective official sources:

- Vimeo-90K
- HEVC Classes B/C/D/E
- UVG
- MCL-JCV
- UCF101
- Cityscapes

These datasets are **not redistributed** in this repository.

Detailed dataset organization and preprocessing instructions will be provided together with the source-code release.

---

# Citation

If you find this work useful for your research, please consider citing our paper.

The final bibliographic information will be updated after publication.

```bibtex
@article{cheng2026efficient,
  title  = {Efficient Task-Driven Video Compression via Lightweight Inter-Frame Compression and Fidelity-Aware Joint Optimization},
  author = {Cheng, Xin and Yang, Lei and Li, Rui},
  year   = {2026}
}
```

# License

The license for the source-code release will be specified when the implementation is made publicly available.
