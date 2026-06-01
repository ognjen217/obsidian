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

## Caveat o percentilima

Za MIGraphX accuracy summary, p50/p95 vrednosti su označene kao weighted averages across chunks, a ne kao egzaktni per-image percentili iz kompletnih detaljnih CSV-ova. Zbog toga su average latency vrednosti najkorisnije za poređenje, dok percentili treba da se tumače opreznije.

## Decision

Compiled NMS je funkcionalno validiran i može da očuva kvalitet zadatka.
