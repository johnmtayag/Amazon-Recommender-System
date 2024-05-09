# Detecting Anomalies in Solar Power Generation

Detecting Anomalies in Solar Power Generation by comparing generation curve profiles.

# Introduction
\[Abstract for now --> Build up on this with visuals]
In order to guarantee that energy consumption demands are met, it is important to ensure that vital power generation systems are functioning as intended. By utilizing a comprehensive time-series dataset featuring over 2.6 billion rows of 30-minute interval power generation data from over 20,000 solar photovoltaic (PV) systems and their respective panel configurations, sourced from the OpenClimateFix repository hosted on Hugging Face, we aim to detect outlier panels that exhibit abnormal power generation patterns. Our methodology involves using the Fourier Transform to parameterize daily power generation curves, then applying Principal Component Analysis (PCA) to identify principal components that capture the most variance between the shapes of the curves. We will analyze the distance between a given panel’s typical generation curve and the average curve allowing for the identification of both global and local irregularities. We can then use this data to identify potential solar panel configurations which are more prone toward anomalous power generation. By providing insights into power generation anomalies, this project enables companies to maintain optimal power generation within their PV systems, thereby contributing to the sustainability and efficiency of solar energy systems.

## Dataset
The dataset that we used in this analysis can be obtained [here](https://huggingface.co/datasets/openclimatefix/uk_pv) on HuggingFace. It is gated, so it requires an account to access.

There are five datasets available in this repository, but only 2 will be used in this analysis:
* **30min.parquet**: Contains data on solar PV generation from over 20,000 PV systems in the UK from 2010 to 2021
* **metadata.csv**: Provides supplemental information on the setup configurations of each PV system

## Setting Up the Environment
We utilize the following non-standard Python libraries in our analysis - these need to be set up via pip install or conda install methods.
* ipyleaflet

# Methods

## Data Exploration
* Completed tasks/visuals/stats
  * Tasks
    * Showing snippets of the datasets
      * .show(3) on 30 min dataset
      * .show(3) on meta dataset
    * Showing dataframe schemas (with column names, data types, nullable)
  * Stats
    * Find number of nulls in the 30 min dataset
    * Find number of date/ID groupings that did not have 48 timestamps
  * Visualizations
    * Plotting generation curves for 3 ss_ids on the same day to show variation between IDs
    * Plotting generation curves for 3 ss_ids throughout a week to show variations
    * Plotting outlier from kwp in the metadataset
    * Plotting maps
     * General map showing locations of solar stations in UK
     * Maps colored by certain values
      * by kwp, tilt, orientation
  
## Preprocessing
The data preprocessing step was split into the following stages:

# Note: Reasons are included here for later reference --> Move them to Discussion

### Preprocessing: Filtering out High-Capacity Solar PV Systems
Exploring the metadata reveals the presence of several solar PV systems with notably high outlier power generation ratings. The goal of this analysis is to detect anomalies both within a system's daily curves and across all systems. As there are not enough high-capacity systems to make meaningful comparisons between, all systems with a capacity higher than 50 kW are removed from both the 30 minute dataset and the metadata dataset.

\[Include histogram showing # kwp at x<=5, 5<x<=50, 50<x, or possibly just the higher ones since x<5 will dominate the ]

### Preprocessing: Converting Energy Output to Power Generation Rate
The "generation_wh" column of the 30 minute dataset gives the amount of Watts generated in the last 30 minutes for a given solar PV system at a given timestamp. However, each solar PV system is associated with a value in the "kwp" column of the metadata which is the power generation capacity of the system in kW. In order to compare these values, the energy outputs in W are converted to average power generated in kW with the following formula:

\[INSERT FORMULA HERE]
Multiply by (60/30) * (1/1000)

This formula transforms each value in "generation_wh" from the amount of Watts generated in the last 30 minutes to the average power generated over the same 30 minute interval. This new value is saved as "power_kW"

### Preprocessing: Removing Missing Data Points
The energy output of each solar PV system is aggregated and reported at 30 minute intervals, and ideally, each solar PV system would have 48 timestamped reports for each day. Due to the coarse-grained nature of these measurements, any missing data points can greatly affect the shapes of the fitted models. Thus, in order to parameterize the power generation curves as accurately as possible, we need to minimize the number of missing data points.

There are two main categories of missing data points:
1. For a given solar PV system at a given timestamp (ex: 04/24/2019 12:30:00), the energy output was reported as NULL
2. For a given solar PV system on a given day, fewer than 48 timestamps exist in the dataset

### Preprocessing: Collecting Timestamp Groupings
\[Edit code to sort the timestamped values by timestamp so they're in order!]

## Parameterizing the Daily Power Generation Curves with the Fourier Transform


## Model 1
KMeans to identify clusters? --> Likely this will group the low kwp systems and higher systems

## Model 2
Isolation forest? Random forest method
or Local Outlier Factor
Distance between parameters and the average parameter values?

[Reference for algorithm descriptions](https://www.datacamp.com/tutorial/introduction-to-anomaly-detection)

# Results

# Discussion
