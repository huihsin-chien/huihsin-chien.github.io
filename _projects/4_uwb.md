---
layout: page
title: Multi-UAV Indoor Localization (UWB)
description: Custom UWB PCB + DS-TWR + MDS self-calibration — ±10 cm static accuracy, SAFMC 2025 4th place
img: assets/img/projects/uwb_thumb.jpg
importance: 4
category: Research
---

**Dec 2024 – Jul 2025 · NYCU ICCL Lab**

A low-cost, self-calibrating **UWB indoor localization system** for multi-UAV collaboration, built from scratch — custom PCB, firmware, and a full Python positioning stack. Designed to support the SAFMC 2025 competition scenario: autonomously delivering fire suppression payloads from a supply zone to hotspots, requiring obstacle avoidance and path planning indoors.

## Why UWB?

GPS fails indoors due to building occlusion. IMU and optical flow suffer from drift. Millimeter-wave, Bluetooth, and ultrasound are either expensive or imprecise. UWB offers:

- **Centimeter-level accuracy** (±10 cm static, ≈20 cm dynamic)
- **Strong multipath resistance** via wide spectrum (>500 MHz bandwidth)
- **Low power, low cost** — practical for drone payloads
- **No inter-anchor clock sync required** with DS-TWR

We chose UWB fused with IMU and optical flow as the primary indoor localization stack for our drones.

## Hardware: AmazingUWB

We designed, manufactured, and hand-soldered our own UWB module — **AmazingUWB**:

| Component | Spec |
|-----------|------|
| MCU | ESP32-S3 |
| UWB chip | Qorvo DW1000 |
| Antenna | Omnidirectional rod antenna |
| UWB channel | 2 |
| Data rate | 6.8 Mbps |
| Pulse frequency | 16 MHz |
| Preamble length | 64 |
| Preamble code | 3 |

<div class="row justify-content-center">
  <div class="col-sm-6 mt-3">
    {% include figure.liquid path="assets/img/projects/uwb_board.jpg" title="AmazingUWB module" class="img-fluid rounded z-depth-1" %}
    <div class="caption">AmazingUWB — custom PCB with Qorvo DW1000 + ESP32-S3</div>
  </div>
</div>

## System Architecture

The system supports **4–8 anchors** and **1–9 tags** in 3D space. Coordinates are computed in Cartesian XYZ, then transformed to drone NED (North-East-Down) for Pixhawk flight controller input.

**Software stack:**

```
AmazingUWB (C/C++, Arduino)
  └── DS-TWR firmware (built on F-Army/arduino-dw1000-ng)
      └── UWB Publisher ROS 2 node (Python)
          ├── IQR outlier rejection + median filter
          ├── Anchor coordinate solve (Classical MDS)
          ├── Tag positioning (Multilateration + Nelder-Mead)
          └── Pixhawk serial → Sensor fusion (UWB + IMU + optical flow)
```

## Algorithm 1 — Anchor Self-Calibration (Classical MDS)

On startup, anchor positions are unknown (taping distances >5 m is impractical). Anchors measure pairwise distances among themselves, then **Classical Multidimensional Scaling** recovers 3D coordinates from the distance matrix:

```python
def ClassicalMDS(D, dim):
    n = D.shape[0]
    C = np.eye(n) - np.ones((n, n)) / n
    B = -C.dot(D ** 2).dot(C) / 2
    eigvals, eigvecs = np.linalg.eig(B)
    idx = np.argsort(eigvals)[::-1]
    V = eigvecs[:, idx[:dim]]
    L = np.diag(eigvals[idx[:dim]])
    return np.dot(V, np.sqrt(L))
```

After MDS, coordinates are aligned to the XYZ axes via **Rodrigues' rotation formula** using the known placement of Anchors A, B, C as origin/X-axis/Y-axis references.

**Anchor placement rules:**
- A at origin (0, 0, 0)
- B on the X-axis
- C on the Y-axis (same Z height as A, B)
- D elevated ~80 cm above A–C plane (omnidirectional antenna has a blind spot directly above — D must not be placed directly over A)

## Algorithm 2 — Tag Positioning (Multilateration + Nelder-Mead)

With anchor coordinates known, the tag's position is estimated by minimizing the residual between measured distances and Euclidean distances from each anchor:

$$E(x) = \sum_{i=1}^{n} \left(\|x - C_i\| - r_i\right)^2$$

Initial guess: weighted average of anchor positions (weights inversely proportional to range). Optimization via **Nelder-Mead** with a custom tolerance stopping condition:

```python
def gps_solve(distances_to_station, stations_coordinates,
              initial_guess=None, tol=0.0009):
    def error(x, c, r):
        return sum((np.linalg.norm(x - c[i]) - r[i]) ** 2
                   for i in range(len(c)))
    if initial_guess is None:
        l = len(stations_coordinates)
        S = sum(distances_to_station)
        W = [((l - 1) * S) / (S - w) for w in distances_to_station]
        x0 = sum([W[i] * stations_coordinates[i] for i in range(len(W))])
    else:
        x0 = initial_guess
    return minimize(error, x0,
                    args=(stations_coordinates, distances_to_station),
                    method='Nelder-Mead', tol=tol).x
```

## Data Filtering

Raw UWB ranges are noisy. Before solving tag position, each anchor's readings are filtered:
1. **IQR outlier rejection** — remove readings outside [Q1 − 1.5·IQR, Q3 + 1.5·IQR]
2. **Median** — take median of remaining readings

This balances noise rejection and update rate (critical since computation on a laptop was already a bottleneck).

## Demo

<div class="row">
  <div class="col-sm-12 mt-3">
    <video width="100%" controls muted playsinline>
      <source src="/assets/video/uwb_demo.mp4" type="video/mp4">
    </video>
    <div class="caption">Live UWB ranging and tag positioning demo</div>
  </div>
</div>

## Results

| Metric | Value |
|--------|-------|
| Static positioning error | ≈ ±10 cm |
| Dynamic (walking) average error | ≈ 20 cm |
| Mapped area | 3.6 m × 3.8 m (actual: 3.6 m × 3.72 m) |
| Competition result | 4th place — SAFMC 2025, Multi-UAV D2 category |

Test field was the basement of Engineering Building 5 at NYCU, using Anchors A–D in a ~3.6×3.8 m quadrilateral.

<div class="row">
  <div class="col-sm-6 mt-3">
    {% include figure.liquid path="assets/img/projects/uwb_test_field.jpg" title="UWB test field" class="img-fluid rounded z-depth-1" %}
    <div class="caption">Anchor setup in the NYCU Engineering Building 5 basement — four tripod-mounted AmazingUWB modules define the coordinate frame</div>
  </div>
  <div class="col-sm-6 mt-3">
    {% include figure.liquid path="assets/img/projects/uwb_localization_result.png" title="UWB localization result" class="img-fluid rounded z-depth-1" %}
    <div class="caption">Localization trace of a person walking the perimeter and diagonals of the 3.6 m × 3.72 m test area — the lower-left drift is caused by Z-axis disturbance when picking up / setting down the tag</div>
  </div>
</div>

## Problems & Solutions

| Problem | Solution |
|---------|----------|
| Computation bottleneck (slow data rate) | IQR + median filtering — faster than raw averaging |
| Metal drone frame interfering with antenna | Redesigned bracket v2: horizontal mount, further from frame |
| DS-TWR library used 4-sided ranging (sequential, prone to dropout) | Rewrote firmware for 2-sided TWR; skips non-responding anchors gracefully |
| Anchor ranging dropout causes positioning failure | Any 4+ anchors responding is sufficient; missing anchors are excluded automatically |

## Stack

`ESP32-S3` · `Qorvo DW1000` · `Arduino / C++` · `Python` · `ROS 2` · `DS-TWR` · `Classical MDS` · `Nelder-Mead` · `Pixhawk` · `NumPy · SciPy`
