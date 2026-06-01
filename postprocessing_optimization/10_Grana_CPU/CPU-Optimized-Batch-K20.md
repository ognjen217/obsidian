---
type: experiment-node
branch: CPU
decision: keep
tags:
  - cpu-optimisation
  - batching
  - k20
  - accuracy-preserving
---

# CPU-Optimized-Batch-K20

## Uloga node-a

`optimized_batch_k20` je odgovor na glavni problem `K10` varijante: AP pad. Zadržava batching pristup, ali povećava candidate budget.

## Šta je promenjeno

Candidate budget se povećava na:

```text
max_keypoints = 20
```

Ideja je da se zadrži veći broj potencijalnih ključnih tačaka, ali da se i dalje koristi brži batch extraction.

## Zašto je testirano

`K10` je pokazao da se extraction može ubrzati, ali da je 10 kandidata premalo za očuvanje preciznosti. `K20` testira da li je 20 kandidata dovoljan kompromis: više kandidata nego K10, ali i dalje mnogo kontrolisaniji tok od standardnog baseline-a.

## Rezultati

| Metric | Standard | optimized_batch_k20 |
|---|---:|---:|
| AP | 0.3995 | 0.3995 |
| AP50 | 0.6706 | 0.6706 |
| AP75 | 0.4020 | 0.4020 |
| AR | 0.4603 | 0.4603 |
| COCO e2e avg | 82.8 ms | 68.9 ms |
| CCTV e2e avg | 230.6 ms | 166.0 ms |
| Extract stage | 113.1 ms | 47.9 ms |
| Group stage | 96.9 ms | 98.6 ms |

## Interpretacija

`K20` vraća kompletan accuracy tuple standardnog pipeline-a. To je važna potvrda: problem kod `K10` nije batching sam po sebi, nego previše agresivno smanjenje kandidata.

Međutim, iako extraction ostaje brz, grouping se ne smanjuje značajno. To znači da `optimized_batch_k20` jeste safe improvement, ali nije dovoljan za konačni runtime cilj.

## Decision

Prva bezbedna CPU optimizacija. Dobra kao stabilan korak, ali dalje treba rešavati grouping/candidate handling.
