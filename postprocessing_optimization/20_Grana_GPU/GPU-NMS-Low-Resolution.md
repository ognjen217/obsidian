---
type: experiment-node
branch: GPU
decision: degraded-accuracy
tags:
  - gpu-optimisation
  - low-resolution
  - degraded-accuracy
  - throughput-boundary
---

# GPU-NMS-Low-Resolution

## Uloga node-a

Ovaj node predstavlja najbrži GPU-assisted runtime pravac, ali sa jasnim padom accuracy-ja.

Finalna varijanta u validacionim tabelama:

```text
gpu_nms_lowres_two_process
```

## Zašto je testirano

Low-resolution GPU put testira gornju granicu throughput-a. Ako se smanji rezolucija PAF/grouping ili NMS puta, postprocess postaje znatno jeftiniji. Cilj je bio da se vidi koliko se latency može spustiti i kolika je cena u AP.

## Rezultati

| Metric | Value |
|---|---:|
| AP | 0.2479 |
| COCO e2e avg | 19.3 ms |
| COCO post avg | 3.7 ms |
| CCTV e2e avg | 17.9 ms |
| CCTV post avg | 8.3 ms |
| CCTV pipeline FPS | 32.11 FPS |

## Interpretacija

Ovo je izuzetno brzo: CCTV end-to-end oko 17.9 ms i pipeline FPS oko 32.1. Međutim, AP pada na 0.2479, što je veliki pad u odnosu na referentnih 0.3995.

Zanimljivo je da je ova varijanta tačnija od CPU low-resolution shortcut-a, ali i dalje nije dovoljno dobra kao reference-accuracy runtime.

## Decision

Koristiti samo kao maximum-throughput degraded-accuracy lane.
