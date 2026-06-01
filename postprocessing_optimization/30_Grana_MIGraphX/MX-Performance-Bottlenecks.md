---
type: experiment-node
branch: MIGraphX
decision: secondary
tags:
  - migraphx-migration
  - performance
  - bottleneck
  - postprocessing
---

# MX-Performance-Bottlenecks

## Uloga node-a

Ovaj node objašnjava zašto MIGraphX compiled NMS, iako tehnički uspešan, nije izabran kao finalni najbrži runtime.

## Video benchmark rezultat

| Variant | CCTV e2e avg | CCTV post avg | Interpretation |
|---|---:|---:|---|
| standard | 230.6 ms | 219.3 ms | reference baseline |
| migraphx_nms | 84.3 ms | 75.1 ms | much faster than standard |
| migraphx_nms_k20 | 84.7 ms | 75.7 ms | preserves reference AP |
| gpu_nms_fullres_two_process | 47.5 ms | 34.1 ms | faster practical runtime |

## Stage-level analiza

Kod MIGraphX NMS varijanti, compiled NMS nije jedini trošak:

| Stage | Approximate cost |
|---|---:|
| `mx_nms_ms` | 41.9–42.3 ms |
| `extract_from_mask_ms` | ~19.2 ms |
| `group_ms` | ~5.3 ms |
| total postprocess | 75.1–75.7 ms |

## Šta ovo znači

Kompajliranje NMS-a značajno popravlja standardni CPU baseline, ali ne uklanja okolne troškove. `extract_from_mask_ms`, CPU-side bridge i sync/transfer overhead i dalje ostaju veliki deo ukupnog vremena.

## Ključna lekcija

Ne treba meriti samo brzinu compiled NMS grafa izolovano. Bitno je end-to-end ponašanje: kako se compiled output vraća u ostatak postprocess pipeline-a i koliko košta taj bridge.

## Decision

MIGraphX NMS je successful feasibility path, ali još nije finalni deployment path.
