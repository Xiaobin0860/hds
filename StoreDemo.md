# Store Demo

## Overview

```mermaid
%%{init: {"theme": "dark"}}%%
flowchart TD
    subgraph ui
        setting[SettingMenu]
        demo[StoreDemo]
    end
    cesi[CESI]
    dtvif[DTVIF]
    vrm[VRM]
    apm[APM]
    pq[PQStack]
    ds[StoreDemoService]

    setting --- cesi
    demo --- cesi

    cesi --- dtvif
    cesi --- apm
    cesi --- ds

    dtvif --- vrm
    apm --- vrm
    ds --- vrm

    vrm --- pq
```
