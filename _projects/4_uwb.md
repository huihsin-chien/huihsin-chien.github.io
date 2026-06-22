---
layout: page
title: Multi-UAV Indoor Localization (UWB)
description: Custom UWB PCB + TWR protocol — <20 cm accuracy in 20×20 m space, SAFMC 2025 4th place
img: assets/img/projects/uwb_thumb.jpg
importance: 4
category: Research
---

**Dec 2024 – Jul 2025 · NYCU ICCL Lab**

A low-cost, self-calibrating **UWB indoor localization system** for multi-UAV collaboration, built from scratch — custom PCB, firmware, and Python positioning stack.

## Hardware: AmazingUWB

We designed and assembled our own UWB module — **AmazingUWB** — using a Qorvo DW1000 chip on an ESP32-S3 microcontroller with an omnidirectional antenna.

<div class="row justify-content-center">
  <div class="col-sm-6 mt-3">
    {% include figure.liquid path="assets/img/projects/uwb_board.jpg" title="AmazingUWB module" class="img-fluid rounded z-depth-1" %}
    <div class="caption">AmazingUWB — custom PCB with DW1000 + ESP32-S3</div>
  </div>
</div>

## Algorithm

- **Double-sided Two-Way Ranging (DS-TWR)** for clock-independent distance measurement
- **Multilateration + least-squares optimization** in Python for 3D position
- **Classical MDS** for self-calibrating anchor coordinate frame
- Achieved **< 20 cm accuracy** in a 20 m × 20 m space

## Demo

<div class="row">
  <div class="col-sm-12 mt-3">
    <video width="100%" controls muted playsinline>
      <source src="/assets/video/uwb_demo.mp4" type="video/mp4">
    </video>
    <div class="caption">UWB ranging demo — live distance measurement</div>
  </div>
</div>

## Results

- 🏆 **4th place** — Singapore Amazing Flying Machine Competition (SAFMC 2025), Multi-UAV Collaboration category
- < 20 cm localization accuracy
- Self-calibrating: no pre-surveyed anchor positions needed

## Stack

`ESP32-S3` · `Qorvo DW1000` · `Arduino` · `Python` · `NumPy` · `DS-TWR` · `Multilateration`
