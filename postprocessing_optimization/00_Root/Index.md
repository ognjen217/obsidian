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
