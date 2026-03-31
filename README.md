# F1 Driver vs Constructor Comparison Model

A data science project that quantifies Formula 1 driver skill and constructor strength using historical race data, then uses simulation to compare matchups and predict race outcomes under controlled conditions.

---

## Overview

This project explores a classic Formula 1 question:

**How much do drivers matter compared to cars, and can we quantify the difference?**

To answer this, we built a system that:

- evaluates **driver performance**
- evaluates **constructor (team/car) performance**
- models circuit-specific race pace
- simulates head-to-head race outcomes between different driver-car combinations

The goal is to create a practical comparison tool that can estimate who would win given:

- two drivers
- two cars / constructors
- a circuit

---

## Research Questions

This project focuses on two main questions:

1. **Can we quantify the skills, strengths, and abilities of Formula 1 drivers and constructors in a meaningful way?**
2. **Can those quantified scores be used to simulate and predict race outcomes?**

More specifically, we wanted to investigate:

- whether the **driver** or the **car** has a greater impact on race outcomes
- how much each contributes
- whether historical data can support realistic race simulation

---

## Dataset

The project uses the **Formula 1 World Championship dataset** from Kaggle, compiled by **Vopani**.

### Main tables used

- `races.csv`
- `results.csv`
- `qualifying.csv`
- `lap_times.csv`
- `pit_stops.csv`
- `drivers.csv`
- `constructors.csv`
- `circuits.csv`
- `status.csv`

These tables provide information about:

- race results
- qualifying performance
- lap-level pace
- pit stop performance
- driver identity
- constructor identity
- circuit metadata

---

## Feature Engineering

We designed features to capture different dimensions of performance.

### Driver-level features

- **Qualifying Score**  
  Measures one-lap pace by comparing a driver’s best qualifying lap to the field average using a z-score.

- **Consistency Score**  
  Measures how stable a driver’s lap times are during races.

- **Racecraft Score**  
  Measures positions gained from grid position to finishing position.

### Constructor-level features

- **Finishing Performance Relative to Field**
- **Lap Pace Relative to Field**
- **Pit Stop Performance Relative to Field**

---

## Data Preprocessing

The raw data required substantial cleaning and standardization before modeling.

### Main preprocessing steps

- merged performance tables with race, driver, constructor, and circuit metadata
- converted lap times and qualifying times into numeric millisecond format
- removed rows with missing or invalid qualifying times
- excluded **lap 1** from race-lap analysis due to start-lap noise
- filtered out abnormal lap times using the **interquartile range (IQR)** method
- handled missing values with **mean imputation**

These steps helped reduce noise caused by:

- pit stops
- traffic
- safety cars
- damage
- strategy variation
- timing anomalies

---

## Modeling Approach

We used a **Random Forest Regressor** for both the driver model and the constructor model.

### Why Random Forest?

- handles nonlinear relationships well
- robust to moderate feature correlation
- does not require strong distributional assumptions
- provides interpretable **feature importance**

### Driver model

**Inputs:**
- qualifying score
- consistency score
- racecraft score

**Target:**
- average points per race

The predicted values were rescaled into a **driver score out of 100**.

### Constructor model

**Inputs:**
- finishing performance relative to field
- lap pace relative to field
- pit stop performance relative to field

**Target:**
- average team points per race

The predicted values were rescaled into a **constructor score out of 100**.

---

## Simulation Framework

After generating driver and constructor scores, we used them in a race simulation framework.

### Simulation idea

For each circuit:

1. estimate a baseline lap-time distribution from historical data
2. adjust the mean lap time using:
   - driver pace effects
   - constructor effects
3. adjust the variance using driver consistency
4. model lap times as normal distributions
5. simulate races by repeatedly sampling lap times over race distance

This allows head-to-head comparisons between different driver-car combinations on a selected circuit.

---

## Evaluation

We evaluated the model using a **pairwise accuracy test**.

### Evaluation method

For every race in a selected season:

- compare every pair of drivers in that race
- predict which driver should be stronger based on estimated lap-time distribution mean
- compare the prediction to actual finishing order

A prediction is counted as correct if the predicted stronger driver finishes ahead of the other driver.

This is a good evaluation strategy because the main purpose of the project is **head-to-head comparison**, not just overall ranking.

---

## Results

The model performed reasonably well on pairwise driver comparison.

### Overall accuracy (2018–2022)

- **70.4%**

### Year-by-year accuracy

- **2018:** 71.15%
- **2019:** 69.67%
- **2020:** 68.58%
- **2021:** 73.06%
- **2022:** 69.21%

### Interpretation

Since pairwise comparison is a binary task, a random baseline would be around **50%** accuracy.  
Achieving about **70%** suggests that the model captures meaningful performance patterns.

The drop in 2022 may reflect the introduction of new technical regulations, while the higher result in 2021 may be related to stability under the previous ruleset.

---

## Feature Importance

### Driver model

The most important factor was:

- **Qualifying ability / peak lap-time performance (~71%)**

Racecraft was the second most important, while consistency contributed less.

This suggests that one-lap pace is a strong indicator of underlying driver strength.

### Constructor model

Important factors included:

- lap-time performance
- race result performance
- pit stop quality

This indicates that constructor strength depends not only on raw car speed, but also on operational execution and race strategy.

---

## Challenges

Several challenges affected the project:

### 1. Noisy race data
Lap times are influenced by many external factors such as:
- pit stops
- traffic
- tire strategy
- safety cars
- race incidents

### 2. Missing values in older seasons
Some historical races have incomplete data, especially for lap times and other detailed records.

### 3. Potential overfitting
The model was trained on broad historical data, while evaluation seasons were also part of that historical range.

To reduce overfitting risk, we constrained the Random Forest by:
- limiting maximum tree depth
- setting a minimum number of samples per leaf

---

## Future Improvements

If given more time, we would improve the project in several ways:

1. **Use a richer dataset**  
   Include more detailed tire compound data, weather, safety car intervals, and other race events.

2. **Adopt a stricter train/test split**  
   For example, split data within each regulation era instead of training on the full historical dataset.

3. **Compare multiple models**  
   Test alternatives such as:
   - linear regression
   - gradient boosting
   - other ensemble methods

4. **Tune simulation parameters more systematically**  
   Replace manually assigned weights with more data-driven calibration.

---
