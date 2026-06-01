---
type: experiment-node
branch: GPU
decision: mixed
tags:
  - gpu-optimisation
  - paf
  - grouping
  - mixed-result
---

# GPU-Full-Resolution-PAF

## Uloga node-a

Ovaj node opisuje pokušaj da se više full-resolution PAF/grouping rada prebaci ka GPU path-u.

## Zašto je testirano

Nakon što je GPU NMS dao dobar rezultat, logično sledeće pitanje je bilo: da li se još veći deo postprocessinga može ubrzati na GPU-u? PAF/grouping je sledeći veliki komad logike, pa je prirodno testirati njegovo pomeranje prema GPU strani.

## Rezultat

Eksperimentalni zaključak je mixed: accuracy je bila prihvatljiva, ali runtime nije bio bolji od kombinacije GPU-NMS + CPU grouping.

## Zašto “više GPU-a” nije automatski bolje

PAF/grouping logika nije jednostavna tensor operacija. Ona uključuje:

- evaluaciju veza između kandidata,
- uslovnu logiku,
- potencijalno promenljiv broj kandidata,
- organizaciju skeleton-a,
- sinhronizaciju i transfer između CPU/GPU strana.

Zato dodatni GPU rad može da uvede overhead koji je veći od dobitka.

## Decision

Važan mixed rezultat. Ne preferirati preko full-resolution GPU NMS + CPU grouping pristupa.
