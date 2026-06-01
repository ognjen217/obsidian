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

## Nalaz: float32 vs float16

Za postprocessing workload `float32` je bio bolji od `float16`. To je važna lekcija jer FP16 nije univerzalno brži. Kod ovog tipa operacija mogu da dominiraju konverzije, scheduling, memory pattern ili backend overhead.

## Decision

Za GPU fullres live/runtime testove koristiti `separable/r6/float32` kao robustan default.
