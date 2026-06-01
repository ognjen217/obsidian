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
| Max keypoints | 10 |
| Detection threshold | 0.15 |
| NMS implementation | separable |
| Full-resolution NMS radius | 8 in prepared run; grid-search default prefers 6 |

## Latest-frame buffering

`latest` buffering znači da sistem prioritizuje najnoviji frame. Ako je pipeline opterećen, stariji frame-ovi mogu biti overwritten/dropped. To je poželjno u live monitoringu jer je svežina rezultata važnija od obrade svakog pojedinačnog frame-a.

Razlog je sistemski: ulazni pritisak je **10 kamera x 24 FPS = 240 frame/s**. Nijedan accuracy-preserving kandidat iz finalne validacije ne proizvodi 240 rezultata/s, pa FIFO red ne bi rešio overload nego bi ga pretvorio u rastući backlog. Latest-frame politika bira bounded-latency ponašanje: drop/replacement je prihvatljiviji od obrade frame-a koji više ne predstavlja trenutno stanje scene.

## Soft backpressure

Soft backpressure dozvoljava sistemu da reaguje na overload bez potpunog blokiranja frame production-a. U praksi pomaže da latency ostane ograničen i da queue ne raste nekontrolisano.

## Grid-search kriterijum

U 10-camera grid-search-u korišćen je pomoćni **Balanced Score**:

```text
balanced_score = avg_e2e_ms
               + 0.5 * p95_e2e_ms
               + 5 * queue_infer_to_post_ms
               - 20 * aggregate_fps
```

Niži skor je bolji. Formula namerno favorizuje svež live output: prosečna latencija i p95 direktno ulaze u cenu, queue pressure je jako kažnjen jer signalizira akumulaciju rada, a aggregate FPS je nagrada tek kada sistem ne proizvodi stale rezultate.

## Najbolji live kandidati iz grid-search-a

| Setup | Aggregate FPS | Avg E2E | P95 E2E | Interpretation |
|---|---:|---:|---:|---|
| `gpu_nms_fullres_two_process`, latest, I1/P3, soft, 300 ms, 2 FPS/cam | 14.15 | 185.88 ms | 263.03 ms | najbolji low-latency live setup |
| `gpu_nms_fullres_two_process`, latest, I1/P5, soft, 300 ms, 3 FPS/cam | 16.84 | 226.42 ms | 284.85 ms | bolji output rate uz i dalje nisku latenciju |
| best raw throughput CPU fast row | 19.46 | 506.71 ms | 733.00 ms | veći FPS, ali stale live output |

Za najbolji low-latency setup dodatno su izmereni **Avg postprocess 113.84 ms** i **Queue infer-to-post 7.43 ms**. Taj nizak queue pressure je presudan: pokazuje da latest buffering i soft backpressure ne samo ubrzavaju stage, nego sprečavaju akumulaciju rada između inference-a i postprocess-a.

Grid-search zato bira GPU full-resolution NMS porodicu, iako CPU optimizovane varijante mogu da imaju veći raw aggregate FPS. Za monitoring je **latency freshness** primarni cilj.

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
