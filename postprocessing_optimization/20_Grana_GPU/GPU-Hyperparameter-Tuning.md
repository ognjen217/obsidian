---
type: experiment-node
branch: GPU
decision: keep
tags:
  - gpu-optimisation
  - tuning
  - nms
  - grid-search
---

# GPU-Hyperparameter-Tuning

## Uloga node-a

Ovaj node objašnjava GPU-side tuning oko full-resolution NMS-a. Cilj nije bio samo izabrati algoritamsku varijantu, već naći stabilne hyperparametre za live/runtime ponašanje.

## Parametri koji su testirani

- NMS implementacija: `2d` vs `separable`
- Full-resolution NMS radius: `r3`, `r6`, `r8`
- GPU compute dtype: `float16` vs `float32`
- worker konfiguracije u live grid-search-u
- throttle / target output FPS po kameri

## Najbolji stabilni default

```text
nms_impl = separable
nms_radius_fullres = 6
gpu_compute_dtype = float32
```

## Nalaz: separable/r6

`separable` NMS sa radius 6 je imao najbolji robustan rezultat. Neke `2d/r3` konfiguracije su mogle da imaju dobre best-case rezultate, ali median ponašanje i stabilnost su favorizovali `separable/r6`.

| NMS impl / radius | Runs | Median FPS | Best FPS | Median avg E2E | Best avg E2E | Median P95 E2E | Interpretation |
|---|---:|---:|---:|---:|---:|---:|---|
| `separable/r6` | 38 | 16.34 | 18.13 | 372.54 ms | 185.88 ms | 543.23 ms | najrobustniji default |
| `2d/r3` | 4 | 12.60 | 17.21 | 579.62 ms | 226.42 ms | 874.06 ms | dobar best-case, slabiji median |
| `separable/r3` | 4 | 12.15 | 17.35 | 602.99 ms | 257.51 ms | 940.86 ms | slabija stabilnost |
| `separable/r8` | 4 | 12.13 | 16.23 | 612.54 ms | 251.47 ms | 947.82 ms | pripremljen u 10-camera run-u, ali nije grid-search default |

## Nalaz: float32 vs float16

Za postprocessing workload `float32` je bio bolji od `float16`. To je važna lekcija jer FP16 nije univerzalno brži. Kod ovog tipa operacija mogu da dominiraju konverzije, scheduling, memory pattern ili backend overhead.

| GPU dtype / radius | Runs | Median FPS | Best FPS | Median avg E2E | Best avg E2E | Median P95 E2E |
|---|---:|---:|---:|---:|---:|---:|
| `float32/r6` | 38 | 16.34 | 18.13 | 367.74 ms | 185.88 ms | 538.39 ms |
| `float16/r6` | 4 | 8.12 | 8.40 | 855.51 ms | 801.64 ms | 1299.13 ms |

Preferiranje **Float32** nad **Float16** za GPU postprocessing je zato empirijski nalaz, ne teorijska pretpostavka. Ovaj workload nije veliki dense GEMM gde FP16 automatski pobeđuje; više liči na mask/NMS/top-k/scheduling problem gde konverzije i memory access pattern mogu da pojedu potencijalnu korist polupreciznosti.

## Decision

Za GPU fullres live/runtime testove koristiti `separable/r6/float32` kao robustan default.
