---
type: experiment-node
branch: CPU
decision: mixed
tags:
  - cpu-optimisation
  - secondary-experiments
  - tuning
---

# CPU-Secondary-Experiments

## Uloga node-a

Ovaj node sakuplja CPU eksperimente koji nisu postali finalni kandidati, ali su važni za razumevanje zašto je `findNonZero v1 + K20` ostao najbolji CPU pravac.

## findNonZero v2 + K20

Nakon uspeha `findNonZero v1`, testirana je alternativna implementacija. Očekivanje je bilo da drugačiji način candidate extraction-a može dodatno smanjiti overhead.

Rezultat je bio slabiji od v1. To je važan negativan nalaz: pobeda CPU grane nije došla samo iz činjenice da se koristi `findNonZero`, nego iz konkretne implementacije i njene interakcije sa K20 candidate budget-om.

## PPL / thread tuning

Testiran je i dodatni CPU parallelism/thread tuning oko `optimized_batch_k20` familije. Ovo je bio logičan pokušaj jer su extraction i grouping CPU-bound faze.

Dobitak je bio ograničen. Kada je `findNonZero v1 + K20` već smanjio glavni candidate/grouping overhead, dodatno paralelizovanje više nije imalo isti leverage.

## Šta ovaj node pokazuje

Ovi eksperimenti sprečavaju pogrešan zaključak da “bilo kakvo CPU paralelizovanje rešava problem”. Pravi dobitak je došao iz smanjenja rada i bolje organizacije kandidata, ne samo iz dodavanja thread-ova.

## Decision

Sačuvati kao dokumentaciju istraženih alternativa. Ne koristiti kao finalni CPU path.
