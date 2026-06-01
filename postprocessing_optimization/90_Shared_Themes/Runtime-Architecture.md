---
type: shared-theme
tags:
  - shared-theme
  - runtime-architecture
  - two-process
graph_rule: intentionally-not-linked-from-main-branches
---

# Runtime-Architecture

Ovaj fajl je zajednička tema i namerno nije wikilinkovan iz glavnih grana.

## Zašto postoji

Mikrobenchmark nije isto što i sistemski benchmark. Funkcija može biti brža izolovano, ali da ne popravi runtime ako se pojave queue pressure, shared-memory overhead, GPU contention ili sinhronizacija između faza.

## Single-process

Single-process benchmark izvršava inference i postprocessing u jednom Python procesu. Koristan je za čistu varijantnu poredbu zato što uklanja inter-process communication iz jednačine.

Testirane porodice uključuju:

- `standard`,
- `optimized_batch_k20_fast`,
- `lowres_cpu_group`,
- GPU-NMS full-resolution paths,
- GPU low-resolution PAF/NMS paths.

## Two-process

Two-process pipeline deli rad na:

```text
Process 1: MIGraphX inference
Process 2: postprocessing
Communication: shared memory + queues
```

Ova arhitektura omogućava overlap inference-a i postprocessinga. Zbog toga pipeline FPS može da poraste čak i kada isolated per-frame latency nije uvek bolji.

## Validirani runtime rezultati

| Variant | AP | COCO e2e avg | CCTV e2e avg | Pipeline FPS | Interpretation |
|---|---:|---:|---:|---:|---|
| cpu_k20_fast_two_process | 0.3995 | 35.9 ms | 76.4 ms | 14.2 | Best CPU runtime fallback |
| gpu_nms_fullres_two_process | 0.3995 | 28.1 ms | 47.5 ms | 17.4 | Best accuracy-preserving runtime |
| gpu_nms_lowres_two_process | 0.2479 | 19.3 ms | 17.9 ms | 32.1 | Fastest, but degraded accuracy |

## Ključna lekcija

Najbolji deployment rezultat nije samo izbor algoritma, već kombinacija algoritma i pipeline arhitekture.
