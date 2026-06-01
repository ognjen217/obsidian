---
type: experiment-node
branch: GPU
decision: keep
tags:
  - gpu-optimisation
  - nms
  - full-resolution
  - best-runtime
  - accuracy-preserving
---

# GPU-NMS-Full-Resolution

## Uloga node-a

Ovo je glavni GPU rezultat. Ideja je da se NMS/peak extraction izvrši na GPU u full resolution režimu, dok se accuracy-sensitive grouping logika zadržava dovoljno blizu referentnom ponašanju.

U finalnim rezultatima ovaj pravac je predstavljen kroz:

```text
gpu_nms_fullres_two_process
```

## Zašto full resolution

Low-resolution shortcut-i su pokazali da agresivno smanjivanje rezolucije može da uništi AP. Zato je full-resolution varijanta važna: ona testira da li GPU može da ubrza NMS bez gubitka informacija koje su bitne za tačnost.

## Rezultati

| Metric | Standard | GPU fullres NMS |
|---|---:|---:|
| AP | 0.3995 | 0.3995 |
| AP50 | 0.6706 | 0.6706 |
| AP75 | 0.4020 | 0.4020 |
| AR | 0.4603 | 0.4603 |
| COCO e2e avg | 82.8 ms | 28.1 ms |
| COCO post avg | 63.7 ms | 12.0 ms |
| CCTV e2e avg | 230.6 ms | 47.5 ms |
| CCTV post avg | 219.3 ms | 34.1 ms |
| Pipeline FPS | 4.34 FPS baseline | 17.41 FPS |

## Interpretacija

Ovo je najbolji balans u celom projektu: zadržava referentnu preciznost, a dramatično smanjuje runtime. COCO end-to-end latency pada sa 82.8 ms na 28.1 ms, a CCTV latency sa 230.6 ms na 47.5 ms.

## Decision

Najbolji accuracy-preserving GPU runtime i glavni deployment kandidat.
