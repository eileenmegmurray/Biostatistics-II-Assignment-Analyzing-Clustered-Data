# Analyzing Clustered Data: Reproducing Moen et al. (2016)

**Course:** Biostatistics II (BIOS621, Assignment 4), CUNY Graduate School of Public Health and Health Policy
**Author:** Eileen M. Murray, MPH
**Date:** December 2025
**Language:** R (R Markdown)

> This was completed as a course assignment. It is meant to serve as an exercise for analyzing clustered data.

## Overview

Neuroscience experiments often measure many neurons from each mouse, which means the observations are clustered: neurons from the same mouse tend to be more alike than neurons from different mice. Treating each neuron as an independent observation can make a result look statistically significant when it isn't.

This project reproduces Table 3 from [Moen et al. (2016)](https://journals.plos.org/plosone/article?id=10.1371/journal.pone.0146721), *Analyzing Clustered Data: Why and How to Account for Multiple Observations Nested within a Study Participant?* (PLOS ONE). It fits the same treatment effect six different ways, from naive neuron-level regression to mixed-effect models, and compares how the conclusions change.

## Data

The dataset contains 1,142 neuron soma size measurements from 14 mice, from the experiment analyzed in Moen et al. (2016). Pten knockdown varies between neurons within each mouse, while fatty acid treatment varies only between mice.

| Variable | Description |
|---|---|
| `somasize` | Soma size of each neuron (outcome) |
| `fa` | Fatty acid treatment: 0 = no treatment, 1 = vehicle control, 2 = fatty acid |
| `pten` | 0 = control, 1 = Pten shRNA knockdown |
| `mouseid` | Mouse identifier (cluster) |

## Methods

1. **Exploring clustering.** Plotted soma size by mouse and ran a one-way ANOVA, which showed that mean soma size differs significantly between mice.
2. **Intraclass correlation.** Fit a random-intercept model to estimate the ICC, which was about 0.16. This means roughly 16% of the variation in soma size is between mice.
3. **Model assumptions.** Wrote out the regression model for each row of Table 3 and assessed which of its assumptions are likely violated.
4. **Reproducing Table 3.** Fit six models with `somasize ~ fa * pten`:
   - Neuron-level linear regression
   - Mouse-level regression on mean soma size, unweighted
   - Mouse-level regression on mean soma size, weighted by number of neurons
   - Marginal regression (GEE, exchangeable correlation, robust standard errors)
   - Fixed-effect regression with mouse indicators
   - Mixed-effect regression with a random intercept for mouse

## Key Findings

| Model | Fatty acid coefficient | 95% CI | p-value |
|:---|---:|:---|---:|
| Neuron-level regression | 3.48 | (0.96, 6.00) | 0.007 |
| Mouse-level, unweighted | 3.45 | (-3.53, 10.42) | 0.32 |
| Mouse-level, weighted | 3.48 | (-4.67, 11.63) | 0.39 |
| Marginal (GEE) | 2.63 | (-3.64, 8.91) | 0.41 |
| Fixed-effect | 6.15 | (2.40, 9.90) | 0.001 |
| Mixed-effect | 2.56 | (-5.03, 10.14) | 0.48 |

- The **neuron-level model** finds a significant fatty acid effect, but only because it treats 1,142 neurons as independent observations and underestimates the standard error.
- Once clustering by mouse is taken into account (**mouse-level, GEE, and mixed-effect models**), the fatty acid effect is no longer significant.
- The **fixed-effect estimate is not valid** for fatty acid treatment. Each mouse received only one treatment, so treatment is perfectly collinear with the mouse indicators, and the estimate depends on which indicator R drops.
- The **GEE and mixed-effect models** are the most appropriate choices for this study design.

## Repository Contents

| File | Description |
|---|---|
| `murraye_a4_bios621(1).Rmd` | R Markdown source code |
| `PtenAnalysisData.csv` | Dataset |

## Reproducing the Analysis

**R packages:** `readr`, `ggplot2`, `nlme`, `tidyverse`, `gee`

```r
install.packages(c("readr", "ggplot2", "nlme", "tidyverse", "gee"))
```

Place `PtenAnalysisData.csv` in the same folder as the `.Rmd` file and knit.

## Skills Demonstrated

Clustered and hierarchical data analysis · Linear mixed-effect models · Generalized estimating equations (GEE) · Intraclass correlation · Fixed-effect regression · Reproducing published results · R Markdown
