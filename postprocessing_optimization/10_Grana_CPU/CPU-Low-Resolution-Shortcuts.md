---
type: experiment-node
branch: CPU
decision: rejected-for-reference-accuracy
tags:
  - cpu-optimisation
  - degraded-accuracy
  - low-resolution
  - throughput-boundary
---

# CPU-Low-Resolution-Shortcuts

## Uloga node-a

Ovaj node opisuje CPU prečice koje namerno smanjuju količinu posla u postprocessingu kako bi se izmerila gornja granica brzine. One nisu zamišljene kao reference-accuracy rešenja.

## Varijante

- `fast_no_resize`
- `lowres_cpu_group`

## Šta je testirano

`fast_no_resize` smanjuje ili preskače resize-related rad. `lowres_cpu_group` koristi low-resolution grouping put. Oba pravca pokušavaju da minimizuju CPU rad pre i tokom grouping-a.

## Rezultati

| Variant | AP | COCO e2e avg | CCTV e2e avg | Extract | Group | CCTV FPS |
|---|---:|---:|---:|---:|---:|---:|
| fast_no_resize | 0.2184 | 20.8 ms | 16.2 ms | 0.8 ms | 2.6 ms | 61.66 FPS |
| lowres_cpu_group | 0.2184 | 20.2 ms | 15.2 ms | 0.5 ms | 2.2 ms | 65.58 FPS |

## Interpretacija

Ove varijante su ekstremno brze, ali AP pada sa referentnih 0.3995 na 0.2184. To je prevelik pad za accuracy-preserving pipeline.

U poređenju ove dve prečice, `lowres_cpu_group` je malo bolji jer je brži uz isti AP. Ipak, obe varijante pripadaju posebnoj degraded-accuracy lane.

## Decision

Koristiti samo kao throughput boundary / degraded-accuracy eksperiment, ne kao finalni deployment kandidat.
