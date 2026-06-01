---
type: experiment-node
branch: CPU
decision: reference
tags:
  - cpu-optimisation
  - baseline
  - postprocessing-bottleneck
---

# CPU-Standard-Baseline

## Uloga node-a

Ovaj node predstavlja početnu CPU referencu: standardni postprocessing sa CPU `extract_keypoints` i CPU `group_keypoints`. Svi kasniji CPU eksperimenti se porede sa ovom tačkom.

## Zašto je baseline važan

Baseline je važan iz dva razloga. Prvo, on čuva očekivani kvalitet detekcije i zato definiše accuracy target. Drugo, on pokazuje gde sistem realno troši vreme kada je inference već ubrzan kroz MIGraphX / FP16 runtime.

U ovom slučaju, inference nije dominantan deo runtime-a. Standardni postprocessing postaje najskuplji deo end-to-end pipeline-a.

## Ključne metrike

| Metric | Value |
|---|---:|
| AP | 0.3995 |
| AP50 | 0.6706 |
| AP75 | 0.4020 |
| AR | 0.4603 |
| COCO e2e avg | 82.8 ms |
| COCO post avg | 63.7 ms |
| CCTV e2e avg | 230.6 ms |
| CCTV post avg | 219.3 ms |
| CCTV FPS | 4.34 FPS |
| Extract stage | 113.1 ms |
| Group stage | 96.9 ms |
| Inference stage | 7.8 ms |
| Preprocess stage | 3.5 ms |

## Šta ove brojke znače

Najvažniji odnos je između inference vremena i postprocess vremena. Ako inference traje oko 7.8 ms, a extraction + grouping zajedno oko 210 ms, onda dalje ubrzavanje model inference-a neće rešiti sistemski bottleneck. Potrebno je dirati postprocessing.

## Interpretacija

Standard baseline je korektan, ali runtime ne može da skalira za real-time video ili multi-camera monitoring. On ostaje correctness reference, ne deployment kandidat.

## Decision

Koristiti samo kao referentnu tačku za tačnost i poređenje.
