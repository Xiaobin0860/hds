# AISR OSD

## Introduction

The AISR engin can locate at video path or Graphic Path.

```mermaid
%%{init: {"theme": "dark"}}%%
flowchart LR
    subgraph Video Path
        vp1[Video Path]
        vp1 --> aisr1[AISR]
        aisr1 --> vp2[Video Path]
    end
    aisr[AISR]
    subgraph Graphic Path
        gpath[Graphic Path]
        gpath --> aisr2[AISR]
        aisr2 --> gp2[Graphic Path]
    end
    vp2 --> c[Compose]
    gp2 --> c[Compose]
    aisr <-->|video>70%| aisr1
    aisr <--> aisr2

    style aisr1 stroke-dasharray: 5 5
    style aisr2 stroke-dasharray: 5 5
```

AISR OSD & video path switch behavior:

if video display size is larger than a specific size(which by default is 70%), the AISR engine will apply to the video path, otherwise it will apply to the GOP path.

## Definitions and Abbreviations

| Term   | Explanation |
|--------|-------------|
| `AISR` | AI Super Resolution |
| `OSD`  | On-Screen Display |
| `GOP`  | Graphics Output Protocol |
| `AOW`  | Android One World |
