# PQ Actor

## Overview

PQ Actor smoothly applies PQ settings according to AI scene confidence.

## Basic Knowledge

### The way to set PQ

```mermaid
%%{init: {"theme": "dark"}}%%
flowchart TB
    subgraph APP
        verifier@{ shape: circle, label: "PQ actor \nVerifier" }
        confidence@{ shape: doc, label: "PQ Scene \nConfidence" }
        json@{ shape: doc, label: "Pqactor.json" }
        actor[PQ Actor]
        verifier -.->|"To adjust PQ settings temporarily and immediately"| actor
        confidence --> actor
        json -->|PQ Settings| actor
    end
    subgraph Driver
        d[ ]
    end
    d -->|Get Histogram| actor
    actor -->|Set PQ Settings| d

```

PQ actor parses configuration files to get PQ settings after being enabled.
PQ actor sets up settings gradually for each scene type according to AI scene confidences and color histogram.
Also, PQ actor is set by the unit test called PQ Actor Verifier during run-time.
PQ Actor applies the PQ settings smoothly, called a transition.
Once the PQ actor gets the setting of the scene type, it applies PQ settings step by step.
The step size is adjustable in the configuration file.

### Use Cases of Scene Type in Scene Detection Model

| ID | Scene Type |
|----|------------|
| 0  | Green     |
| 1  | Sky    |
| 2  | Face      |
| 3  | Architecture |
| 4  | Food |
| 5  | Animation |
| 6  | Sport |
| 7  | Movie |

MTK implements PQ flow in PQ Actor for scene types such as green, sky,
face, architecture, and food. However, for scene types like animation, sport, and movie, customers are required to develop and implement their own PQ flow.

### Hue, Saturation and Luma Settings

The Hue, Saturation, and Luma settings are based on sections of Hue.
There are 14 sections in Hue:

| Index | Name |
|-------|------|
| 0     | Red  |
| 1     | Flesh-Red |
| 2     | Flesh |
| 3     | Flesh-Yellow |
| 4     | Yellow |
| 5     | Yellow-Cyan |
| 6     | Green |
| 7     | Green-Cyan |
| 8     | Cyan |
| 9     | Cyan-Blue |
| 10    | Blue |
| 11    | Blue-Magenta |
| 12    | Magenta |
| 13    | Magenta-Red |

The details of Hue, Saturation, and Luma shown in the following table.
The Format "S.15.16" represents that the parameter is signed, 15 bits of integer, and 16 bits of float

| Value | Desc | Default | Min | Max | Format |
|-------|------|---------|-----|-----|--------|
| Hue   | Rotate Degree | 0x0000 | 0xFFC40000(-60.0) | 0x3C0000(60.0) | S.15.16 |
| Saturation | Gain | 0x010000(1.0) | 0x000000(0.0) | 0x020000(2.0) | S.15.16 |
| Luma  | Gain | 0x010000(1.0) | 0x000000(0.0) | 0x020000(2.0) | S.15.16 |

### Sharpness Strength and Skin Protection Gain

| Value | Desc | Default | Min | Max |
|-------|------|---------|-----|-----|
| Sharpness | Gain | 26 | 0 | 63 |
| Skin | Gain | 31 | 0 | 255 |

### PQ Decision Tree

The dicision trees show how PQ Actor sets PQ settings according to scene confidence levels.

#### Green

```mermaid
%%{init: {"theme": "dark"}}%%
flowchart LR
    r[Green Confidence >= Threshold of Green]
    r -- Y --> n1[1.Face Confidence < Threshold of Face
    2.Architecture Confidence < Threshold of Architecture]
    r -- N --> o1[Not apply Green]
    n1 -- Y --> n2@{label: "Apply Green PQ(Rotate Hue, enhance Saturation in Turnkey)"}
    n1 -- N --> o2[Not apply Green]
```

#### Sky

```mermaid
%%{init: {"theme": "dark"}}%%
flowchart LR
    r[Sky Confidence >= Threshold of Sky]
    r -- Y --> n1@{label: "Apply Sky PQ(Enhance Saturation in Turnkey)"}
    r -- N --> o1[Not apply Sky]
```

#### Face

```mermaid
%%{init: {"theme": "dark"}}%%
flowchart LR
    r[Face Confidence >= Threshold of Face]
    r -- Y --> n1@{label: "Apply Face PQ(Enhance Skin Protection in Turnkey)"}
    r -- N --> o1[Not apply Face]
```

#### Architecture

```mermaid
%%{init: {"theme": "dark"}}%%
flowchart LR
    r[Architecture Confidence >= Threshold of Architecture]
    r -- Y --> n1[Face Confidence < Threshold of Face]
    r -- N --> o1[Not apply Architecture]
    n1 -- Y --> n2@{label: "Apply Architecture PQ(Enhance Sharpness in Turnkey)"}
    n1 -- N --> o2[Not apply Architecture]
```

## Introduction of Configuration File

The Configuration File is a type of JSON file. By modifying the Configuration File, users can easily change the PQ settings of each scene type.
After the PQ Actor is enabled, it will read file to store PQ settings first and then set up PQ settings.

### List of Keyords

| Keyword | Description | Note |
|---------|-------------|------|
| ColorHSYEnable | To enable HSY settings | Switch default TRUE(Enabled) |
| SharpnessEnable | To enable Sharpness gain settings | Switch default TRUE(Enabled) |
| SkinWeightEnable | To enable Skin gain settings | Switch default TRUE(Enabled) |
| Step | The step number for the transition of PQ settings | "Step":["Hue","Sat","Luma","Sharpness"] |
| ChangeToDefault | Default settings when the scene changes or not | Switch default TRUE(Enabled) |
| DelayMsToDefault | Timeout to default settings when scene changes | 0 |
| CheckTimeoutMs | The callback function that retrieves the AI confidence of scene types may experience a timeout when no scene changes| 200 |
| CheckValidTimeMs | Valid duration from callback function which informs to get the confidences of scene types to exactly get them | ["LowBound(100)","HighBound(900)"] |
| SceneChangeThreshold | Threshold of scene change | ["Hue(3800)","Sat(3800)","Luma(3800)"] The scene change becomes more sensitive when the value is lower |
| SkinThreshold | Threshold of skin detection | ["Group1(10000)","Group2(10000)"] |
| DefaultSetting | Default PQ settings |  |
| SceneSettings | PQ settings of scenes |  |
| Id | Scene type identification of scene settings |  |
| Sharpness | The sharpness strength of the scene settings | 26 |
| Skin | The skin weight of the scene settings | 31 |
| HSY | HSY values used for scene settings | "HSY":{"Num":14,"Hue":[...],"Sat":[...],"Luma":[...]} |
| Hue | Hue of HSY of scene settings | "Hue":[v0~v13] |
| Sat | Saturation of HSY of scene settings | "Sat":[v0~v13] |
| Luma | Luminance of HSY of scene settings | "Luma":[v0~v13] |

## Introduction to the PQ Actor Verifier

### Description

PQ actor are set by unit test called PQ Actor Verifier.
During run time, users can user unit tests to temporarily set PQ settings.
After determining the PA settings by unit test, users can store the PQ settings in a configuration file.

### Commands

```shell
vendor/bin/PQActor_Verifer
```