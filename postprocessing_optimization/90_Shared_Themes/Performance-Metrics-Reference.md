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
