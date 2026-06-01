---
type: shared-theme
tags:
  - shared-theme
  - metrics
  - validation
graph_rule: intentionally-not-linked-from-main-branches
---

# Performance-Metrics-Reference

Ovaj fajl je zajednička tema i namerno nije wikilinkovan iz glavnih grana.

## Accuracy metrics

COCO-style validacija koristi:

- AP,
- AP50,
- AP75,
- AR.

Standard reference tuple je:

| Metric | Value |
|---|---:|
| AP | 0.3995 |
| AP50 | 0.6706 |
| AP75 | 0.4020 |
| AR | 0.4603 |

## Runtime metrics

Speed/live validacija koristi:

- mean latency,
- p95 latency,
- FPS,
- pipeline FPS,
- stage breakdown,
- queue pressure,
- dropped/processed frame counts.

Glavne skripte koje proizvode ove metrike su `accuracy_validation.py` za COCO-style AP validaciju i `speed_validation.py` za CCTV speed/stage breakdown. Osnovna referentna implementacija candidate extraction-a i grouping-a nalazi se u `modules/keypoints.py`, posebno u funkcijama `extract_keypoints`, `connections_nms` i `group_keypoints`.

## Final validation table

| Variant | AP | COCO e2e avg | CCTV e2e avg | CCTV FPS / pipeline FPS | Decision |
|---|---:|---:|---:|---:|---|
| `standard` | 0.3995 | 82.8 ms | 230.6 ms | 4.34 FPS | correctness reference |
| `optimized_batch_k10` | 0.3777 | 69.5 ms | 163.8 ms | 6.11 FPS | dominated by K20 |
| `optimized_batch_k20` | 0.3995 | 68.9 ms | 166.0 ms | 6.02 FPS | safe CPU improvement |
| `optimized_batch_k20_fast` | 0.3995 | 39.2 ms | 73.0 ms | 13.70 FPS | best single-process CPU fallback |
| `cpu_k20_fast_two_process` | 0.3995 | 35.9 ms | 76.4 ms | 14.24 pipeline FPS | CPU runtime fallback |
| `gpu_nms_fullres_two_process` | 0.3995 | 28.1 ms | 47.5 ms | 17.41 pipeline FPS | best accuracy-preserving runtime |
| `migraphx_nms` | 0.4061 | 37.6 ms | 84.3 ms | 11.87 FPS | compiled NMS feasibility |
| `migraphx_nms_k20` | 0.3995 | 31.9 ms | 84.7 ms | 11.80 FPS | accuracy-preserving compiled NMS |
| `fast_no_resize` | 0.2184 | 20.8 ms | 16.2 ms | 61.66 FPS | degraded-accuracy CPU boundary |
| `lowres_cpu_group` | 0.2184 | 20.2 ms | 15.2 ms | 65.58 FPS | degraded-accuracy CPU shortcut |
| `gpu_nms_lowres_two_process` | 0.2479 | 19.3 ms | 17.9 ms | 32.11 pipeline FPS | degraded-accuracy runtime |

## Accuracy-preserving lane

Varijante koje čuvaju referentnu preciznost:

| Variant | AP | Role |
|---|---:|---|
| optimized_batch_k20 | 0.3995 | safe CPU improvement |
| optimized_batch_k20_fast | 0.3995 | best single-process CPU fallback |
| cpu_k20_fast_two_process | 0.3995 | CPU runtime fallback |
| gpu_nms_fullres_two_process | 0.3995 | best practical runtime |
| migraphx_nms_k20 | 0.3995 | compiled NMS proof |

## Degraded-accuracy throughput lane

Varijante koje su brze, ali gube AP:

| Variant | AP | Role |
|---|---:|---|
| fast_no_resize | 0.2184 | CPU speed boundary |
| lowres_cpu_group | 0.2184 | fastest CPU degraded shortcut |
| gpu_nms_lowres_two_process | 0.2479 | fastest runtime, degraded accuracy |

## Najvažnija interpretacija

Ne postoji jedan “najbolji” rezultat bez cilja. Ako je cilj referentna preciznost, najbolji izbor je GPU full-resolution NMS runtime. Ako je cilj maksimalan throughput uz prihvatljiv gubitak tačnosti, low-resolution varijante čine posebnu lane.

Za live-feed poređenje koristi se i pomoćni **Balanced Score**:

```text
balanced_score = avg_e2e_ms
               + 0.5 * p95_e2e_ms
               + 5 * queue_infer_to_post_ms
               - 20 * aggregate_fps
```

Niži skor je bolji. Formula namerno jače kažnjava prosečnu latenciju, tail latency i infer-to-post queue pressure, dok throughput nagrađuje tek nakon što sistem ostane svež i stabilan.
