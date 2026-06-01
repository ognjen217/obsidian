---
type: experiment-node
branch: MIGraphX
decision: boundary
tags:
  - migraphx-migration
  - onnx
  - feasibility
  - graph-boundary
---

# MX-ONNX-Feasibility-Boundary

## Uloga node-a

Ovaj node definiše granicu između postprocess logike koja je pogodna za MIGraphX/ONNX i logike koja trenutno nije dobra za direktno `.mxr` kompajliranje.

## Tensor-friendly kandidati

Ovi delovi su dobri kandidati za MIGraphX zato što su shape-preserving i tensor-oriented:

- heatmap thresholding,
- MaxPool-like local NMS,
- TopK / candidate filtering,
- shape-preserving tensor operations,
- lokalno poređenje piksela/kandidata,
- mask generation.

## Teški kandidati

Ovi delovi su teški za čistu ONNX/MIGraphX migraciju:

- PAF line integration,
- skeleton assembly,
- promenljiv broj keypoint kandidata,
- variable-length liste,
- Python loops,
- conditional grouping,
- logika koja zavisi od broja detektovanih kandidata po slici.

## Zašto je ovo problem

MIGraphX najbolje radi kada je graf statičan i kada su oblici tenzora poznati ili kontrolisani. Pose assembly je po prirodi dinamičniji: broj kandidata varira, veze između kandidata zavise od PAF rezultata, a izlaz može imati promenljiv broj osoba i keypoint veza.

## Decision

Ne pokušavati odmah kompletan postprocessing u `.mxr`. Prvo širiti compiled deo samo na tensor-only operacije.
