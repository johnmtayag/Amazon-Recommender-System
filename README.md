<<<<<<< HEAD
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

The 30 minute dataset has 2,644,013,376 rows representing timestamped energy output measurements from various solar PV systems located across the UK. There are three columns:
1. **generation_wh**: The amount of energy outputted over 30 minutes in Wh (double)
2. **datetime**: The corresponding timestamp of when the measurement was made (timestamp_ntz)
3. **ss_id**: The solar PV system ID number (long)

<center>

|generation_wh|           datetime|ss_id|
|-------------|-------------------|-----|
|          0.0|2010-11-18 00:00:00| 2405|
|          0.0|2010-11-18 00:30:00| 2405|
|          0.0|2010-11-18 01:00:00| 2405|

</center>

The metadata dataset has 24,662 rows containing supplementary information on how each solar PV system was configured. There are eight columns:
1. **ss_id**: The solar PV system ID number (integer)
2. **latitude_rounded**: The latitude that the solar PV system is located (double)
3. **longitude_rounded**: The longitude that the solar PV system is located (double)
4. **llsoacd**: This variable is not defined in the source repo (string)
5. **orientation**: The direction angle from North that the solar PV system faces (double)
6. **tilt**: The tilt angle of the solar PV system (double)
7. **kwp**: The energy generation capacity of the solar PV system in kw (double)
8. **operational_at**: The date when the solar PV system was activated (date)

<center>

|ss_id|latitude_rounded|longitude_rounded|  llsoacd|orientation|tilt| kwp|operational_at|
|-----|----------------|-----------------|---------|-----------|----|----|--------------|
| 2405|           53.53|            -1.63|E01007430|      180.0|35.0|3.36|    2010-11-18|
| 2406|           54.88|            -1.38|E01008780|      315.0|30.0|1.89|    2010-12-03|
| 2407|           54.88|            -1.38|E01008780|      225.0|30.0|1.89|    2010-12-03|

</center>

## Setting Up the Environment
We utilize the following non-standard Python libraries in our analysis - these need to be set up via pip install or conda install methods.
* ipyleaflet

# Methods

## Exploring the Metadata Dataset

### Null Values
There are 36 null values in the metadata dataset:

<center>

|ss_id|latitude_rounded|longitude_rounded|  llsoacd|orientation|tilt| kwp|operational_at|
|-----|----------------|-----------------|---------|-----------|----|----|--------------|
|    0|               0|                0|        0|          0|   0|   0|            36|


</center>

### Exploring the variables: kwp

\[Need to makes these smaller]

![Distribution of kwp](images/kwp_distribution.png)
![Hist of kwp without outliers](images/hist_kwp_no_outliers.png)
![Hist of kwp](images/hist_kwp.png)

### Exploring the variables: operational_at
![temp](images/monthly_num_systems_coming_online.png)
![temp](images/num_systems_coming_online.png)

### Exploring the variables: tilt
![temp](images/tilt_distribution.png)

### Exploring the variables: orientation
![temp](images/orientation_distribution.png)

### Exploring the variables: Mapping
![temp](images/map_system_locations.png)
![temp](images/map_by_kwp.png)
![temp](images/map_by_orientation.png)
![temp](images/map_by_tilt.png)


## Exploring the 30 Minute Dataset
There are 1,824,316 null values in the 30 minute dataset:

<center>

|numNulls_generation|numNulls_timestamp|numNulls_ss_id|
|-------------------|------------------|--------------|
|            1824316|                 0|             0|

</center>>

### Exploring the variables

  
## Preprocessing

Steps required for preprocessing:
* Filter out systems with kwp > 50 
* Convert energy output to power generation rate
* Filter out timestamps associated with systems on dates that have fewer than 48 timestamps
* For each ss_id/date grouping with 48 timestamps, get a list of power_kW ordered by timestamp
* Perform eigendecomposition to get coefficients to parametrize each curve with sinusoids

### Note: Reasons are included here for later reference --> Move them to Discussion

### Preprocessing: Filtering out High-Capacity Solar PV Systems
Exploring the metadata reveals the presence of several solar PV systems with notably high outlier power generation ratings. The goal of this analysis is to detect anomalies both within a system's daily curves and across all systems. As there are not enough high-capacity systems to make meaningful comparisons between, all systems with a capacity higher than 50 kW are removed from both the 30 minute dataset and the metadata dataset.



### Preprocessing: Converting Energy Output to Power Generation Rate
The "generation_wh" column of the 30 minute dataset gives the amount of Watts generated in the last 30 minutes for a given solar PV system at a given timestamp. However, each solar PV system is associated with a value in the "kwp" column of the metadata which is the power generation capacity of the system in kW. In order to compare these values, the energy outputs in W are converted to average power generated in kW with the following formula:

\[CONVERT TO FORMULA]
power_kW = generation_wh * ((60/30) * (1/1000))

This formula transforms each value in "generation_wh" from the amount of Watts generated in the last 30 minutes to the average power generated over the same 30 minute interval. This new value is saved as "power_kW"

### Preprocessing: Removing Missing Data Points
The energy output of each solar PV system is aggregated and reported at 30 minute intervals, and ideally, each solar PV system would have 48 timestamped reports for each day. Due to the coarse-grained nature of these measurements, any missing data points can greatly affect the shapes of the fitted models. Thus, in order to parameterize the power generation curves as accurately as possible, we need to minimize the number of missing data points.

There are two main categories of missing data points:
1. For a given solar PV system at a given timestamp (ex: 04/24/2019 12:30:00), the energy output was reported as NULL
  1. These were removed in a previous step.
2. For a given solar PV system on a given day, fewer than 48 timestamps exist in the dataset

Out of all pairings of ID-Date, only 144,730 had fewer than 48 timestamps - these were all removed.

### Preprocessing: Collecting Timestamp Groupings
\[Edit code to sort the timestamped values by timestamp so they're in order!]

## Parameterizing the Daily Power Generation Curves with the Fourier Transform
Perform eigenvalue decomposition

## Model 1
KMeans to identify clusters? --> Likely this will group the low kwp systems and higher systems

## Model 2
Isolation forest? Random forest method
Local Outlier Factor?
Distance between parameters and the average parameter values?

[Reference for algorithm descriptions](https://www.datacamp.com/tutorial/introduction-to-anomaly-detection)

# Results

# Discussion
=======
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

The 30 minute dataset has 2,644,013,376 rows representing timestamped energy output measurements from various solar PV systems located across the UK. There are three columns:
1. **generation_wh**: The amount of energy outputted over 30 minutes in Wh (double)
2. **datetime**: The corresponding timestamp of when the measurement was made (timestamp_ntz)
3. **ss_id**: The solar PV system ID number (long)

<center>

|generation_wh|           datetime|ss_id|
|-------------|-------------------|-----|
|          0.0|2010-11-18 00:00:00| 2405|
|          0.0|2010-11-18 00:30:00| 2405|
|          0.0|2010-11-18 01:00:00| 2405|

</center>

The metadata dataset has 24,662 rows containing supplementary information on how each solar PV system was configured. There are eight columns:
1. **ss_id**: The solar PV system ID number (integer)
2. **latitude_rounded**: The latitude that the solar PV system is located (double)
3. **longitude_rounded**: The longitude that the solar PV system is located (double)
4. **llsoacd**: This variable is not defined in the source repo (string)
5. **orientation**: The direction angle from North that the solar PV system faces (double)
6. **tilt**: The tilt angle of the solar PV system (double)
7. **kwp**: The energy generation capacity of the solar PV system in kw (double)
8. **operational_at**: The date when the solar PV system was activated (date)

<center>

|ss_id|latitude_rounded|longitude_rounded|  llsoacd|orientation|tilt| kwp|operational_at|
|-----|----------------|-----------------|---------|-----------|----|----|--------------|
| 2405|           53.53|            -1.63|E01007430|      180.0|35.0|3.36|    2010-11-18|
| 2406|           54.88|            -1.38|E01008780|      315.0|30.0|1.89|    2010-12-03|
| 2407|           54.88|            -1.38|E01008780|      225.0|30.0|1.89|    2010-12-03|

</center>

## Setting Up the Environment
We utilize the following non-standard Python libraries in our analysis - these need to be set up via pip install or conda install methods.
* ipyleaflet

# Methods

## Exploring the Metadata Dataset

### Null Values
There are 36 null values in the metadata dataset:

<center>

|ss_id|latitude_rounded|longitude_rounded|  llsoacd|orientation|tilt| kwp|operational_at|
|-----|----------------|-----------------|---------|-----------|----|----|--------------|
|    0|               0|                0|        0|          0|   0|   0|            36|


</center>

### Exploring the variables: kwp

\[Need to makes these smaller]

![Distribution of kwp](images/kwp_distribution.png)
![Hist of kwp without outliers](images/hist_kwp_no_outliers.png)
![Hist of kwp](images/hist_kwp.png)

### Exploring the variables: operational_at
![temp](images/monthly_num_systems_coming_online.png)
![temp](images/num_systems_coming_online.png)

### Exploring the variables: tilt
![temp](images/tilt_distribution.png)

### Exploring the variables: orientation
![temp](images/orientation_distribution.png)

### Exploring the variables: Mapping
![temp](images/map_system_locations.png)
![temp](images/map_by_kwp.png)
![temp](images/map_by_orientation.png)
![temp](images/map_by_tilt.png)


## Exploring the 30 Minute Dataset
There are 1,824,316 null values in the 30 minute dataset:

<center>

|numNulls_generation|numNulls_timestamp|numNulls_ss_id|
|-------------------|------------------|--------------|
|            1824316|                 0|             0|

</center>>

### Exploring the variables

  
## Preprocessing

Steps required for preprocessing:
* Filter out systems with kwp > 50 
* Convert energy output to power generation rate
* Filter out timestamps associated with systems on dates that have fewer than 48 timestamps
* For each ss_id/date grouping with 48 timestamps, get a list of power_kW ordered by timestamp
* Perform eigendecomposition to get coefficients to parametrize each curve with sinusoids

### Note: Reasons are included here for later reference --> Move them to Discussion

### Preprocessing: Filtering out High-Capacity Solar PV Systems
Exploring the metadata reveals the presence of several solar PV systems with notably high outlier power generation ratings. The goal of this analysis is to detect anomalies both within a system's daily curves and across all systems. As there are not enough high-capacity systems to make meaningful comparisons between, all systems with a capacity higher than 50 kW are removed from both the 30 minute dataset and the metadata dataset.



### Preprocessing: Converting Energy Output to Power Generation Rate
The "generation_wh" column of the 30 minute dataset gives the amount of Watts generated in the last 30 minutes for a given solar PV system at a given timestamp. However, each solar PV system is associated with a value in the "kwp" column of the metadata which is the power generation capacity of the system in kW. In order to compare these values, the energy outputs in W are converted to average power generated in kW with the following formula:

\[CONVERT TO FORMULA]
power_kW = generation_wh * ((60/30) * (1/1000))

This formula transforms each value in "generation_wh" from the amount of Watts generated in the last 30 minutes to the average power generated over the same 30 minute interval. This new value is saved as "power_kW"

### Preprocessing: Removing Missing Data Points
The energy output of each solar PV system is aggregated and reported at 30 minute intervals, and ideally, each solar PV system would have 48 timestamped reports for each day. Due to the coarse-grained nature of these measurements, any missing data points can greatly affect the shapes of the fitted models. Thus, in order to parameterize the power generation curves as accurately as possible, we need to minimize the number of missing data points.

There are two main categories of missing data points:
1. For a given solar PV system at a given timestamp (ex: 04/24/2019 12:30:00), the energy output was reported as NULL
  1. These were removed in a previous step.
2. For a given solar PV system on a given day, fewer than 48 timestamps exist in the dataset

Out of all pairings of ID-Date, only 144,730 had fewer than 48 timestamps - these were all removed.

### Preprocessing: Collecting Timestamp Groupings
\[Edit code to sort the timestamped values by timestamp so they're in order!]

## Parameterizing the Daily Power Generation Curves with the Fourier Transform
Perform eigenvalue decomposition

## Model 1
KMeans to identify clusters? --> Likely this will group the low kwp systems and higher systems

## Model 2
Isolation forest? Random forest method
Local Outlier Factor?
Distance between parameters and the average parameter values?

[Reference for algorithm descriptions](https://www.datacamp.com/tutorial/introduction-to-anomaly-detection)

# Results

# Discussion
>>>>>>> 6404787b098194615b56f0d36f44d22f130f424c
