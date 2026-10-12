# Aviation Accidents Analysis

## Project Overview

This project analyzes historical aviation accident data from 1948 to 2023 to help an airline and aircraft insurance client identify aircraft makes and models associated with lower rates of aircraft destruction and fewer fatal or serious injuries when accidents occur.

The analysis focuses on records from **1983 onwards** and excludes amateur-built aircraft. It compares small and large aircraft, examines injury outcomes, and investigates the conditions associated with more severe accidents.

The goal is to support insurance risk assessment by identifying aircraft types that may warrant further consideration based on their recorded accident outcomes.

## Business Questions

- Which aircraft makes and models have the lowest fatal and serious injury fractions?
- Which aircraft makes and models have the lowest destruction rates?
- How do safety outcomes differ between small and large aircraft?
- How do injury outcomes vary across accidents involving the same aircraft type?
- Which aircraft characteristics and accident conditions are associated with more severe outcomes?

## Tools and Technologies

- **Python:** Data processing and analysis
- **Pandas:** Data cleaning, transformation, grouping, filtering, and aggregation
- **NumPy:** Handling missing values and numerical operations
- **Matplotlib:** Data visualization
- **Seaborn:** Statistical plots, including bar plots, violin plots, and strip plots
- **Jupyter Notebook:** Interactive analysis and documentation

## Repository Structure

```text
aviation-accidents-analysis/
├── data/
│   ├── AviationData.csv
│   └── cleaned_aviation_data.csv
├
│   Aviation_Accidents_Cleaning.ipynb
│   Aviation_Accidents_Data_Analysis.ipynb
└── README.md
```

## Data Cleaning and Preparation

The raw dataset was loaded from `./data/AviationData.csv` and cleaned before exploratory analysis.

### 1. Filtering the Data

- Restricted the analysis to accidents occurring from 1983 onwards.
- Excluded amateur-built aircraft using `Amateur.Built == "No"`.
- Standardized selected text columns and handled unknown categories.
- Filtered out aircraft makes with fewer than 50 recorded events for the make-level analysis.

### 2. Standardizing Text and Missing Values

Standardized fields such as `Make`, `Model`, `Engine.Type`, `Weather.Condition`, `Purpose.of.flight`, and `Broad.phase.of.flight`.

For example, engine types were stripped of extra spaces and standardized to title case. Values such as `"Unknown"`, `"None"`, and `"Unk"` were converted to missing values where appropriate.

```python
import numpy as np
import pandas as pd

df["Engine.Type"] = (
    df["Engine.Type"]
    .str.strip()
    .str.title()
    .replace({
        "Unknown": np.nan,
        "None": np.nan,
        "Unk": np.nan
    })
)
```

Unknown aircraft damage values were also treated as missing rather than being assumed to represent a particular damage outcome.

### 3. Creating Safety Metrics

**Total people involved**

Created `Total.People` by adding the numbers of uninjured, seriously injured, slightly injured, and fatally injured people.

```python
df["Total.People"] = (
    df["Total.Uninjured"]
    + df["Total.Serious.Injuries"]
    + df["Total.Minor.Injuries"]
    + df["Total.Fatal.Injuries"]
)
```

**Fatal and serious injury fraction**

Calculated the percentage of people involved who suffered fatal or serious injuries.

```python
df["pct_fatal_and_serious"] = (
    (
        df["Total.Fatal.Injuries"]
        + df["Total.Serious.Injuries"]
    )
    / df["Total.People"]
) * 100
```

The metric helps compare injury outcomes across accidents. A value of 0% means no recorded fatal or serious injuries among the people counted; 100% means all people counted had fatal or serious injuries.

**Aircraft destruction indicator**

Created `is_destroyed` to identify accidents where the aircraft was recorded as destroyed.

```python
df["Aircraft.damage"] = df["Aircraft.damage"].replace(
    "Unknown", np.nan
)

df["is_destroyed"] = df["Aircraft.damage"].map({
    "Destroyed": True,
    "Substantial": False,
    "Minor": False
})
```

Unknown or unmapped damage categories remain missing rather than being automatically classified as not destroyed.

**Combined aircraft identifier**

Combined the aircraft make and model into a single identifier to support model-level comparisons.

```python
df["Plane.Type"] = (
    df["Make"].astype("string").str.strip()
    + " "
    + df["Model"].astype("string").str.strip()
)
```

### 4. Removing Sparse Columns

Columns with more than 50% missing values were removed to reduce the impact of highly incomplete data.

These included:

- `Schedule`
- `Air.carrier`
- `FAR.Description`
- `Aircraft.Category`
- `Longitude`
- `Latitude`

The cleaned dataset was exported for the next stage of analysis.

```python
df.to_csv("./data/cleaned_aviation_data.csv", index=False)
```

## Exploratory Data Analysis

The second notebook loads the cleaned dataset and investigates injury outcomes and aircraft destruction rates.

### 1. Aircraft Size Classification

Aircraft were classified into two groups using `Total.People`:

- **Small:** 20 or fewer people involved.
- **Large:** More than 20 people involved.

````



### 2. Grouping and Aggregation

Used `groupby()` and `mean()` to calculate average fatal and serious injury fractions for each aircraft make and model.


For model-level comparisons, the analysis used `Plane.Type` to distinguish individual makes and models.


### 3. Injury Rate Visualizations

**Horizontal bar plots**

Compared the 15 aircraft makes with the lowest mean fatal and serious injury fractions for small and large aircraft.

```python
lowest_15 = (
    mean_injury
    .sort_values("pct_fatal_and_serious")
    .groupby("Size")
    .head(15)
)
````

**Violin plots**

Examined the distribution of injury fractions for selected small aircraft makes. Unlike a plot of averages alone, violin plots help reveal variation in accident outcomes.

**Strip plots**

Displayed individual accident observations for selected aircraft makes and models. These plots helped identify concentrations near 0% injury and occasional severe outcomes approaching 100%.

The plots use individual accident records, while the mean injury fractions summarize outcomes across those records.

### 4. Aircraft Destruction Rates

Compared destruction rates across aircraft makes and size categories using the `is_destroyed` indicator.

```python
destruction_rates = (
    df.groupby(["Size", "Make"])["is_destroyed"]
    .mean()
    .reset_index(name="destruction_rate")
)

destruction_rates["destruction_rate"] *= 100
```

Because `is_destroyed` is Boolean, its mean represents the proportion of non-missing damage records classified as destroyed. Multiplying by 100 converts that proportion to a percentage.

## Key Findings

### 1. Injury Risk by Aircraft Make

The analysis identified several makes with low average fatal and serious injury fractions.

- **Small aircraft:** Waco, Helio, and Grumman-Schweizer showed low mean injury fractions.
- **Large aircraft:** Aero Commander, Gulfstream, Swearingen, and Cessna showed low mean injury fractions.

At the model level, the Boeing 757 and McDonnell Douglas DC-10-10 appeared among the large aircraft models with low mean injury fractions. For small aircraft, examples included the Diamond DA20 C1, Grumman G164B, Cessna 180C, and several Piper models.

### 2. Aircraft Destruction

The analysis identified the following makes with low recorded destruction rates:

- **Small aircraft:** Balloon Works and Robinson Helicopter Company.
- **Large aircraft:** Aero Commander, Cessna, and Gulfstream.

### 3. Variation in Injury Outcomes

The distributional plots showed that many recorded accidents had injury fractions close to 0%, while some reached 100%.

This variation means that a low average injury fraction does not guarantee that every accident involving a particular aircraft type will have a mild outcome.

### 4. Recommendations for Insurers

- Consider both injury fractions and aircraft destruction rates when evaluating aircraft makes and models.
- Review the number of recorded events for each make and model before comparing their averages.
- Investigate aircraft types that perform well across both measures.
- Consider accident conditions and other relevant variables when assessing risk.
