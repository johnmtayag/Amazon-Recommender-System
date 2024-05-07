# Detecting Anomalies in Solar Power Generation

Detecting Anomalies in Solar Power Generation by comparing generation curve profiles.

## Introduction

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
  
## Preprocessing
* Steps taken
  * 30 min dataset
    * Extract date from timestamp as stamp_date
    * Extract minutes from 00:00:00 as stamp_time_m
      * This was mainly for graph purposes, but likely this should be switched to Number of timestamps from 00:00:00 for parameterizing purposes
    * Convert generation_wh to power_kW by multiplying by a conversion factor
      * generation_wh is W in a 30 min interval --> Multiply by (60/30) * (1/1000) to convert to kW
    * Remove null values from dataset
    * Removed rows from date/ID groupings that did not have 48 timestamps
      * The power is measured every 30 minutes --> There are 48 30 minute intervals in a single day
      * All rows for a given date and for a given ss_id are removed if there were not 48 timestamps present
      * This ensures that each curve profile being analyzed is complete
  * Meta dataset
    * Filter for only rows where ss_id exists in the 30 min dataset
    * Remove ss_id=26966: This has an extreme outlier value in kwp at 25 (next lowest is ~ 4)
