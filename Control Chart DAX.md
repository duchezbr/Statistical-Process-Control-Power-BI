# Individuals and Moving Range (I-MR) Control Chart in Power BI

An **Individuals and Moving Range (I-MR) Control Chart** is a Statistical Process Control (SPC) tool used to monitor process stability and detect special-cause variation when data are collected one observation at a time. The **Individuals Chart** tracks individual process measurements relative to statistically derived control limits, while the **Moving Range Chart** monitors the variation between consecutive observations to assess process consistency.

This guide describes the DAX measures used to build an I-MR Control Chart in Power BI using native visuals and calculations. The measures calculate process statistics, generate control limits, identify out-of-control observations, dynamically adjust chart scaling, monitor short-term process variability using rolling standard deviations, and highlight signals that warrant further investigation.

---

# Individual Control Chart

## Value

```DAX
Value =
AVERAGE('Facts'[Value])
```

---

## Mean

```DAX
Mean =
CALCULATE(
    AVERAGE(Facts[Value]),
    REMOVEFILTERS(Facts[X-Axis])
)
```

---

## Standard Deviation

```DAX
Standard Deviation =
CALCULATE(
    STDEV.P(Facts[Value]),
    REMOVEFILTERS(Facts[X-Axis])
)
```

---

# Monitoring Process Variability

These measures evaluate short-term process variability by calculating the standard deviation over rolling seven-batch windows.

Displaying these measures as KPI cards allows users to quickly determine whether process variability is increasing or decreasing without manually interpreting the control chart. The rolling window size can be adjusted to fit the manufacturing process by modifying the number of batches used in the calculations.

## Rolling 7-Batch Standard Deviation

Calculates the standard deviation of the seven most recent completed batches.

```DAX
Rolling 7-batch SD =
VAR current_date = MAX('Facts'[DoM])

VAR window_table =
    TOPN(
        7,
        FILTER(
            ALLSELECTED(Facts),
            Facts[DoM] < current_date
        ),
        Facts[DoM],
        DESC
    )

RETURN
CALCULATE(
    STDEVX.P(window_table, Facts[Value])
)
```

---

## Previous Rolling 7-Batch Standard Deviation

Calculates the standard deviation for the seven batches immediately preceding the current rolling window.

```DAX
Rolling Previous 7-batch SD =
VAR current_date =
    MAX(Facts[DoM])

VAR window_table =
    TOPN(
        14,
        FILTER(
            ALLSELECTED(Facts),
            Facts[DoM] < current_date
        ),
        Facts[DoM],
        DESC
    )

VAR previous7 =
    EXCEPT(
        window_table,
        TOPN(
            7,
            window_table,
            Facts[DoM],
            DESC
        )
    )

RETURN
STDEVX.P(
    previous7,
    Facts[Value]
)
```

---

## Rolling Standard Deviation Change (%)

Calculates the percent change in variability between the current and previous seven-batch windows.

Positive values indicate increasing process variation, while negative values indicate decreasing variation.

```DAX
Rolling SD Change (%) =
DIVIDE(
    [Rolling 7-batch SD] - [Rolling Previous 7-batch SD],
    [Rolling Previous 7-batch SD]
)
```

---

# Control Limits

## Individual Upper Control Limit (UCL)

Calculates the upper control limit using the process mean plus three standard deviations.

```DAX
Individual UCL =
[Mean] + (3 * [Standard Deviation])
```

---

## Individual Lower Control Limit (LCL)

Calculates the lower control limit using the process mean minus three standard deviations.

```DAX
Individual LCL =
[Mean] - (3 * [Standard Deviation])
```

---

---

# Highlight Nelson Rule 1 Violations

Create an **Individual Outlier Value** measure.

Add both **Value** and **Individual Outlier Value** to the **Y-Axis** of the Line Chart.

Use the **Format pane** to modify the marker size and color of the outlier series.

```DAX
Individual Outlier Value =
VAR CurrentValue =
    SELECTEDVALUE(Facts[Value])

VAR UCL_Value =
    CALCULATE(
        [Individual UCL],
        REMOVEFILTERS(Facts[X-Axis])
    )

VAR LCL_Value =
    CALCULATE(
        [Individual LCL],
        REMOVEFILTERS(Facts[X-Axis])
    )

RETURN
    IF(
        CurrentValue > UCL_Value
            || CurrentValue < LCL_Value,
        CurrentValue,
        BLANK()
    )
```

---

# Specification Limits

Create constant lines for the upper and lower specification limits.

## Upper Specification Limit

```DAX
Upper Spec =
VAR _val = SELECTEDVALUE('Parameter'[USL])
RETURN
IF(
    NOT ISBLANK(_val),
    _val
)
```

---

## Lower Specification Limit

```DAX
Lower Spec =
VAR _val = SELECTEDVALUE('Parameter'[LSL])
RETURN
IF(
    NOT ISBLANK(_val),
    _val
)
```

---

# Dynamic Y-Axis Range

These measures ensure that the specification limits and all data points remain visible.

## Y-Axis Minimum

```DAX
Y-Axis Min =
VAR _lsl = SELECTEDVALUE('Parameter'[LSL])
VAR _dataMin = MINX(ALLSELECTED(Facts), [Value])

RETURN
IF(
    NOT ISBLANK(_lsl),
    MIN(_lsl, _dataMin) * 0.98,
    _dataMin * 0.98
)
```

---

## Y-Axis Maximum

```DAX
Y-Axis Max =
VAR _usl = SELECTEDVALUE('Parameter'[USL])
VAR _dataMax = MAXX(ALLSELECTED(Facts), [Value])

RETURN
IF(
    NOT ISBLANK(_usl),
    MAX(_usl, _dataMax) * 1.02,
    _dataMax * 1.02
)
```

---

# Moving Range Control Chart

The Moving Range chart plots the absolute difference between consecutive observations.

Add both **Moving Range** and **Moving Range Outlier Value** to the **Y-Axis** of a Line Chart.

## Moving Range

```DAX
Moving Range =
VAR current_index =
    MAX(Facts[X-Axis])

VAR current_value =
    CALCULATE(
        MAX(Facts[Value])
    )

VAR previous_value =
    CALCULATE(
        MAX(Facts[Value]),
        Facts[X-Axis] = current_index - 1
    )

RETURN
IF(
    ISBLANK(previous_value),
    BLANK(),
    ABS(current_value - previous_value)
)
```

---

# Highlight Moving Range Outliers

Use the **Format pane** to modify the marker size and color of the outlier series.

```DAX
Moving Range Outlier Value =
VAR mr = [Moving Range]
VAR ucl = [Moving Range UCL]
VAR lcl = 0

RETURN
IF(
    ISBLANK(mr),
    BLANK(),
    IF(
        mr > ucl || mr < lcl,
        mr,
        BLANK()
    )
)
```

---

# Moving Range Center Line

Create a constant line using the Moving Range Average.

```DAX
Moving Range Average =
VAR current_index =
    MAX('Facts'[X-Axis])

RETURN
AVERAGEX(
    FILTER(
        ALL(Facts[X-Axis]),
        NOT ISBLANK([Moving Range])
    ),
    [Moving Range]
)
```

---

# Moving Range Upper Control Limit &  Lower Control Limit

```DAX
Moving Range UCL =
3.267 * [Moving Range Average]
```

The Lower Control Limit (LCL) for a Moving Range chart can be hardcoded to 0.


---

# Moving Range Outlier Count

This measure counts the number of Moving Range points outside the control limits.

```DAX
Moving Range Outlier Count =
VAR ucl = [Moving Range UCL]

RETURN
COUNTROWS(
    FILTER(
        VALUES(Facts[X-Axis]),
        VAR mr = [Moving Range]

        RETURN
            NOT ISBLANK(mr)
                && (mr > ucl || mr < 0)
    )
)
```
