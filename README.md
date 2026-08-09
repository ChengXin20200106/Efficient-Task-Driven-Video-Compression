# Efficient Task-Driven Video Compression via Lightweight Inter-Frame Compression and Fidelity-Aware Joint Optimization

Official project repository for the manuscript:

**Efficient Task-Driven Video Compression via Lightweight Inter-Frame Compression and Fidelity-Aware Joint Optimization**

**Authors:** Xin Cheng, Lei Yang, Rui Li

School of Information Science and Engineering, Hunan University, China

---

## Overview

This work presents an efficient task-driven video compression framework for human–machine vision. The proposed framework supports both high-fidelity video reconstruction for human perception and semantic analysis for machine vision using a unified bitstream.

To improve the efficiency of inter-frame coding, we redesign four major functional stages of the compression backbone:

- **RDFR-ME**: Residual Deformable Flow Refinement for Motion Estimation
- **FWCR-MC**: Flow-Warp and Contextual Refinement for Motion Compensation
- **MS-RC**: Multi-Scale Residual Coding
- **ILFE**: In-Loop Feature Enhancement

In addition, a fidelity-aware joint optimization strategy is employed to coordinate bitrate, reconstruction fidelity, and downstream task performance. In the joint-training stage, a Charbonnier-based reconstruction objective is used together with adaptive objective weighting.

The complete framework supports four representative downstream machine-vision tasks:

- Action recognition
- Semantic segmentation
- Object detection
- Instance segmentation

---

## Repository Status

> **Current status:** Experimental configuration and evaluation protocol are available.

| Component | Status |
|---|---|
| Experimental configuration | ✅ Available |
| Evaluation protocol | ✅ Available |
| Dataset and task settings | ✅ Available |
| Training protocol | ✅ Available |
| Source code | ⏳ To be released upon acceptance |
| Pretrained compression models | ⏳ To be released upon acceptance |
| Trained downstream models | ⏳ To be released upon acceptance |
| Trained joint models | ⏳ To be released upon acceptance |

The source code and trained checkpoints will be released upon acceptance.

---

## Method

The proposed framework follows a unified-bitstream human–machine video compression paradigm.

For each inter frame, motion information and residual information are compressed and transmitted through the same compression pipeline. The reconstructed representation is subsequently used by both the human-vision and machine-vision branches.

The proposed inter-frame compression backbone consists of four successive stages:

### 1. RDFR-ME: Residual Deformable Flow Refinement for Motion Estimation

RDFR-ME adopts a coarse-to-refined motion-estimation strategy.

A coarse optical-flow field is first estimated using SPyNet. Instead of predicting a complete motion field from scratch, the coarse flow is used as the dominant displacement prior, while a lightweight deformable refinement branch predicts only local residual motion corrections.

The module consists of:

- Motion Feature Extraction (**MFE**)
- Motion Channel Weighting (**MCW**)
- Flow-guided Deformable Residual Refinement (**FDRR**)

---

### 2. FWCR-MC: Flow-Warp and Contextual Refinement for Motion Compensation

FWCR-MC replaces computationally expensive multi-frame feature aggregation with a single-reference prediction–correction structure.

The previous reconstructed feature is first aligned to the current frame using flow-guided feature-domain bilinear warping. A compact contextual refinement branch then predicts residual corrections for local alignment errors.

This design avoids:

- Multi-frame feature concatenation
- Multiple-reference feature storage
- 3D spatio-temporal aggregation

while preserving explicit flow-guided temporal prediction.

---

### 3. MS-RC: Multi-Scale Residual Coding

MS-RC performs scale-aware residual processing around the retained probabilistic latent coding core.

It consists of:

- **MRE**: Multi-Scale Residual Enhancer
- Latent analysis, quantization, entropy coding, and synthesis
- **RRD**: Residual Refinement Decoder

MRE uses complementary receptive fields to enhance the residual representation before latent analysis, while RRD reconstructs and refines the entropy-decoded residual feature.

The design retains a single entropy-coded latent stream and does not introduce additional scale-specific coded branches.

---

### 4. ILFE: In-Loop Feature Enhancement

ILFE directly fuses the intermediate reconstructed feature with the previously reconstructed feature inside the coding loop.

The fused representation is refined using residual processing and channel recalibration. The enhanced feature is used for both:

1. Current-frame reconstruction
2. The reference feature for subsequent predictive coding

Compared with explicit reference-matching approaches, ILFE avoids patch unfolding, similarity matching, matched-feature transfer, and feature reorganization.

---

## Task-Driven Joint Optimization

Training is organized into three stages.

### Stage I: Compression Backbone Pretraining

The video compression backbone is pretrained using a conventional rate–distortion objective:

\[
\mathcal{L}_{RD} = R + \lambda D
\]

where:

- \(R\) denotes bitrate
- \(D\) denotes reconstruction distortion
- \(\lambda\) controls the rate–distortion trade-off

Both MSE-oriented and MS-SSIM-oriented compression models are trained.

---

### Stage II: Downstream Task Pretraining

The downstream task networks are independently pretrained to obtain task-aware initialization.

The evaluated tasks and models are:

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

\[
\mathcal{L}_{joint}
=
\frac{1}{2s_r^2}\mathcal{L}_{RD}
+
\log(1+s_r^2)
+
\frac{1}{2s_t^2}\mathcal{L}_{task}
+
\log(1+s_t^2),
\]

where \(s_r\) and \(s_t\) are learnable scale parameters associated with the rate–distortion and downstream task objectives, respectively.

During Stage III, the reconstruction distortion is modeled using the Charbonnier loss:

\[
\mathcal{L}_{Charb}(\hat{x},x)
=
\frac{1}{N}
\sum_p
\sqrt{
(\hat{x}_p-x_p)^2+\epsilon^2
}.
\]

The corresponding rate–distortion objective is:

\[
\mathcal{L}_{RD}
=
R+\lambda_{charb}\mathcal{L}_{Charb}.
\]

The Charbonnier formulation provides a bounded gradient response to relatively large reconstruction errors and is used to provide a more robust reconstruction constraint during joint human–machine optimization.

---

# Experimental Configuration

## Hardware

All experiments are conducted on:

- **GPU:** NVIDIA RTX 3090, 24 GB
- **CPU:** Intel Xeon Platinum 8358P @ 2.60 GHz

Unless otherwise specified, the reported experiments use a single GPU.

---

## Compression Training Dataset

### Vimeo-90K

The proposed video compression backbone is pretrained on **Vimeo-90K**.

Training configuration:

- Number of training clips: **91,701**
- Random crop size: **256 × 256**

Vimeo-90K is used for compression-backbone pretraining and is not used as a downstream evaluation dataset.

---

## Rate–Distortion Evaluation Datasets

The compression backbone is evaluated on the following standard video compression datasets:

### HEVC Test Sequences

- Class B: 1920 × 1080
- Class C: 832 × 480
- Class D: 416 × 240
- Class E: 1280 × 720

### UVG

Resolution:

- 1920 × 1080

### MCL-JCV

Resolution:

- 1920 × 1080

These datasets are used to evaluate the generalization of the compression backbone across different resolutions and video content characteristics.

---

## Downstream Task Evaluation

Four representative machine-vision tasks are considered.

### Action Recognition

- **Dataset:** UCF101
- **Network:** R3D
- **Evaluation metric:** Top-1 Accuracy

### Semantic Segmentation

- **Dataset:** Cityscapes
- **Network:** DeepLabV3+
- **Evaluation metric:** mIoU

### Object Detection

- **Dataset:** Cityscapes
- **Network:** Faster R-CNN with FPN
- **Evaluation metric:** Box mAP

### Instance Segmentation

- **Dataset:** Cityscapes
- **Network:** Mask R-CNN with FPN
- **Evaluation metric:** Mask mAP

---

# Compression Training Settings

Four rate points are trained for the proposed video compression backbone.

## PSNR-Oriented Models

For MSE-based rate–distortion optimization:

| Rate Point | λ |
|---|---:|
| 1 | 512 |
| 2 | 1024 |
| 3 | 2048 |
| 4 | 4096 |

---

## MS-SSIM-Oriented Models

For MS-SSIM-based optimization:

| Rate Point | λ |
|---|---:|
| 1 | 16 |
| 2 | 32 |
| 3 | 64 |
| 4 | 128 |

---

## Compression Backbone Optimizer

**Optimizer:** Adam

Training schedule:

| Epochs | Learning Rate |
|---|---:|
| 1–30 | \(1\times10^{-4}\) |
| 31–40 | \(5\times10^{-5}\) |
| 41–50 | \(1\times10^{-5}\) |

Total:

- **50 epochs**

---

# Downstream Task Training Settings

The downstream task models are pretrained independently before joint optimization.

**Optimizer:** Adam

**Learning rate:**

\[
1\times10^{-5}
\]

**Training epochs:**

- **30 epochs**

---

# Joint Training Settings

After independently pretraining the compression backbone and downstream task network, the complete framework is jointly optimized.

**Optimizer:** SGD

**Learning rate:**

\[
1\times10^{-5}
\]

**Training epochs:**

- **50 epochs**

For the Charbonnier reconstruction loss:

\[
\epsilon = 1\times10^{-3}.
\]

The four rate points use:

\[
\lambda_{charb}
=
[4096, 2048, 1024, 512].
\]

The same values are kept fixed across all downstream machine-vision tasks.

The adaptive objective scale parameters are initialized as:

\[
s_r = 1.0,\qquad s_t = 1.0.
\]

---

# GOP Configuration

The evaluation GOP settings are:

| Dataset | GOP Size |
|---|---:|
| HEVC Classes B/C/D/E | 10 |
| UVG | 12 |
| MCL-JCV | 12 |
| UCF101 | 12 |
| Cityscapes | 12 |

The first frame or image of each GOP is encoded as an intra frame using:

**`cheng2020anchor` from CompressAI**

The remaining samples in the GOP are encoded using the evaluated inter-frame video compression backbone.

---

# Evaluation Protocol

## Bitrate

Coding rate is reported in **bits per pixel (BPP)**.

The total bitrate includes the encoded:

- Motion information
- Residual information

---

## Reconstruction Quality

Reconstruction fidelity is evaluated using:

### PSNR

Peak Signal-to-Noise Ratio is used to measure pixel-level reconstruction fidelity.

### MS-SSIM

Multi-Scale Structural Similarity is used to measure structural reconstruction quality.

---

## Rate–Distortion Evaluation

Rate–distortion performance is evaluated using:

- **BD-PSNR**
- **BD-MS-SSIM**
- **BD-BR / BD-Rate**

The Bjøntegaard metrics summarize the performance difference across multiple bitrate operating points.

For the reported BD-metric comparisons, **x265 with the `veryslow` preset is used as the anchor**.

Lower BD-Rate indicates better bitrate efficiency at equivalent reconstruction quality.

---

## Rate–Task Evaluation

Downstream task performance is evaluated at different coding rates.

The following task metrics are reported:

| Task | Metric |
|---|---|
| Action recognition | Top-1 Accuracy |
| Semantic segmentation | mIoU |
| Object detection | Box mAP |
| Instance segmentation | Mask mAP |

Rate–task performance is additionally summarized using BD-Task and the corresponding task-oriented BD-Rate.

---

## Computational Complexity

Backbone-level computational efficiency is evaluated using:

- Number of parameters
- FLOPs
- Encoding runtime
- Decoding runtime

The complexity comparison is performed for the **video compression backbone only** on **1080p videos**.

The runtime experiments are conducted on:

- Intel Xeon Platinum 8358P @ 2.60 GHz
- NVIDIA RTX 3090

---

# Compared Methods

## Compression Backbone Comparison

The proposed compression backbone is compared with representative video compression methods, including:

- x265 (`veryslow`)
- HM-16.20
- VTM-13.20
- DVC
- DVCPro
- FVC
- SPME (FVC*)
- DMVC
- HDCVC
- STFE-VC (RES)

For complexity analysis, additional available results such as DeepSVC are included where applicable.

---

## Task-Driven Human–Machine Comparison

For end-to-end human–machine video coding, the proposed method is primarily compared with:

- **TDVC**

To separate the contribution of compression-backbone redesign from task-driven joint optimization, four settings are reported:

- **VCB (TDVC)** — TDVC compression backbone without task-driven joint optimization
- **TDVC** — complete TDVC framework
- **VCB (Ours)** — proposed compression backbone without Stage-III task-driven joint optimization
- **Ours** — complete proposed task-driven framework

---

# Main Results

## Compression Performance

Using x265 (`veryslow`) as the anchor, the proposed video compression backbone achieves the following average results over HEVC Classes B/C/D/E, UVG, and MCL-JCV.

### PSNR-Oriented Evaluation

- **BD-PSNR:** +1.4345 dB
- **BD-Rate:** −36.41%

### MS-SSIM-Oriented Evaluation

- **BD-MS-SSIM:** +0.0140
- **BD-Rate:** −62.24%

---

## Computational Efficiency

For 1080p video compression, the proposed backbone requires:

| Metric | Ours |
|---|---:|
| FLOPs | 10.73 T |
| Parameters | 24.88 M |
| Encoding time | 0.37 s |
| Decoding time | 0.28 s |

Compared with the TDVC compression backbone under the same experimental conditions, the proposed backbone reduces:

- **FLOPs:** 32.0%
- **Parameters:** 5.2%
- **Encoding time:** 31.5%
- **Decoding time:** 31.7%

---

## End-to-End Human–Machine Performance

Across the four downstream tasks, compared with TDVC, the complete proposed framework achieves average improvements of:

- **BD-PSNR:** +0.809 dB
- **BD-MS-SSIM:** +0.0049
- **BD-Task:** +0.537 percentage points

and provides additional bitrate savings of:

- **12.10 percentage points** at equivalent PSNR
- **23.83 percentage points** at equivalent MS-SSIM
- **4.69 percentage points** at equivalent task performance

---

# Ablation Studies

## Structural Modules

The contributions of the four proposed inter-frame coding stages are evaluated using a stage-replacement protocol:

- RDFR-ME
- FWCR-MC
- MS-RC
- ILFE

Each proposed stage is individually replaced with its corresponding component from the TDVC compression backbone while the remaining proposed stages are retained.

The complete model achieves:

- **24.88 M parameters**
- **BD-Rate (PSNR): −41.59% on UVG**
- **BD-PSNR: +1.3848 dB**
- **BD-Rate (MS-SSIM): −51.25%**
- **BD-MS-SSIM: +0.0088**

These experiments evaluate the contribution of each complete functional stage rather than attempting to isolate every individual convolution, residual block, or attention operation.

---

## Reconstruction Loss

The effect of replacing MSE with Charbonnier loss during Stage-III joint optimization is evaluated while keeping the compression backbone and adaptive weighting strategy fixed.

### Low-Rate Setting

| Loss | BPP | PSNR | MS-SSIM | Top-1 |
|---|---:|---:|---:|---:|
| MSE | 0.0465 | 32.14 | 0.9874 | 82.95% |
| Charbonnier | 0.0460 | 32.10 | 0.9896 | 83.80% |

### High-Rate Setting

| Loss | BPP | PSNR | MS-SSIM | Top-1 |
|---|---:|---:|---:|---:|
| MSE | 0.1699 | 37.82 | 0.9962 | 85.90% |
| Charbonnier | 0.1690 | 37.80 | 0.9983 | 86.50% |

The results indicate that the Charbonnier reconstruction objective provides a more favorable empirical balance among bitrate efficiency, structural reconstruction fidelity, and downstream action-recognition performance under the evaluated joint-training setting.

---

# Planned Code Release

The source code and trained checkpoints are currently being prepared for public release.

Upon acceptance, this repository will be updated with:

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
