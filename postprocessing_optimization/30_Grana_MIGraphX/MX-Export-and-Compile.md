---
type: experiment-node
branch: MIGraphX
decision: keep
tags:
  - migraphx-migration
  - export
  - compile
  - onnx
  - mxr
---

# MX-Export-and-Compile

## Uloga node-a

Ovaj node opisuje engineering tok kojim je NMS-related deo postprocessinga izolovan, eksportovan i kompajliran u MIGraphX runtime.

## Zašto je ovo poseban pravac

Ovo nije isto što i “prebaciti operaciju na GPU”. MIGraphX zahteva da se operacije predstave kao statički graf koji može da se parsira, optimizuje i kompajlira. Zato je prvo pitanje bilo: koji deo postprocessinga uopšte može da postane graph-friendly?

## Skripte u eksperimentu

| Script | Role |
|---|---|
| `export_heatmap_nms_head.py` | eksport heatmap / NMS head-a |
| `compile_heatmap_nms_migraphx.py` | kompajliranje ONNX grafa kroz MIGraphX |
| `migraphx_nms.py` | runtime wrapper za compiled NMS path |
| `test_migraphx_nms_sanity.py` | sanity-check output ponašanja |
| `benchmark_migraphx_postprocess.py` | performance benchmark MIGraphX NMS varijanti |

## Tok rada

1. Izolovan je heatmap/NMS deo koji je najviše tensor-friendly.
2. Taj deo je eksportovan u ONNX-like formu.
3. ONNX graf je kompajliran u `.mxr`.
4. Napravljen je runtime wrapper koji vraća output u postojeći Python postprocess.
5. Sanity test je potvrdio da output može da se poredi sa očekivanim NMS ponašanjem.
6. Benchmark je proverio da li compiled NMS stvarno popravlja end-to-end runtime.

## Decision

Eksport/compile tok je uspešan i dokazuje tehničku izvodljivost compiled NMS pravca.
