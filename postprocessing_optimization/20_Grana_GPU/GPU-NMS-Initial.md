---
type: experiment-node
branch: GPU
decision: stepping-stone
tags:
  - gpu-optimisation
  - nms
  - integration
---

# GPU-NMS-Initial

## Uloga node-a

Ovo je prvi korak GPU postprocessing grane: uvođenje GPU heatmap NMS-a u postojeći pipeline, dok CPU grouping ostaje prisutan.

## Zašto baš NMS

Heatmap NMS i peak extraction su prirodni kandidati za GPU zato što se operacije primenjuju paralelno po heatmap lokacijama i kanalima. Nasuprot tome, finalno grupisanje poza ima više uslovne logike, zavisi od PAF integracije i koristi strukture promenljive dužine.

## Šta je testirano

Cilj početnog moda nije bio samo da “pobedi” sve ostale varijante, nego da pokaže da GPU NMS može da se ubaci u pipeline bez rušenja output strukture.

## Šta je naučeno

Početni GPU-NMS je otvorio put za full-resolution varijantu. Zadržavanje CPU grouping-a se pokazalo kao dobra strategija jer je čuvalo correctness dok se najparalelniji deo posla prebacuje na GPU.

## Decision

Stepping-stone prema `GPU-NMS-Full-Resolution`.
