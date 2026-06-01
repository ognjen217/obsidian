---
type: experiment-node
branch: CPU
decision: mixed
tags:
  - cpu-optimisation
  - batching
  - k10
---

# CPU-Optimized-Batch-K10

## Uloga node-a

`optimized_batch_k10` je prvi konkretan CPU pokušaj da se ubrza ekstrakcija ključnih tačaka bez promene kompletne logike postprocessinga.

## Šta je promenjeno

Umesto da se zadrži pun broj kandidata, varijanta ograničava candidate budget na:

```text
max_keypoints = 10
```

Time se smanjuje količina kandidata koji prolaze kroz kasnije faze. Ideja je bila da manji broj kandidata ubrza extraction i smanji downstream pritisak.

## Zašto je testirano

Baseline je pokazao da je extraction jedan od dva najveća CPU troška. `K10` je najjednostavniji način da se testira koliko candidate budget direktno utiče na brzinu.

## Rezultati

| Metric | Standard | optimized_batch_k10 |
|---|---:|---:|
| AP | 0.3995 | 0.3777 |
| COCO e2e avg | 82.8 ms | 69.5 ms |
| CCTV e2e avg | 230.6 ms | 163.8 ms |
| Extract stage | 113.1 ms | 48.3 ms |
| Group stage | 96.9 ms | 96.2 ms |

## Interpretacija

Rezultat je potvrdio da batching i smanjenje broja kandidata pomažu extraction fazi. Extraction pada sa oko 113.1 ms na 48.3 ms. Međutim, grouping ostaje praktično isti red veličine, a AP pada na 0.3777.

To znači da `K10` rešava samo deo problema: ubrzava extraction, ali gubi deo ključnih kandidata koji su potrebni za referentnu preciznost.

## Decision

Koristan međukorak. Nije finalna preporuka jer je `K20` skoro jednako logičan, a vraća punu preciznost.
