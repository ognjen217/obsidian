---
type: experiment-node
branch: MIGraphX
decision: future-work
tags:
  - migraphx-migration
  - future-work
  - onnx
  - mxr
---

# MX-Next-Direction

## Uloga node-a

Ovaj node opisuje realan sledeći pravac MIGraphX migracije nakon compiled NMS eksperimenata.

## Šta ne treba raditi odmah

Ne treba pokušavati da se ceo postprocessing odjednom prebaci u `.mxr`. Eksperimenti su pokazali da variable-length grouping i skeleton assembly nisu čisti kandidati za jednostavnu ONNX/MIGraphX migraciju.

## Preporučeni split

```text
U ONNX / MIGraphX graf:
  heatmap thresholding
  local NMS
  TopK / candidate filtering
  mask generation
  shape-preserving tensor ops

Van grafa:
  final variable-length assembly
  PAF-dependent grouping
  skeleton construction
  conditional pose logic
```

## Sledeći cilj

Najveći potencijal je u smanjenju CPU/GPU transfera i Python bridge overhead-a. Ako compiled deo može da proizvede pogodniji format za downstream grouping, ukupni runtime može da padne i bez kompletne migracije skeleton assembly-ja.

## Potencijalni istraživački pravci

- fuzija threshold + NMS + TopK u jedan compiled subgraph,
- statički candidate tensor layout umesto promenljivih Python listi,
- smanjenje `extract_from_mask_ms`,
- direktniji output layout za CPU grouping,
- custom GPU kernel za delove koji nisu dobro izraženi u ONNX-u.

## Decision

MIGraphX pravac ostaje relevantan, ali sledeći korak mora biti širenje tensor-only grafa i smanjenje bridge overhead-a, ne direktna migracija celog postprocessinga.
