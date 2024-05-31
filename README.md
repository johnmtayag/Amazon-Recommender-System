<<<<<<< HEAD
# Detecting Anomalies in Solar Power Generation

Detecting Anomalies in Solar Power Generation by comparing generation curve profiles.

# Introduction
\[Abstract for now --> Build up on this with visuals]
In order to guarantee that energy consumption demands are met, it is important to ensure that vital power generation systems are functioning as intended. By utilizing a comprehensive time-series dataset featuring over 2.6 billion rows of 30-minute interval power generation data from over 20,000 solar photovoltaic (PV) systems and their respective panel configurations, sourced from the OpenClimateFix repository hosted on Hugging Face, we aim to detect outlier panels that exhibit abnormal power generation patterns. Our methodology involves using the Fourier Transform to parameterize daily power generation curves, then applying Principal Component Analysis (PCA) to identify principal components that capture the most variance between the shapes of the curves. We will analyze the distance between a given panel’s typical generation curve and the average curve allowing for the identification of both global and local irregularities. We can then use this data to identify potential solar panel configurations which are more prone toward anomalous power generation. By providing insights into power generation anomalies, this project enables companies to maintain optimal power generation within their PV systems, thereby contributing to the sustainability and efficiency of solar energy systems.

### Dataset
The dataset that we used in this analysis can be obtained [here](https://huggingface.co/datasets/openclimatefix/uk_pv) on HuggingFace. It is gated, so it requires an account to access.

There are five datasets available in this repository, but only 2 will be used in this analysis:
* **30min.parquet**: Contains timestamped power generation values from over 20,000 PV systems in the UK from 2010 to 2021
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

The metadata dataset has 24,662 rows containing supplementary information on how each solar PV system was configured. Notably, there are more PV systems identified in the metadata dataset than the number actually represented in the 30 minute dataset. There are eight columns:
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

### Setting Up the Environment
We utilize the following non-standard Python libraries in our analysis - these need to be set up via pip install or conda install methods.
* ipyleaflet --> [[Setup documentation]](https://ipyleaflet.readthedocs.io/en/latest/installation/index.html)

# Methods

## Exploring the Metadata Dataset

### Null Values
There are 36 null values in the metadata dataset, specifically in the operational_at column:

<center>

|ss_id|latitude_rounded|longitude_rounded|  llsoacd|orientation|tilt| kwp|operational_at|
|-----|----------------|-----------------|---------|-----------|----|----|--------------|
|    0|               0|                0|        0|          0|   0|   0|            36|


</center>

### Exploring the variables: kwp (Solar PV System Power Rating)
Most of the PV systems represented in this dataset have fairly low power ratings in the range of 2-4 kW. However, there are some notable outliers, mainly those with kwp well above 50 kW. 

<center>
<img src="images/kwp_distribution.png" alt="Distribution of kwp" width="500"/>

<img src="images/hist_kwp_no_outliers.png" alt="Hist of kwp without outliers" width="400"/>
<img src="images/hist_kwp.png" alt="Hist of kwp" width="400"/>
</center>

### Exploring the variables: tilt
Most systems are oriented such that the panels are tilted at about 30 degrees or 35 degrees. The distribution then tapers off fairly uniformly at 10 degrees and 50 degrees. Interestingly, there is a tendency toward angles divisible by 5 degrees noted by prominent spikes in the distribution.

<center>
<img src="images/tilt_distribution.png" alt="Tilt Distribution" width="500"/>
</center>

### Exploring the variables: orientation
Most systems are oriented such that the panels are oriented about 180 degrees from North. Considering that these systems are located in the UK which is, itself, located at a fairly high latitude, this direction may be maximizing the power generated in this region. However, there is also a notable cluster of systems oriented in the 0-50 degree range which bucks this trend.

<center>
<img src="images/orientation_distribution.png" alt="Orientation Distribution" width="500"/>
</center>

### Exploring the variables: Mapping via iPyLeaflet
These systems are plotted on a map using iPyLeaflet to identify any patterns between system location and configuration parameters. The PV systems in the dataset are mainly located  in England and Scotland and are particularly concentrated in populated regions. However, there are no obvious patterns between where a system is located and the other configuration parameters:

<center>
<img src="images/map_system_locations.png" alt="System Locations Map" width="300"/>
</center>


**Coloring Systems by System Power Rating**:

The majority of solar PV systems throughout the country have very low power output ratings. There are few with higher power output ratings, and these are located close to major population centers.

<center>
<img src="images/map_by_kwp.png" alt="Map by kWp" width="400"/>
</center>

**Coloring Systems by Panel Orientation**:

Most panels throughout the country are oriented at 180 degrees as identified earlier without any particular geographic trends.

<center>
<img src="images/map_by_orientation.png" alt="Map by Orientation" width="400"/>
</center>

**Coloring Systems by Panel Tilt**:

Most panels throughout the country are tilted at about 30 degrees as identified earlier without any particular geographic trends.

<center>
<img src="images/map_by_tilt.png" alt="Map by Tilt" width="400"/>
</center>


## Exploring the 30 Minute Dataset
There are 1,824,316 null values in the 30 minute dataset:

<center>

|numNulls_generation|numNulls_timestamp|numNulls_ss_id|
|-------------------|------------------|--------------|
|            1824316|                 0|             0|

</center>

**Number of Timestamped Entries per Year**

Though the dataset spans the range 2010 to 2021, the number of timestamped entries from each year varies greatly. Most of the timestamps in the dataset fall within the range 2015-2021, with the range 2017-2020 being particularly highly represented. 2010 and 2011 are both represented quite poorly, especially relative to all other years in the dataset.

<center>
  <img src="images/num_timestamps_yearly.png" alt="Number of Timestamped Entries per Year" width="400"/>
</center>
  
## Main Preprocessing
First, all systems outside of the interquartile range of the kwp variable (IQR: 2.28 <= kwp <= 3.42) are filtered out of the dataset. The energy generation values, measured as kW*(30 minutes) are converted into average power generation values, measured as kW. The timestamped data is then grouped by ss_id and date to get a series of sequences of power generation values representing the power generated by a single system over a single day. The dataset is then filtered further such that only groupings with 48 non-null timestamped power generation measurements are retained.

**The Formula Used to Convert the Energy Generation Values to Power Generation Values:**
```math
\text{power}_{\text{kW}} = \text{generation}_{\text{wh}} \times \left(\frac{60}{30}\right) \times \left(\frac{1}{1000}\right)
```

## Reconstructing the Daily Power Generation Curves with the Fourier Transform
The Fourier Transform is used to analyze and parameterize daily power generation curves. This method decomposes power generation data into simpler sinusoidal components, reducing the noise of the measurements and facilitating the identification of patterns and anomalies.

### Defining Constants and Basis
The process begins by defining the necessary constants and creating an orthonormal basis of sinusoids. This involves:
- Setting up 10 pairs of sinusoids at increasing frequencies along with a bias curve. This results in a total of 21 basis vectors.
- Collecting each grouping of 48 time interval measurements into an array for processing.

The basis vectors are constructed as follows:
1. Calculate a step size and formulate the basis vectors as a bias vector and cosine/sine function pairs with increasing frequency values.
2. Append the desired number of vectors to an array to prepare for matrix operations.

### Visualizing Basis Sinusoids
To understand how each sinusoid contributes to the overall model, we plot the first five basis sinusoids. This visualization helps in understanding how the individual components are used in the Fourier Transform. The first basis vector is a flat bias curve which helps capture the curve's height while the rest of the basis vectors are sinusoids of varying frequency to capture the complexity of the waveforms.

<p align="center">
  <img src="images/first_five_basis_sinusoids.png" alt="First 5 Basis Sinusoids" width="400"/>
</p>

### Comparing Projected Reconstructions to Original Curves
The power generation curves are approximated by projecting them onto the basis vectors. This comparison shows how increasing the number of basis vector projections makes the resulting reconstruction more accurate. For our analysis, all 21 basis vectors are used to reconstruct the curves as accurately as possible while still minimizing the overall noise in the measurements.

<p align="center">
  <img src="images/compare_power_curves_approx.png" alt="Comparison of Original and Approximated Curves" width="800"/>
</p>

## Anomaly Detection with Principal Component Analysis (PCA)
To aid in the identification of anomalous curves, PCA is used to reduce the dimensionality of the reconstructed power generation data to just 2 dimensions. This process involves several key steps, including computing the covariance matrix, performing eigenvalue decomposition, and projecting the data onto the top principal components.

1. **Compute Covariance Matrix of the Reconstructed Curves**
   * The covariance matrix for the given data column is computed, handling NaN values appropriately. To parallelize this operation, the following steps are taken:
     1. Each set of coefficients is grouped up into an array with a 1 inserted into the first element
     2. The outer product of each array is calculated along with the indices of each non-null element
     3. The outer product matrices are reduced by summing up each matrix as well as summing up the indices of non-null elements
     4. From this combined matrix, extract and compute the outer product average which is necessary to compute the covariance matrix
2. **Eigenvalue Decomposition**
   * The eigenvalues and eigenvectors are extracted from the covariance matrix via eigenvalue decomposition and ordered by eigenvalue magnitude. 
3. **Plotting Explained Variance**
   * The amount of variance explained by each eigenvector is visualized to understand the significance of each component.
4. **Projecting the Reconstructions**
   * The reconstructed power values are projected onto the top 2 principal components

After projecting the data, a series of cutoff values along the top two principal components are used to identify anomalies at different scales until the PCA method becomes less effective. The anomalies detected here are separated out of the original dataset and are marked as labeled anomalies for later supervised anomaly detection methods.

## Further Anomaly Detection Using Statistical Methods
After the initial PCA anomaly filtering, two different statistical approaches are used to identify anomalous ranges:

1. **Interquartile Range Method**
   * The first and third quartile values along both PC1 and PC2 are identified, and any points that lie outside of the following cutoff values along either PC1 and/or PC2 are marked as anomalies:
    1. Low Cutoff = Q1 - 1.5 x (Q3 - Q1)
    2. High Cutoff = Q3 + 1.5 x (Q3 - Q1)
2. **Z-Score Method**
   * Like the previous method, a series of cutoff values along both PC1 and PC2 are identified, and any points that lie outside of the boundaries along either PC1 and/or PC2 are marked as anomalies. This time, the cutoff values are defined using the mean and the sample standard deviation of the datatset:
    1. Low Cutoff = Mean - 3 x STD
    2. High Cutoff = Mean + 3 x STD

## Further Anomaly Detection Using Isolation Forests
An isolation forest is an unsupervised machine learning algorithm which utilizes binary trees to detect anomalies. The algorithm relies on the fact that data points located further away from the centers of distribution are more likely to be quickly separated from the rest of the points when splitting the variable space.

**Typical Isolation Forest Process**:
* For each tree:
    * Randomly sample a subset of the data points
    * Recursively split the space along random variables at random values until some stopping criterion (ex: until every leaf node contains 3 points at maximum)
    * Give each point a score which corresponds to how close its leaf node is to the root node
    * Average this score across multiple trees
 
However, Pyspark does not have any built-in isolation forest algorithms - instead, we use a customized ensemble of single-node binary trees that split the space with a randomized boundary line, and we record the average score for each point across all iterations.

**Process:**
* First get the minimum and maximum values of the two variables (here, PC1 and PC2)
* For each forest:
    * For each tree:
        * Pick two random points within the space and get the parameters of the line that connects them in standard form (Ax +  By + C = 0)
        * Split the points into two groups according to which side of the line they are on
        * Give each point a score which corresponds to the fraction of all points that lie on the same side of the line
        * To minimize data storage requirements, track the scores in a running total, then divide by the number of splits for the average scores for each tree.
* These results are averaged across trees, and across forests.

To help visualize this process, the following figure shows 4 randomly generated boundary lines along with the resulting scores assigned to each group.

<p align="center">
  <img src="images/iforest_example_boundary_lines.png" alt="Example Isolation Forest Boundary Lines" width="600"/>
</p>

This method relies on both randomness and the aggregated results of weak learner predictions, so there can be a lot of variability in results. Thus, averaging the results across multiple trees and even multiple forests is ideal. For this analysis, we take the average scores over 5 runs using 7 trees with 10 splits each to identify further anomalies.

With these scores, we plot the data once again along the principal component axes, but now we can color each point according to its outlier score. We also plot the change in the average reconstructed power curve as the outlier score decreases.

## Supervised Anomaly Detection Using Labeled Data

## Identifying Correlations Between System Configurations and Outlier Frequency
The ss_ids of the outliers identified using the above methods are then used to try and note any correlations between a given system configuration parameter and the rate of anomalous curve occurences. To achieve this, a series of histograms are generated to compare the configuration parameters of each group of identified anomalies with the rest of the data.

# Results

# Discussion

## Explaining the Main Preprocessing Steps:

### Preprocessing: Removing Outlier Solar PV Systems
There are several solar PV systems with notably high outlier power generation ratings, and since the goal of this analysis is to detect anomaly power generation curves, larger curves may end up as false-flags during the detection process. As there are not enough high-capacity systems to make meaningful comparisons, and since the vast majority of systems fall within a tiny subset of kwp values, the dataset was restricted to only systems where the power rating was within the Interquartile Region (between 2.28 and 3.42). This ensures that any detected anomalies will more-likely represent actual anomalies.

The below plot shows the distribution of principal components from the later analysis without removing outlier solar PV systems. More than 99% of the data points fall within the red bounding box, so in a way, all of the points outside of the box may represent anomalies. By limiting the data to systems with lower power values, the distribution shrinks, with any remaining anomalies becoming much more apparent. 

<p align="center">
  <img src="images/pc1pc2_by_maxpower.png" alt="PC Boundaries for Anomalies Without Filtering" width="500"/>
</p>

### Preprocessing: Converting Energy Output to Power Generation Rate
The "generation_wh" column of the 30 minute dataset gives the amount of Watts generated in the last 30 minutes for a given solar PV system at a given timestamp. However, each solar PV system is associated with a value in the "kwp" column of the metadata which is the power generation capacity of the system in kW. In order to more easily compare these values, the energy outputs in W*(30 minutes) are converted to average power generated in kW with the following formula:

```math
\text{power}_{\text{kW}} = \text{generation}_{\text{wh}} \times \left(\frac{60}{30}\right) \times \left(\frac{1}{1000}\right)
```

This formula transforms each value in "generation_wh" from the amount of Watts generated in the last 30 minutes to the average power generated over the same 30 minute interval. This new value is saved as "power_kW."

### Preprocessing: Collecting Timestamp Groupings
Each entry in the 30 minute dataset consists of the system ID, the timestamp, and the measured energy output. In order to analyze the power output throughout an entire day, the timestamps must first be collected into groups by both system ID and date. As the measurements are taken every 30 minutes, there should be, at most, 48 timestamps per system ID for every date.

### Preprocessing: Removing Missing Data Points
The energy output of each solar PV system is aggregated and reported at 30 minute intervals, and so, ideally, each solar PV system would have 48 timestamped reports for each day. Due to the coarse-grained nature of these measurements, any missing data points can greatly affect the shapes of the fitted models, leading to possible false flags. Thus, in order to parameterize the power generation curves as accurately as possible, we need to minimize the number of missing data points.

There are two main categories of missing data points:
1. For a given solar PV system at a given timestamp (ex: 04/24/2019 12:30:00), the energy output was reported as NULL
2. For a given solar PV system on a given day, fewer than 48 timestamps exist in the dataset

All NULL values were removed from the dataset, and out of all pairings of ID-Date, only 144,730 had fewer than 48 timestamps - these were all also removed.

# Conclusion

# Collaboration


###############################################

Take everything below and separate into the above categories





This PCA process was performed on the original power generation values as well as the reconstructed values. The first principal component of the reconstructed power values explains over 90% of the variance while the first two principal components of the original power values only explain about 70% of the variance. 

<p align="center">
  <img src="images/comparing_variance_explained.png" alt="Explained Variance" width="400"/>
</p>

The reconstructions model the curves well as the mean curve aligns almost perfectly with the mean reconstruction. The standard deviations for each timestamp for the reconstructions are also generally much lower and smoother than the standard deviations of the original power values.

<p align="center">
  <img src="images/lim_3kwp_mean_curves.png" alt="EMean Curves" width="400"/>
</p>

## Exploring the Relationships Between the Top 2 Principal Components
As PCA is used to help identify outliers, it is important to determine any important properties that each principal component may represent. To aid this, the major outliers were filtered out. Then, the data is sampled such that one principal component is set to its respective mean, while the other increases. The generation curves of a small sample of points linearly spaced across the respective PC range are plotted to visualize any changes in shape. Then, the maximum and minimum reconstructed power values are plotted for all points within this range.

<p align="center">
  <img src="images/main_cluster_with_mean.png" alt="Plotting the Main Cluster" width="500"/>
</p>

### Analyzing Mean PC1 With Increasing PC2
For the following figures, the points are sampled such that PC1 is within 0.1 of its mean (-1.21)

<p align="center">
  <img src="images/mean_pc1_curves.png" alt="Visualization of Mean PC1 With Increasing PC2" width="700"/>
</p>

PC2 appears to correlate with a translation of the bulk of the curve from left to right as it increases.

<p align="center">
  <img src="images/mean_pc1_power.png" alt="Max and Min Power Generation With Mean PC1" width="500"/>
</p>

No major patterns with power generation are observed as PC2 increases, though at the edges of the distribution, there appear to be large spikes in maximum power measurements

### Analyzing Mean PC2 With Increasing PC1
For the following figures, the points are sampled such that PC2 is within 0.001 of its mean (-0.064)

<p align="center">
  <img src="images/mean_pc2_curves.png" alt="Visualization of Mean PCw With Increasing PC1" width="700"/>
</p>

PC1 appears to correlate with the height of the spike in power, though similarly to PC2, the height seems to increase at either end of the spectrum.

<p align="center">
  <img src="images/mean_pc2_power.png" alt="Max and Min Power Generation With Mean PC2" width="500"/>
</p>

As PC1 increases, the average maximum power generated decreases across this range. However, the standard deviation is fairly large across the whole range.

### Identifying Major Outliers via PCA
When plotting the data along the top two principal components, four major groupings are present:
1. A central grouping of points
2. Points where PC1 > 100
3. Points where PC1 < 100 and PC2 > 100
4. Points where PC1 < 100 and PC2 < -100

The anomaly curves for each example in these groupings are plotted to analyze their shapes.

<p align="center">
  <img src="images/pca_top_two_principal_components2.png" alt="Principal Components" width="500"/>
</p>


<p align="center">
  <img src="images/plotting_major_anomalies.png" alt="Plotting Major Anomalies" width="700"/>
</p>

There appear to be no significant differences between these anomaly groups - in fact, 8/10 are from the same ss_id, 7635, within a fairly small timeframe (2017-11-20 to 2017-12-07). The major connection between all of these outlier points is that they contain extreme maximum and/or minimum power generation values. 

### Identifying Closer Outliers via PCA
After filtering out the major outliers, a few more outliers can be identified located much more closely to the center of the distribution - these are data points where PC1 and/or PC2 are larger than 1. The curves of these minor outliers are similarly plotted below.

<p align="center">
  <img src="images/5_26_closeOutliers.png" alt="Plotting Minor Anomalies" width="500"/>
</p>
<p align="center">
  <img src="images/5_26_closeOutliers_visualized.png" alt="Visualizing Minor Anomalies" width="700"/>
</p>

Unlike the previous set of outliers, a few patterns become clear here:
* When PC1 is relatively high, the amplitude of the curve becomes extremely large
* When PC2 is relatively high, the peak of the curve is offset from the center. In fact, many of these curves appear to show peak power generation at midnight.

### Further Anomaly Detection

While there are a few obvious anomalous groupings of points that are located far from the main cluster, there are possibly many, many more located much closer. The below plot shows only data points where both PC1 and PC2 are below 1. At this level of granularity, the use of PC boundaries becomes more nuanced and arbitrary, so we will utilize other methods to further identify any anomalies.

<p align="center">
  <img src="images/5_26_closeOutliers_1.png" alt="Plotting Minor Anomalies" width="500"/>
</p>

## PCA for Anomaly Detection: Conclusion

### Effectiveness of PCA in Highlighting Anomalies
In this analysis, PCA proved to be an effective method for identifying anomalies in the dataset. By transforming the high-dimensional data into two principal components, we were able to visualize and distinguish most normal data points from anomalous ones. The scatter plots of the first two principal components (PC1 and PC2) clearly showed clusters of normal points and isolated anomalies. By setting cutoff values in the principal component space, these anomalous points can be easily filtered out of the dataset.

The specific anomalies identified had particularly high or low PC1 and PC2 values, which correlated with spikes in the measured power and/or shifts in the timing of the peak(s). This indicates that PCA can successfully capture and highlight abnormal variations in the data, particularly those associated with sudden increases in power generation.

### Benefits of Using PCA for Anomaly Detection
Most real-world datasets don't have labeled anomalies as it can be difficult to identify which points are actually anomalous. The lack of a labeled dataset heavily restricts the number of available anomaly detection methods which often consist of supervised learning approaches. Identifying anomalies via PCA not only helps in detecting outliers but also enables the creation of a labeled dataset. This labeled dataset can then be used to train and evaluate supervised learning models, enhancing our ability to predict and manage anomalies in future data.

### Drawbacks of Relying Only on PCA for Anomaly Detection
After filtering out the major anomalies identified using PCA, there is a fairly tight cluster of points that remain. In order to further identify anomalies, cutff values have to be set, often arbitrarily. So far, in order to verify that the points are truly anomalies, the power generation curves have to be individually identified. This was fine for the major anomalies as there are relatively few major anomalies. However, this becomes time-consuming and ineffective when approaching the center of the distribution

### Recommendations for Improvement
1. **Combine PCA with Other Techniques**: While PCA was effective, combining it with other anomaly detection techniques, such as clustering methods or separate supervised learning using our identified anomalies, could provide a more robust anomaly detection framework. This hybrid approach could help in capturing a wider variety of anomalies that PCA alone might miss.

### Final Thoughts
Overall, PCA has shown to be a valuable tool in detecting anomalies within the dataset, especially those related to power spikes. Moreover, the identification of anomalies via PCA helps in creating a labeled dataset, which is crucial for training supervised learning models. By incorporating additional techniques and insights, we can further enhance the model's effectiveness and accuracy, providing a robust framework for anomaly detection and management.

-------------------------------------------

## Combining PCA with Other Anomaly Detection Methods

### Statistical Methods

#### Z-Score

#### Interquartile Range (IQR)

### Isolation Forest

## Identifying Anomaly-Prone Solar PV System Configurations

vvv Below is old data, but keeping them in here for template purposes

Out of nearly 55 million data points, only 609,778 were identified as outliers whose principal components fell outside of the bounding box. To identify any patterns, the distributions between various configuration settings are compared here.

<p align="center">
  <img src="images/pca_outliers_kwp.png" alt="Comparing Distributions Between All Systems and Identified Outliers (PV System Rating)" width="500"/>
</p>

Most systems represented in this dataset have a power rating centered around 3 kW. However, outlier data points are spread out across the entire axis. There are two notable spikes in the distribution: one with systems rated at around 30 kW and another much larger one with systems rated at around 50 kW.
These could indicate that certain PV systems entirely have a high rate of anomalous measurements.

<p align="center">
  <img src="images/pca_outliers_latitude.png" alt="Comparing Distributions Between All Systems and Identified Outliers (Latitude)" width="500"/>
</p>

Most anomalous measures seem to come from latitudes between 51 and 52 with another spike at around latitude 55. While the histogram shapes are very similar, the outliers have a much tighter distribution.

<p align="center">
  <img src="images/pca_outliers_longitude.png" alt="Comparing Distributions Between All Systems and Identified Outliers (Longitude)" width="500"/>
</p>

There doesn't seem to be much indication that the longitude of each system affects the rate of anomalous measurements.

<p align="center">
  <img src="images/pca_outliers_orientation.png" alt="Comparing Distributions Between All Systems and Identified Outliers (Orientation)" width="500"/>
</p>

Overall, the overall panel orientation distribution is very similar to the anomalous panel distributions, but the outlier distribution is slightly more left-skewed.

<p align="center">
  <img src="images/pca_outliers_tilt.png" alt="Comparing Distributions Between All Systems and Identified Outliers (Tilt)" width="500"/>
</p>

Panel tilt seems to affect the rate of anomalous measurements - panels tilted closer to horizontal (angles below 30 degrees) appear to produce more anomalous measurements than those tilted more vertically.


[Reference for algorithm descriptions](https://www.datacamp.com/tutorial/introduction-to-anomaly-detection)

# Discusion

# Conclusion
