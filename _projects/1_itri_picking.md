---
layout: page
title: Autonomous Warehouse Picking
description: Full robotic pick-and-place pipeline — SAM2 + dual RealSense + MoveIt 2, ~70% end-to-end success rate
img: assets/img/projects/picking_thumb.jpg
importance: 1
category: ITRI Internship
---

**Feb 2026 – Jul 2026 · Industrial Technology Research Institute (ITRI)**

A fully autonomous pick-and-place system for warehouse logistics on an **Elephant Robotics Pro 630** 6-DOF arm.

## System Architecture

The system operates across three layers:

**Perception** — A world-mounted RealSense D435i detects box poses via ArUco markers and builds a live Octomap collision scene from point clouds. A wrist-mounted RealSense runs SAM2 segmentation to find the optimal suction point on each item.

**Planning** — MoveIt 2 + RRTConnect plans collision-free trajectories. A custom **Yaw-sweep IK** pre-filter generates 25 candidate end-effector poses (±180° in 15° steps), selects the closest to the current joint config, and eliminates large-joint-rotation failures that caused planning timeouts.

**Execution** — `sync_plan.py` drives the real arm via ElephantRobot API.

## Results

- ~70% end-to-end success rate, full pick cycle under 100 seconds
- Live digital twin: ArUco → box collision models auto-generated in MoveIt scene
- Yaw-sweep IK reduced planning timeouts significantly vs. naive IK

## Demo

<div class="row">
  <div class="col-sm-6 mt-3">
    <video width="100%" controls muted playsinline>
      <source src="/assets/video/warehouse_aruco.mp4" type="video/mp4">
    </video>
    <div class="caption">ArUco detection + suction point selection</div>
  </div>
  <div class="col-sm-6 mt-3">
    <video width="100%" controls muted playsinline>
      <source src="/assets/video/warehouse_rviz.mp4" type="video/mp4">
    </video>
    <div class="caption">MoveIt 2 planning + live Octomap scene</div>
  </div>
</div>

## Stack

`ROS 2 Humble` · `MoveIt 2` · `SAM2` · `Intel RealSense D435i` · `Octomap` · `ArUco` · `ElephantRobot API`
