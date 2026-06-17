# Process Capability Analysis in Power BI

This guide describes the DAX measures used to calculate **Process Capability** statistics in Power BI as part of a Statistical Process Control (SPC) dashboard.

Process capability analysis evaluates how consistently a process produces results within defined specification limits by comparing the overall process variation to the allowable tolerance range. The measures in this document calculate the long-term capability indices (**Pp**, **Ppk**, **Ppl**, and **Ppu**) along with expected nonconformance rates based on the normal distribution.

These DAX measures are intended to complement the Individuals and Moving Range (I-MR) control chart by providing insight into whether a process is capable of consistently meeting customer or engineering specifications.

---

## Overview

Process capability analysis in Statistical Process Control (SPC) quantifies how consistently a process produces output within specification limits by comparing the natural variation of the process to the allowable tolerance range.

Process capability **P metrics** (Pp, Ppk, Ppu, and Ppl) describe process performance using **overall (long-term) variation**, rather than within-subgroup variation used by Cp/Cpk.

- **Pp** measures the overall process spread relative to the specification limits.
- **Ppk** measures the actual process capability by accounting for process centering.
- **Ppl** evaluates capability relative to the lower specification limit (LSL).
- **Ppu** evaluates capability relative to the upper specification limit (USL).

Together, these metrics indicate both the amount of process variation and whether the process is centered between the specification limits.

---

# Base Statistics

### Estimated Mean

```DAX
Estimated Mean =
AVERAGE('Facts'[Value])
```

### Sigma (Overall Standard Deviation)

```DAX
Sigma =
STDEV.S('Facts'[Value])
```

### Sample Size

```DAX
N =
COUNT('Facts'[Value])
```

---

# Process Capability Metrics

## Pp

Measures the overall capability of the process assuming it is centered between the specification limits.

```DAX
Pp =
VAR sigma = STDEV.S('Facts'[Value])
VAR LSL = MIN('Parameter'[LSL])
VAR USL = MAX('Parameter'[USL])

RETURN
IF(
    NOT(ISBLANK(LSL)) &&
    NOT(ISBLANK(USL)),
    (USL - LSL) / (6 * sigma)
)
```

---

## Ppk

Measures actual process capability by accounting for both process variation and process centering.

```DAX
Ppk =
VAR pplower =
    IF(
        NOT(ISBLANK(MIN('Parameter'[LSL]))),
        ([Estimated Mean] - MIN('Parameter'[LSL])) / (3 * [Sigma])
    )

VAR ppupper =
    IF(
        NOT(ISBLANK(MIN('Parameter'[USL]))),
        (MIN('Parameter'[USL]) - [Estimated Mean]) / (3 * [Sigma])
    )

RETURN
IF(
    NOT(ISBLANK(MIN('Parameter'[LSL]))) &&
    NOT(ISBLANK(MAX('Parameter'[USL]))),
    MIN(pplower, ppupper),
    IF(
        NOT(ISBLANK(MIN('Parameter'[LSL]))),
        pplower,
        ppupper
    )
)
```

---

## Ppl

Measures capability relative to the lower specification limit.

```DAX
Ppl =
IF(
    NOT(ISBLANK(MIN('Parameter'[LSL]))),
    ([Estimated Mean] - MIN('Parameter'[LSL])) / (3 * [Sigma])
)
```

---

## Ppu

Measures capability relative to the upper specification limit.

```DAX
Ppu =
IF(
    NOT(ISBLANK(MIN('Parameter'[USL]))),
    (MIN('Parameter'[USL]) - [Estimated Mean]) / (3 * [Sigma])
)
```

---

# Expected Nonconformance

The Z-value measures below are helper calculations used only to estimate the expected percentage of observations outside the specification limits. These measures do **not** need to be displayed in report visuals and can be hidden within the semantic model.

## Lower Z-Value

```DAX
_zlower =
[Ppl] * 3
```

## Upper Z-Value

```DAX
_zupper =
[Ppu] * 3
```

---

## Expected Above USL (%)

Calculates the expected percentage of observations above the upper specification limit assuming a normal distribution.

```DAX
Expected Above USL (%) =
IF(
    [_zupper] = BLANK(),
    0.00,
    IF(
        [_zupper] < 4,
        (1 - NORM.S.DIST([_zupper], TRUE)) * 100,
        0
    )
)
```

---

## Expected Below LSL (%)

Calculates the expected percentage of observations below the lower specification limit assuming a normal distribution.

```DAX
Expected Below LSL (%) =
IF(
    [_zlower] = BLANK(),
    0.00,
    IF(
        [_zlower] < 4,
        (1 - NORM.S.DIST([_zlower], TRUE)) * 100,
        0
    )
)
```

---

## Expected Outside Specification Limits (%)

Calculates the total expected percentage of observations falling outside either specification limit.

```DAX
Expected Outside Spec Limits (%) =
[Expected Above USL (%)] +
[Expected Below LSL (%)]
```

---

# Visual Title

Generates a dynamic histogram title based on the selected parameter.

```DAX
Histogram Title =
VAR selected_param =
    SELECTEDVALUE('Parameter'[Parameter], "Parameter")

RETURN
selected_param & " Histogram"
```
