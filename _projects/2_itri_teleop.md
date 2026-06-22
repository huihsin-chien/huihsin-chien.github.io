---
layout: page
title: VR Teleoperation for Imitation Learning
description: Meta Quest 2 → 7-DOF dual-arm robot, 97% IK success rate — data collection for downstream imitation learning
img: assets/img/projects/teleop_thumb.jpg
importance: 2
category: ITRI Internship
---

**Feb 2026 – Jul 2026 · Industrial Technology Research Institute (ITRI)**

A bilateral teleoperation interface that lets an operator intuitively drive a **7-DOF dual-arm robot** in real time using **Meta Quest 2** VR controllers — capturing high-quality human demonstrations for downstream imitation learning.

## Technical Details

- **TRAC-IK** solver with workspace retargeting achieves **97% IK success rate**
- **Swing-Twist quaternion decomposition** for full 3-axis end-effector orientation tracking
- **EMA smoothing** on controller input eliminates jitter artifacts in recorded trajectories
- Low-latency streaming: controller pose → IK solve → joint command → robot execution

## Demo

The two videos below show both sides of the teleoperation: the human operator (left) and the robot arm mirroring the movements in real time (right).

<div class="row">
  <div class="col-sm-6 mt-3">
    <video width="100%" controls muted playsinline>
      <source src="/assets/video/teleoperation_human.mp4" type="video/mp4">
    </video>
    <div class="caption">Human operator with Meta Quest 2 controllers</div>
  </div>
  <div class="col-sm-6 mt-3">
    <video width="100%" controls muted playsinline>
      <source src="/assets/video/teleoperation_robot.mp4" type="video/mp4">
    </video>
    <div class="caption">Robot arm following in real time</div>
  </div>
</div>

## Stack

`Meta Quest 2` · `TRAC-IK` · `ROS 2` · `Swing-Twist decomposition` · `EMA filter` · `Python`
