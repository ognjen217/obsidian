---
type: root
project: amd-migraphx-pose-pipeline
tags:
  - migraphx-pipeline
  - experiment-map
  - root
graph_rule: root-links-only-to-three-main-branches
---

# Index

## Osnovna priča o eksperimentima

Ovaj vault opisuje seriju eksperimenata za optimizaciju end-to-end pipeline-a za detekciju ljudskih ključnih tačaka na AMD hardveru. Polazni sistem koristi model koji se izvršava kroz MIGraphX / `.mxr`, dok je klasični deo postprocessinga ostao u Python/CPU logici.

Ključni nalaz cele serije eksperimenata je promena bottleneck-a. Kada se neuronska mreža ubrza kroz MIGraphX runtime, samo model inference više nije glavni problem. Dominantna cena prelazi na postprocessing: resize heatmapa i PAF-ova, izdvajanje pikova, NMS, ekstrakciju kandidata, PAF-based grouping i finalno sklapanje skeleton-a.

Finalna validacija zato razdvaja dve klase rezultata. **Accuracy-preserving lane** čuva referentni COCO tuple `AP/AP50/AP75/AR = 0.3995/0.6706/0.4020/0.4603`; u toj klasi najbolji praktični runtime je `gpu_nms_fullres_two_process` sa **COCO e2e 28.1 ms**, **CCTV e2e 47.5 ms** i **17.41 pipeline FPS**. **Throughput lane** svesno žrtvuje AP: `gpu_nms_lowres_two_process` pada na **AP 0.2479**, ali stiže do **17.9 ms CCTV e2e** i **32.11 pipeline FPS**.

Najvažniji sistemski rezultat nije samo da je GPU NMS brži od CPU NMS-a, nego **zašto**: finalna varijanta kombinuje full-resolution GPU NMS sa **two-process arhitekturom**, tako da se MIGraphX inference i postprocessing mogu preklapati. U live 10-camera grid-search-u dodatno se koristi **latest-frame buffering**, jer FIFO red u overload-u akumulira stare frame-ove i direktno pretvara backlog u latenciju.

Zato eksperimenti nisu organizovani samo kao lista varijanti, već kao tri nezavisna pravca optimizacije:

- [[Grana-CPU]]
- [[Grana-GPU]]
- [[Grana-MIGraphX]]

## Kako čitati ovaj vault

Prvo otvoriti jednu od tri glavne grane. Svaka grana ima svoju lokalnu priču, svoje eksperimente i svoje zaključke.

- CPU grana odgovara na pitanje: koliko daleko može da se optimizuje postojeći CPU-side postprocessing?
- GPU grana odgovara na pitanje: koji delovi postprocessinga imaju smisla kao GPU accelerated runtime?
- MIGraphX grana odgovara na pitanje: šta se može prevesti u ONNX-compatible statički graf i kompajlirati u `.mxr`?

## Važno pravilo grafa

Root node namerno ima samo tri izlazne veze. CPU, GPU i MIGraphX branch-evi se ne ukrštaju direktno. Zajedničke teme postoje u folderu `90_Shared_Themes`, ali nisu wikilinkovane iz glavnog toka kako bi Graph View ostao pregledan.
