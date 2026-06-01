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

Korišćeni ranking helper je:

```text
balanced_score = avg_e2e_ms
               + 0.5 * p95_e2e_ms
               + 5 * queue_infer_to_post_ms
               - 20 * aggregate_fps
```

Niži skor je bolji. Formula je namerno live-feed orijentisana: ne nagrađuje FPS ako se dobija kroz rast queue-a i stale rezultate.

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

## Common-setup poređenje

Isti setup: `latest`, `I1/P5`, `soft`, `max_pending_age_ms=300`, `target_output_fps_per_camera=3`.

| Variant | Aggregate FPS | Avg E2E | P95 E2E | Avg postprocess | Queue infer-to-post | Balanced score |
|---|---:|---:|---:|---:|---:|---:|
| `cpu_k20_fast_two_process` | 16.60 | 273.93 ms | 336.27 ms | 206.63 ms | 5.04 ms | 135.1 |
| `optimized_batch_k20_fast` | 16.66 | 277.07 ms | 348.08 ms | 209.15 ms | 6.66 ms | 151.1 |
| `gpu_nms_fullres_two_process` | 15.91 | 254.54 ms | 338.21 ms | 160.13 ms | 10.11 ms | 155.9 |
| `migraphx_nms` | 11.71 | 630.73 ms | 893.20 ms | 410.78 ms | 163.49 ms | 1660.6 |
| `migraphx_nms_k20` | 11.97 | 636.34 ms | 932.38 ms | 401.87 ms | 175.82 ms | 1742.1 |
| `standard` | 3.95 | 1530.12 ms | 1915.44 ms | 1217.23 ms | 247.85 ms | 3648.0 |

Pod istim setup-om CPU fast varijante su vrlo jake u Balanced Score-u, ali GPU fullres ima niži prosečan E2E i značajno niži postprocess. Kada se posmatraju najbolji latency setup-i kroz celu mrežu eksperimenata, prvih 10 avg-latency rangova pripada `gpu_nms_fullres_two_process`.

## Decision

Za live multi-camera režim koristiti GPU fullres porodicu sa latest buffering i soft backpressure.
