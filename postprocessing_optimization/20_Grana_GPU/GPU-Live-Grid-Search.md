---
type: experiment-node
branch: GPU
decision: keep
tags:
  - gpu-optimisation
  - live-simulation
  - grid-search
  - multi-camera
---

# GPU-Live-Grid-Search

## Uloga node-a

Ovaj node lokalizuje live/grid-search rezultate unutar GPU grane, jer je najbolji live-feed kandidat bio `gpu_nms_fullres_two_process`.

## Šta je grid-search merio

Grid-search je testirao postprocessing varijante i software architecture parametre pod istim 10-camera live scenarijom. Glavne metrike nisu bile samo raw FPS, već:

- aggregate output FPS,
- average E2E latency,
- P95 E2E latency,
- average postprocess time,
- queue infer-to-post pressure,
- uticaj worker topologije,
- backpressure i throttle ponašanje.

## Najbolji low-latency setup

```text
variant = gpu_nms_fullres_two_process
buffer_mode = latest
workers = I1/P3
backpressure = soft
max_pending_age_ms = 300
target_output_fps_per_camera = 2
```

| Metric | Value |
|---|---:|
| Aggregate FPS | 14.15 |
| Avg E2E latency | 185.88 ms |
| P95 E2E latency | 263.03 ms |
| Avg postprocess | 113.84 ms |
| Queue infer to post | 7.43 ms |

## Balanced higher-output setup

```text
variant = gpu_nms_fullres_two_process
workers = I1/P5
backpressure = soft
target_output_fps_per_camera = 3
```

| Metric | Value |
|---|---:|
| Aggregate FPS | 16.84 |
| Avg E2E latency | 226.42 ms |
| P95 E2E latency | 284.85 ms |

## Važan zaključak

CPU optimized varijante mogu da pobede u raw aggregate FPS-u, ali po live-feed kriterijumu GPU fullres NMS ima bolji latency i tail latency. Za monitoring sistem, svežina output-a je važnija od obrade svakog frame-a.

## Decision

Za live multi-camera režim koristiti GPU fullres porodicu sa latest buffering i soft backpressure.
