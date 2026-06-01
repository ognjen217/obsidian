---
type: experiment-node
branch: MIGraphX
decision: keep
tags:
  - migraphx-migration
  - validation
  - nms
  - accuracy-preserving
---

# MX-NMS-Validation

## Uloga node-a

Ovaj node opisuje correctness i accuracy validaciju MIGraphX compiled NMS varijanti.

## Validirane varijante

- `migraphx_nms`
- `migraphx_nms_k20`

## Rezultati

| Variant | AP | AP50 | AP75 | AR | COCO e2e avg | COCO post avg |
|---|---:|---:|---:|---:|---:|---:|
| standard | 0.3995 | 0.6706 | 0.4020 | 0.4603 | 82.8 ms | 63.7 ms |
| migraphx_nms | 0.4061 | 0.6729 | 0.4117 | 0.4661 | 37.6 ms | 23.3 ms |
| migraphx_nms_k20 | 0.3995 | 0.6706 | 0.4020 | 0.4603 | 31.9 ms | 19.5 ms |

## Interpretacija accuracy-ja

`migraphx_nms_k20` je čist accuracy-preserving rezultat jer tačno vraća standardni AP/AP50/AP75/AR tuple. `migraphx_nms` je zanimljiv jer ima blago viši izmereni AP, ali to ne znači automatski da je bolji deployment kandidat.

## Runtime placement

| Variant | AP | COCO e2e avg | CCTV e2e avg | CCTV post avg | CCTV FPS | Placement |
|---|---:|---:|---:|---:|---:|---|
| `migraphx_nms` | 0.4061 | 37.6 ms | 84.3 ms | 75.1 ms | 11.87 FPS | accurate compiled-NMS path |
| `migraphx_nms_k20` | 0.3995 | 31.9 ms | 84.7 ms | 75.7 ms | 11.80 FPS | accuracy-preserving compiled-NMS path |
| `gpu_nms_fullres_two_process` | 0.3995 | 28.1 ms | 47.5 ms | 34.1 ms | 17.41 pipeline FPS | preferred runtime |

Ovo pozicionira MIGraphX NMS kao uspešan correctness/feasibility rezultat, ali ne kao najbrži deployment path. `migraphx_nms_k20` ima vrlo dobar **COCO e2e 31.9 ms**, ali se na CCTV workload-u zadržava oko **84.7 ms**, jer compiled NMS nije jedini trošak.

## Caveat o percentilima

Za MIGraphX accuracy summary, p50/p95 vrednosti su označene kao weighted averages across chunks, a ne kao egzaktni per-image percentili iz kompletnih detaljnih CSV-ova. Zbog toga su average latency vrednosti najkorisnije za poređenje, dok percentili treba da se tumače opreznije.

## Decision

Compiled NMS je funkcionalno validiran i može da očuva kvalitet zadatka.
