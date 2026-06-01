---
type: main-branch
branch: GPU
tags:
  - gpu-optimisation
  - postprocessing
graph_role: main_branch
---

# Grana-GPU

## GPU-based optimisations

GPU grana testira prebacivanje paralelnih delova postprocessinga na GPU. Fokus je na heatmap NMS-u i peak extraction-u, jer su te operacije pogodnije za paralelizaciju od finalnog skeleton/grouping dela.

Za razliku od MIGraphX grane, ovde cilj nije nužno da sve bude statički ONNX/MIGraphX graf. Cilj je praktično GPU ubrzanje postprocessinga uz očuvanje accuracy-ja.

## Eksperimentalni tok

- [[GPU-NMS-Initial]]
- [[GPU-NMS-Full-Resolution]]
- [[GPU-NMS-Low-Resolution]]
- [[GPU-Full-Resolution-PAF]]
- [[GPU-Hyperparameter-Tuning]]
- [[GPU-Live-Grid-Search]]

## Logika grane

GPU grana je išla inkrementalno:

1. ubaciti GPU heatmap NMS bez menjanja cele pipeline logike;
2. zadržati CPU grouping kao correctness anchor;
3. proveriti da li full-resolution GPU NMS čuva accuracy;
4. testirati low-resolution varijante kao throughput boundary;
5. proveriti da li dodatno prebacivanje PAF/grouping rada na GPU ima smisla;
6. kroz grid-search odabrati live-feed konfiguracije.

## Glavni zaključak grane

`gpu_nms_fullres_two_process` je najbolji accuracy-preserving runtime. Low-resolution GPU put je brži, ali gubi previše AP. Pokušaj da se više PAF/grouping rada prebaci na GPU nije automatski bolji, jer transfer/sync overhead može pojesti dobitak.
