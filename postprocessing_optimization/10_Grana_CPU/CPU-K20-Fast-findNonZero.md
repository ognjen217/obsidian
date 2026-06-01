---
type: experiment-node
branch: CPU
decision: keep
tags:
  - cpu-optimisation
  - findNonZero
  - k20
  - best-cpu-fallback
  - accuracy-preserving
---

# CPU-K20-Fast-findNonZero

## Uloga node-a

Ovo je glavni CPU-side proboj. Varijanta se u rezultatima pojavljuje kao `optimized_batch_k20_fast`, a konceptualno odgovara putu `findNonZero v1 + k20`.

## Problem koji rešava

`optimized_batch_k20` je rešio AP, ali nije rešio grouping. Nakon K20, sistem je i dalje imao skup candidate/grouping tok. Cilj ove varijante je bio da se zadrži accuracy-preserving K20 budžet, ali da se ubrza način pronalaženja i organizovanja kandidata.

## Šta je promenjeno

Umesto sporijeg per-pixel/per-channel skeniranja, koristi se brži `findNonZero` pristup za candidate extraction. Time se smanjuje overhead oko maski i kandidata pre grupisanja.

## Rezultati

| Metric | Standard | K20 fast / findNonZero v1 |
|---|---:|---:|
| AP | 0.3995 | 0.3995 |
| AP50 | 0.6706 | 0.6706 |
| AP75 | 0.4020 | 0.4020 |
| AR | 0.4603 | 0.4603 |
| COCO e2e avg | 82.8 ms | 39.2 ms |
| CCTV e2e avg | 230.6 ms | 73.0 ms |
| CCTV postprocess | 219.3 ms | 61.7 ms |
| Extract stage | 113.1 ms | 48.2 ms |
| Group stage | 96.9 ms | 5.3 ms |
| CCTV FPS | 4.34 FPS | 13.70 FPS |

## Najvažniji nalaz

Najveći pomak nije samo u extraction-u, nego u kolapsu grouping vremena sa 96.9 ms na 5.3 ms. To znači da je varijanta promenila runtime klasu CPU postprocessinga.

## Interpretacija

Ovo je prva CPU-only varijanta koja je i tačna i praktično brza. Ona ne zahteva GPU postprocessing i zato je odlična fallback opcija. Ipak, kao finalni live runtime, kasnije GPU full-resolution NMS varijante daju bolji latency.

## Decision

Najbolji stabilni CPU fallback.
