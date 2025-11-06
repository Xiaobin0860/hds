# AI Contrast

## Overview

```mermaid
%%{init: {"theme": "dark"}}%%
flowchart TD
    ui
    cesi
    vrm
    vrm_db@{ shape: doc, label: "Vrm_MenuSupportConfig.txt" }
    pq[PQStack]

    ui ---|"SYS_IsSupported(PICTURE_DETAIL_ENHANCER)"| cesi
    cesi ---|"VRM_IsSupported(VRM_PROPERTY_PICTURE_DETAIL_ENHANCER)"| vrm
    vrm --> vrm_db
```
