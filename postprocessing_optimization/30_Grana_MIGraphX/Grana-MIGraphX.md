---
type: main-branch
branch: MIGraphX
tags:
  - migraphx-migration
  - onnx
  - mxr
graph_role: main_branch
---

# Grana-MIGraphX

## MIGraphX migration

MIGraphX grana je odvojena od GPU grane. GPU grana se bavi praktičnim GPU ubrzanjem postprocessinga, dok MIGraphX grana istražuje šta može da se predstavi kao ONNX-compatible statički graf i kompajlira u `.mxr`.

Ovaj pravac je važan jer bi idealno rešenje smanjilo Python/CPU overhead, smanjilo transfer/sync troškove i približilo inference + postprocess jednom compiled runtime-u.

## Eksperimentalni tok

- [[MX-Export-and-Compile]]
- [[MX-NMS-Validation]]
- [[MX-Performance-Bottlenecks]]
- [[MX-ONNX-Feasibility-Boundary]]
- [[MX-Next-Direction]]

## Logika grane

MIGraphX branch prati sledeći tok:

1. izolovati tensor-friendly NMS logiku;
2. eksportovati je kao ONNX graph;
3. kompajlirati je kroz MIGraphX u `.mxr`;
4. napraviti runtime wrapper;
5. sanity-checkovati output;
6. benchmarkovati `migraphx_nms` i `migraphx_nms_k20`;
7. definisati granicu šta je zaista `.mxr`-ready.

## Glavni zaključak

Compiled NMS je tehnički uspešan. Može da očuva AP i značajno poboljša standardni CPU postprocess. Ipak, trenutno nije najbrži deployment path zato što okolni troškovi, naročito extraction-from-mask i sync/transfer, ostaju značajni.
