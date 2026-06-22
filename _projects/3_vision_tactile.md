---
layout: page
title: Vision-Tactile Manipulation Policy
description: "🚧 WIP — UMI + dual DIGIT tactile adaptation of ImplicitRDP for fragile object grasping"
img: assets/img/projects/umi_thumb.jpg
importance: 3
category: Independent Research
---

**Jan 2026 – Present · NYCU Systems Engineering Lab (Independent Research)**

> 🚧 **Work in Progress** — currently in data collection phase.

Adapting **ImplicitRDP**'s slow-fast diffusion policy architecture to use **UMI hand-held demonstrations** with synchronized **dual DIGIT tactile sensors** — targeting contact-aware grasping of fragile objects (egg, tofu, test tube).

## The Problem

Fragile object manipulation requires knowing *how hard* you're gripping — visual feedback alone isn't enough. We want to learn a policy that reacts to contact in real time.

## Our Approach

We replace ImplicitRDP's wrist force/torque sensor (Flexiv-specific) with **dual DIGIT tactile sensors** in the fast-path GRU:

```python
# Original: 6D wrench → GRU → temporal conditioning
GRU(input_dim=6,  hidden_dim=512) → Linear(512→768)

# Ours: dual-DIGIT PCA → GRU → temporal conditioning
GRU(input_dim=64, hidden_dim=512) → Linear(512→768)
# input_dim = pca_dim(32) × 2 DIGIT sensors
```

Action space adapted from 19D (Flexiv impedance) → **7D (xyz + axis_angle + gripper)** for position-controlled arms.

## Data Collection Pipeline

UMI clapperboard-analog sync: pressure spike on DIGIT → detect Δt → align GoPro video + tactile streams.

<div class="row">
  <div class="col-sm-12 mt-3">
    <video width="100%" controls muted playsinline>
      <source src="/assets/video/umi_gopro.mp4" type="video/mp4">
    </video>
    <div class="caption">UMI GoPro data collection — third-person view</div>
  </div>
</div>

<div class="row">
  <div class="col-sm-6 mt-3">
    <video width="100%" controls muted playsinline>
      <source src="/assets/video/umi_tactile_left.mp4" type="video/mp4">
    </video>
    <div class="caption">Left DIGIT tactile sensor stream</div>
  </div>
  <div class="col-sm-6 mt-3">
    <video width="100%" controls muted playsinline>
      <source src="/assets/video/umi_tactile_right.mp4" type="video/mp4">
    </video>
    <div class="caption">Right DIGIT tactile sensor stream</div>
  </div>
</div>

## Planned: SparSH Integration

Replace the timm ViT backbone in `TransformerObsEncoder` with a `SparshViTWrapper` for the DIGIT image keys — cross-attention fusion of tactile tokens with visual tokens into the slow-path diffusion transformer.

## Evaluation (planned)

Ablation: **w/ tactile** vs **w/o tactile** on fragile object grasping. Objects: egg · tofu · test tube.

## Stack

`ImplicitRDP` · `UMI` · `DIGIT Tactile Sensor` · `SparSH` · `PyTorch` · `Diffusion Policy` · `ROS 2`
