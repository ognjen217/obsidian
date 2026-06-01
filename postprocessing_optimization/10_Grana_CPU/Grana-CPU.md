---
type: main-branch
branch: CPU
tags:
  - cpu-optimisation
  - postprocessing
graph_role: main_branch
---

# Grana-CPU

## CPU-based optimisations

Ova grana pokriva sve optimizacije koje ostaju na CPU strani. To znači da se heatmap/PAF obrada, ekstrakcija kandidata i grouping i dalje izvršavaju kroz CPU-side logiku, ali se menjaju implementacioni detalji kako bi se smanjio runtime.

Polazna tačka je standardni CPU postprocessing: `extract_keypoints` + `group_keypoints`. Taj put je precizan, ali je prespor za video i multi-camera scenarije. Profilisanje je pokazalo da je problem dominantno u `extract_ms` i `group_ms`, a ne u samom MIGraphX inference-u.

## Eksperimentalni tok

- [[CPU-Standard-Baseline]]
- [[CPU-Optimized-Batch-K10]]
- [[CPU-Optimized-Batch-K20]]
- [[CPU-K20-Fast-findNonZero]]
- [[CPU-Secondary-Experiments]]
- [[CPU-Low-Resolution-Shortcuts]]

## Logika grane

CPU grana ide od najkonzervativnijeg ka agresivnijem:

1. prvo se meri standardni CPU baseline;
2. zatim se testira batching sa manjim brojem kandidata (`K10`);
3. zatim se povećava budžet kandidata na `K20` da bi se povratila preciznost;
4. zatim se menja način ekstrakcije kandidata kroz `findNonZero v1`;
5. dodatni pokušaji kao `findNonZero v2`, PPL/thread tuning i low-resolution shortcut-i služe da se potvrdi granica CPU pristupa.

## Glavni zaključak grane

Najbolji CPU-side rezultat je `optimized_batch_k20_fast` / `findNonZero v1 + k20`. Ta varijanta čuva referentni AP, ali dramatično smanjuje grouping i total postprocess cenu. Ona nije najbrži ukupni runtime u celom projektu, ali jeste najbolji stabilni CPU fallback.
