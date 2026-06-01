---
type: shared-theme
tags:
  - shared-theme
  - multi-camera
  - live-simulation
  - buffering
  - backpressure
graph_rule: intentionally-not-linked-from-main-branches
---

# Multi-Camera-Simulation

Ovaj fajl je zajednička tema i namerno nije wikilinkovan iz glavnih grana.

## Cilj simulacije

Multi-camera simulacija proverava kako se optimizovani pipeline ponaša kada se istovremeno obrađuje više kamera. Cilj nije samo raw throughput, već real-time stabilnost.

## Konfigurisani setup

| Parameter | Selected configuration |
|---|---:|
| Simulation script | `simulate_10_camera_stream_profiled.py` |
| Number of cameras | 10 |
| Camera FPS target | 24 FPS per camera |
| Theoretical input pressure | 240 input frames/s |
| Test duration | 60 s |
| Warmup | 10 s |
| Model | `pose_model1_fp16_ref1.mxr` |
| Variant | `gpu_nms_fullres_two_process` |
| Buffer mode | latest |
| Backpressure mode | soft |
| Inference workers | 1 |
| Postprocess workers | 3 |
| Shared-map slots | 16 |
| GPU compute dtype | float32 |

## Latest-frame buffering

`latest` buffering znači da sistem prioritizuje najnoviji frame. Ako je pipeline opterećen, stariji frame-ovi mogu biti overwritten/dropped. To je poželjno u live monitoringu jer je svežina rezultata važnija od obrade svakog pojedinačnog frame-a.

## Soft backpressure

Soft backpressure dozvoljava sistemu da reaguje na overload bez potpunog blokiranja frame production-a. U praksi pomaže da latency ostane ograničen i da queue ne raste nekontrolisano.

## Metrike koje treba gledati

- per-camera effective FPS,
- total processed throughput,
- average E2E latency,
- p95 latency,
- processed frames,
- dropped frames,
- queue depth / backlog,
- fairness između kamera,
- da li run završava bez crash/stall stanja.

## Interpretacija

Ako processed FPS ne može da stigne theoretical input pressure od 240 FPS, sistem mora da dropuje ili overwrituje frame-ove. To nije nužno neuspeh: za live-feed sistem cilj je stabilan i svež output, ne processing svakog ulaznog frame-a.
