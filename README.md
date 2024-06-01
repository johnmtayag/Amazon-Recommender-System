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

<p align="center">

|generation_wh|           datetime|ss_id|
|-------------|-------------------|-----|
|          0.0|2010-11-18 00:00:00| 2405|
|          0.0|2010-11-18 00:30:00| 2405|
|          0.0|2010-11-18 01:00:00| 2405|

</p>

The metadata dataset has 24,662 rows containing supplementary information on how each solar PV system was configured. Notably, there are more PV systems identified in the metadata dataset than the number actually represented in the 30 minute dataset. There are eight columns:
1. **ss_id**: The solar PV system ID number (integer)
2. **latitude_rounded**: The latitude that the solar PV system is located (double)
3. **longitude_rounded**: The longitude that the solar PV system is located (double)
4. **llsoacd**: This variable is not defined in the source repo (string)
5. **orientation**: The direction angle from North that the solar PV system faces (double)
6. **tilt**: The tilt angle of the solar PV system (double)
7. **kwp**: The energy generation capacity of the solar PV system in kw (double)
8. **operational_at**: The date when the solar PV system was activated (date)

<p align="center">

|ss_id|latitude_rounded|longitude_rounded|  llsoacd|orientation|tilt| kwp|operational_at|
|-----|----------------|-----------------|---------|-----------|----|----|--------------|
| 2405|           53.53|            -1.63|E01007430|      180.0|35.0|3.36|    2010-11-18|
| 2406|           54.88|            -1.38|E01008780|      315.0|30.0|1.89|    2010-12-03|
| 2407|           54.88|            -1.38|E01008780|      225.0|30.0|1.89|    2010-12-03|

</p>

### Setting Up the Environment
We utilize the following non-standard Python libraries in our analysis - these need to be set up via pip install or conda install methods.
* ipyleaflet --> [[Setup documentation]](https://ipyleaflet.readthedocs.io/en/latest/installation/index.html)

# Methods

## Exploring the Metadata Dataset

### Null Values
There are 36 null values in the metadata dataset, specifically in the operational_at column:

<p align="center">

|ss_id|latitude_rounded|longitude_rounded|  llsoacd|orientation|tilt| kwp|operational_at|
|-----|----------------|-----------------|---------|-----------|----|----|--------------|
|    0|               0|                0|        0|          0|   0|   0|            36|


</p>

### Exploring the variables: Kwp (Solar PV System Power Rating)
Most of the PV systems represented in this dataset have fairly low power ratings in the range of 2-4 kW. However, there are some notable outliers with kwp values approaching 200. 

<p align="center">
<img src="images/kwp_distribution.png" alt="Distribution of kwp" width="650"/>
</p>

<p float="left" align="center">
<img src="images/hist_kwp_no_outliers.png" alt="Hist of kwp without outliers" width="400"/>
<img src="images/hist_kwp.png" alt="Hist of kwp" width="405"/>
</p>


### Exploring the variables: Panel Tilt
Most systems are oriented such that the panels are tilted at about 30 degrees or 35 degrees. The distribution then tapers off fairly uniformly at 10 degrees and 50 degrees. Interestingly, there is a tendency toward angles divisible by 5 degrees noted by prominent spikes in the distribution.

<p align="center">
<img src="images/tilt_distribution.png" alt="Tilt Distribution" width="500"/>
</p>

### Exploring the variables: Panel Orientation
Most systems are oriented such that the panels are oriented about 180 degrees from North. Considering that these systems are located in the UK which is, itself, located at a fairly high latitude, this direction may be maximizing the power generated in this region. However, there is also a notable cluster of systems oriented in the 0-50 degree range which bucks this trend.

<p align="center">
<img src="images/orientation_distribution.png" alt="Orientation Distribution" width="500"/>
</p>

### Exploring the variables: Mapping via iPyLeaflet
These systems are plotted on a map using iPyLeaflet to identify any patterns between system location and configuration parameters. The PV systems in the dataset are mainly located in England and Scotland and are particularly concentrated in populated regions. However, there are no obvious patterns between where a system is located and the other configuration parameters:

<p align="center">
<img src="images/map_system_locations.png" alt="System Locations Map" width="300"/>
</p>


**Coloring Systems by System Power Rating**:

The majority of solar PV systems throughout the country have very low power output ratings. There are few with higher power output ratings, and these are located close to major population centers.

<p align="center">
<img src="images/map_by_kwp.png" alt="Map by kWp" width="400"/>
</p>

**Coloring Systems by Panel Orientation**:

Most panels throughout the country are oriented at 180 degrees as identified earlier without any particular geographic trends.

<p align="center">
<img src="images/map_by_orientation.png" alt="Map by Orientation" width="400"/>
</p>

**Coloring Systems by Panel Tilt**:

Most panels throughout the country are tilted at about 30 degrees as identified earlier without any particular geographic trends.

<p align="center">
<img src="images/map_by_tilt.png" alt="Map by Tilt" width="400"/>
</p>


## Exploring the 30 Minute Dataset
There are 1,824,316 null values in the 30 minute dataset:

<p align="center">

|Number of Null Power Values|Number of Null Timestamps|Number of Nulls ss_ids|
|---------------------------|-------------------------|----------------------|
|                    1824316|                        0|                     0|

</p>

**Number of Timestamped Entries per Year**

Though the dataset spans the range 2010 to 2021, the number of timestamped entries from each year varies greatly. Most of the timestamps in the dataset fall within the range 2015-2021, with the range 2017-2020 being particularly highly represented. 2010 and 2011 are both represented quite poorly, especially relative to all other years in the dataset.

<p align="center">
  <img src="images/num_timestamps_yearly.png" alt="Number of Timestamped Entries per Year" width="500"/>
</p>
  
## Main Preprocessing
First, all systems outside of the interquartile range of the kwp variable (IQR: 2.28 <= kwp <= 3.42) are filtered out of the dataset. The energy generation values, measured as kW*(30 minutes) are converted into average power generation values, measured as kW. The timestamped data is then grouped by ss_id and date to get a series of sequences of power generation values representing the power generated by a single system over a single day. The dataset is then filtered further such that only groupings with 48 non-null timestamped power generation measurements are retained.

**The Formula Used to Convert the Energy Generation Values to Power Generation Values:**
```math
\text{power}_{\text{kW}} = \text{generation}_{\text{wh}} \times \left(\frac{60}{30}\right) \times \left(\frac{1}{1000}\right)
```

## Reconstructing the Daily Power Generation Curves with the Fourier Transform
The Fourier Transform is used to analyze and parameterize the daily power generation curves. This method decomposes the power generation data into simpler sinusoidal components, reducing the noise of the measurements and facilitating the identification of patterns and anomalies.

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

After projecting the data, a series of cutoff values along the top two principal components are used to identify anomalies at different scales until the PCA method becomes less effective. 

### The anomalies detected here are separated out of the original dataset and are marked as labeled anomalies for later supervised anomaly detection methods. (Change this potentially)

### Exploring the Relationships Between the Top 2 Principal Components
As PCA is used to help identify outliers, it is important to determine any important properties that each principal component may represent. To aid this, the major outliers were first filtered out. Then, the data is sampled such that one principal component is set to its respective mean, while the other increases. The generation curves of a small sample of points linearly spaced across the respective PC range are plotted to visualize any changes in shape as the value of one of the principal components increases. Then, the maximum and minimum reconstructed power values are plotted for all points within this range to understand any relationships between the principal components and the power generation curves.

The below plot shows the distribution of points after all major outliers identified during the PCA process were filtered out. At this level of granularity, the use of PC boundaries becomes more nuanced and arbitrary, so we utilize other methods to further identify any anomalies.

<p align="center">
  <img src="images/5_26_closeOutliers_1.png" alt="Plotting Minor Anomalies" width="500"/>
</p>

## Further Anomaly Detection Using Statistical Methods
After the initial PCA anomaly filtering, two different statistical approaches are applied to identify possible outlier ranges:

1. **Interquartile Range Method**
   * The first and third quartile values along both PC1 and PC2 are identified, and any points that lie outside of the following cutoff values along either PC1 and/or PC2 are marked as anomalies:
    1. Low Cutoff = Q1 - 1.5 x (Q3 - Q1)
    2. High Cutoff = Q3 + 1.5 x (Q3 - Q1)
2. **Z-Score Method**
   * Like the previous method, a series of cutoff values along both PC1 and PC2 are identified, and any points that lie outside of the boundaries along either PC1 and/or PC2 are marked as anomalies. This time, the cutoff values are defined using the mean and the sample standard deviation of the datatset:
    1. Low Cutoff = Mean - 3 x STD
    2. High Cutoff = Mean + 3 x STD

## Further Anomaly Detection Using Isolation Forests
An isolation forest is an unsupervised machine learning algorithm which utilizes binary trees to detect anomalies. The algorithm relies on the fact that data points located further away from the centers of distribution are more likely to be quickly separated from the rest of the points when randomly splitting the variable space.

**Typical Isolation Forest Process**:
* For each tree:
    * Randomly sample a subset of the data points
    * Recursively split the space along random variables at random values until some stopping criterion (ex: until every leaf node contains 3 points at maximum)
    * Give each point a score which corresponds to how close its leaf node is to the root node
    * Average this score across multiple trees
 
However, Pyspark does not have any built-in isolation forest algorithms - instead, we use a customized ensemble of single-node binary trees that split the space with a randomized boundary line, and we record the average score for each point across all iterations.

**Custom Isolation Forest Process:**
* First get the minimum and maximum values of the two variables (here, PC1 and PC2)
* For each forest:
    * For each tree:
        * Pick two random points within the space and get the parameters of the line that connects them in standard form (Ax +  By + C = 0)
        * Split the points into two groups according to which side of the line they are on
        * Give each point a score which corresponds to the fraction of all points that lie on the same side of the line
        * To minimize data storage requirements, track the scores in a running total, then divide by the number of splits for the average scores for each tree.
* The resulting score for each point is averaged across trees, and across forests.

To help visualize this process, the following figure shows 4 randomly generated boundary lines along with the resulting scores assigned to each group. Note how for each boundary line, the group where the mean PC value is included is given a very high score while the group on the other side is given a score close to zero. In the case that the boundary line splits the data in half, both sides get a score of about 0.5.

<p align="center">
  <img src="images/iforest_example_boundary_lines.png" alt="Example Isolation Forest Boundary Lines" width="800"/>
</p>

This method relies on both randomness and the aggregated results of weak learner predictions, so there can be a lot of variability in results. Thus, averaging the results across multiple trees and even multiple forests is ideal. To maximize the likelihood that an outliers identified with this method are truly anomalous, we will test different combinations of number of trees and number of splits per forest, and we will use the results of the combination with this lowest variance in results.

With the chosen set of scores, we plot the data once again along the principal component axes, but this time with each point colored according to its outlier score. We also plot the change in the average reconstructed power curve as the outlier score decreases.

## Testing Supervised Anomaly Detection Methods Using Labeled Data

## Identifying Correlations Between System Configurations and Outlier Frequency
To see if there is any correlation between the rate of occurence of the identified anomalies with any of the PV system configuration parameters, the data is partitioned into two main groups - outliers and all points. Then, the distributions are visualized with histograms. Only the following outlier subgroups are included for these visualizations:

* Major and close anomalies identified during PCA
* Interquartile Method (Low PC1 / Low PC2 only)
* Z-Score Method (Low PC1 / Low PC2 only)
* Isolation Forest (Score < 0.746)

Similarly, only the following parameters are included in this analysis:

* System Power Rating
* Latitude
* Longitude
* Panel Orientation
* Panel Tilt

# Results

## Comparing the Effectiveness of PCA Performed on the Original Power Values Versus the Reconstructed Values
To understand how reconstructing the power generation curves via the Fourier Transform impacts our analysis, we perform PCA on both and plot the amount of variance explained by both sets of eigenvectors. The first principal component of the reconstructed power values explains over 90% of the variance while the first two principal components of the original power values only explain about 70% of the variance.

<center>

<p float="left">
<img src="images/comparing_variance_explained.png" alt="Explained Variance" width="420"/>
<img src="images/comparing_variance_explained_values.png" alt="Hist of kwp" width="300"/>
</p>

</center>

The resulting reconstructions model the overall average curve quite well, and they smooth out the spikes and dips in the standard deviations at different timestamps throughout the day.

<p align="center">
  <img src="images/lim_3kwp_mean_curves.png" alt="EMean Curves" width="600"/>
</p>

## Exploring the Relationships Between the Top 2 Principal Components

### Analyzing Mean PC1 With Increasing PC2
For the following figures, all points where PC1 is within 0.1 of its mean (-1.21) are extracted. Six points are then randomly sampled to visualize how the curves change as PC2 increases along the range. The next figure shows the maximum and minimum power generation values for each point within the range ordered by PC2 value.

When PC2 is relatively low or relatively high, the resulting curves are highly irregular and deviate from a single central peak. However, no particular trends are present when analyzing the maximum and minimum power generation values within this range.

<p align="center">
  <img src="images/mean_pc1_curves.png" alt="Visualization of Mean PC1 With Increasing PC2" width="700"/>
</p>

<p align="center">
  <img src="images/mean_pc1_power.png" alt="Max and Min Power Generation With Mean PC1" width="500"/>
</p>

### Analyzing Mean PC2 With Increasing PC1
For the following figures, all points where PC2 is within 0.001 of its mean (-0.064) are extracted. Again, six points are then randomly sampled to visualize how the curves change as PC1 increases along the range. The next figure shows the maximum and minimum power generation values for each point within the range ordered by PC1 value.

When PC1 is relatively low or relatively high, the resulting curve has a higher peak value. Within this range, the maximum power value decreases overall as PC1 grows, though there is significant variance between the values.

<p align="center">
  <img src="images/mean_pc2_curves.png" alt="Visualization of Mean PC2 With Increasing PC1" width="700"/>
</p>

<p align="center">
  <img src="images/mean_pc2_power.png" alt="Max and Min Power Generation With Mean PC2" width="500"/>
</p>

### Identifying Major Outliers via PCA
When plotting the data along the top two principal components, four major groupings are present:
1. A central grouping of points
2. Points where PC1 > 100
3. Points where PC1 < 100 and PC2 > 100
4. Points where PC1 < 100 and PC2 < -100

In total, 10 major anomalies were identified. The anomaly curves for each example in these groupings are plotted to analyze their shapes.

<p align="center">
  <img src="images/pca_top_two_principal_components2.png" alt="Principal Components" width="500"/>
</p>

<p align="center">
  <img src="images/plotting_major_anomalies.png" alt="Plotting Major Anomalies" width="700"/>
</p>

### Identifying Closer Outliers via PCA
After filtering out the major outliers, a total of 7 more outliers can be identified located much more closely to the center of the distribution - these are data points where PC1 and/or PC2 are larger than 1. The curves of these minor outliers are similarly plotted below.

<p align="center">
  <img src="images/5_26_closeOutliers.png" alt="Plotting Minor Anomalies" width="500"/>
</p>
<p align="center">
  <img src="images/5_26_closeOutliers_visualized.png" alt="Visualizing Minor Anomalies" width="700"/>
</p>

## Further Anomaly Detection Using Statistical Methods
To aid in the further identification of outliers, the previously identified points were filtered out to minimize the region of interest:

<p align="center">
  <img src="images/5_26_closeOutliers_1.png" alt="Visualizing Minor Anomalies" width="500"/>
</p>

### Interquartile Range Method
As previously stated, the interquartile range method identifies outliers along a single axis using Q1 - 1.5\*IQR and Q3 + 1.5\*IQR as the cutoff values to define outlier points.

**PC1 Outlier Statistics**:

<p align="center">

|High Cutoff|Low Cutoff|Number of High Outliers| Number of Low Outliers|
|-----------|----------|-----------------------|-----------------------|
|     1.5374|   -3.8405|                      0|                  16108|

</p>

**PC2 Outlier Statistics**:

<p align="center">

|High Cutoff|Low Cutoff|Number of High Outliers| Number of Low Outliers|
|-----------|----------|-----------------------|-----------------------|
|     1.5374|   -3.8405|                  93105|                   2463|

</p>

**Visualizing the Outliers**

No high outliers for PC1 were identified, so the figure only shows the low outliers.

<p align="center">
  <img src="images/iqr_outliers_visualized.png" alt="Visualizing Minor Anomalies" width="600"/>
</p>

**Visualizing the Average Outlier Curves**

The shape of the average curve for both low PC1 outliers and low PC2 outliers are very similar. However, the average curve for the low PC1 outliers has a taller peak at about 3 kW while the low PC2 outliers has a shorter peak at about 2.5 kW. The curve for the low PC2 outlier group is also slightly skinnier.

Unlike the low outlier groups, the shape of the average curve for the high PC2 outliers has no significant difference from the shape of the average curve for the normal points.

<p align="center">
  <img src="images/iqr_outliers_curves.png" alt="Visualizing Minor Anomalies" width="700"/>
</p>

### Z-Score Method
As previously stated, the Z-score method identifies outliers along a single axis using the mean +- 3 standard deviations as the cutoff values to define outlier points.

**PC1 Outlier Statistics**:

<p align="center">

|High Cutoff|Low Cutoff|Number of High Outliers| Number of Low Outliers|
|-----------|----------|-----------------------|-----------------------|
|     1.3410|   -3.7621|                      0|                  26770|

</p>

**PC2 Outlier Statistics**:

<p align="center">

|High Cutoff|Low Cutoff|Number of High Outliers| Number of Low Outliers|
|-----------|----------|-----------------------|-----------------------|
|     0.0847|   -0.2128|                  84879|                   3186|

</p>

**Visualizing the Outliers**

There is almost no difference between this visualization and the previous visualization obtained using the interquartile method.

<p align="center">
  <img src="images/std_outliers_visualized.png" alt="Visualizing Minor Anomalies" width="600"/>
</p>

**Visualizing the Average Outlier Curves**

Again, there is almost no difference between this visualization and the previous visualization obtained using the interquartile method.

<p align="center">
  <img src="images/std_outliers_curves.png" alt="Visualizing Minor Anomalies" width="700"/>
</p>

## Further Anomaly Detection Using Isolation Forests
The isolation forest algorithm incorporates a lot of randomness, so to obtain stable results, a high number of trees and splits is ideal. However, the more trees and splits utilized, the more costly the algorithm becomes (in terms of both time-elapsed and memory requirements). Thus, a series of tests were run, each using a different combination of the main two hyperparameters: The number of trees and the number of splits per tree. Each combination was run 5 times with the resulting score for each point averaged across all 5 runs.

<p align="center">

|Number of Trees|Number of Splits|Minimum Score| Maximum Score|Mean Score|Standard Deviation|
|---------------|----------------|-------------|--------------|----------|------------------|
|              1|              10|       0.4112|        0.9017|    0.8699|            0.0407|
|              1|              30|       0.3994|        0.8858|    0.8470|            0.0475|
|              3|              10|       0.4117|        0.9212|    0.8910|            0.0308|
|              3|              30|       0.4247|        0.9187|    0.8908|            0.0344|
|              5|              10|       0.4243|        0.9181|    0.8895|            0.0377|
|              5|              30|       0.4295|        0.9165|    0.8880|            0.0345|
|              7|              10|       0.4379|        0.9235|    0.8945|            0.0298|
|              7|              30|       0.4409|        0.9247|    0.8962|            0.0358|

</p>

Overall, the resulting scores were all quite similar, but the combination of 7 trees with 10 splits had the lowest overall standard deviation in scores, so these hyperparameters were chosen. For each point in this resulting model, the individual mean score and standard deviation across runs was taken, and the shapes of the distributions were visualized.

**Distribution of Mean Scores and Individual Standard Deviations**

Most points had very high scores with the first quartile being located at around 0.875. The lower whisker for outlier values was located at about 0.825, so all scores below that cutoff were classified as outliers for this analysis. The individual standard deviations were all also extremely small, with most being well under 0.0025. The largest standard deviations were around 0.03, so overall, the results for this run were extremely stable.

<p align="center">
  <img src="images/7t10s_distribution.png" alt="Distributions of scores" width="800"/>
</p>

**Visualizing the Resulting Outlier Scores**

The highest scores were located near the mean (demarcated with a red x). The further away from the mean, the lower the score, with the lowest scores being clustered mainly to the far left of the distribution. The figure on the right shows the same data except with the normal points greyed out to highlight the outliers.

<p align="center">
  <img src="images/visualizing_score_distribution_combined.png" alt="Distributions of scores" width="800"/>
</p>

**Visualizing the Average Curve as the Outlier Score Changes**

The average curve for the normal points has a bell-shape that peaked at about 0.8. As the outlier score decreases, the height of the curve's peak rises. At about 0.60, the curve starts to split, and the shape becomes more irregular. At the lowest outlier score values, the curve displays a prominent initial peak at about timestamp 20, followed by a dip, then a smaller peak at about timestamp 30. 

<p align="center">
  <img src="images/iforest_average_curves.png" alt="Change in Average Curve as Outlier Score changes" width="500"/>
</p>

## Supervised Anomaly Detection with Extracted Labeled Anomalies

## Identifying Correlations Between System Configurations and Outlier Frequency
The following plots show the shapes of the distribution in the metadata variables according to anomaly grouping.

**Distributions of kwp**

Compared to the distribution for all points, each of the anomaly groups had higher anomaly counts associated with systems in the higher kwp range. Though there appears to be spikes in the distribution for the PCA grouping, keep in mind that the scales on the y-axis are  significantly different between plots.

<p align="center">
  <img src="images/outliers_kwp.png" alt="Distribution by Anomaly Grouping: kwp" width="650"/>
</p>

**Distributions of latitude**

There is a spike in outlier counts around 56 degrees in the PCA outlier grouping - this is from the single ss_id with 8 major anomalous curves identified previously. The other distributions have fairly similar shapes to the distribution for all systems, though the isolation forest group has very few instances with higher latitude

<p align="center">
  <img src="images/outliers_latitude.png" alt="Distribution by Anomaly Grouping: latitude" width="650"/>
</p>

**Distributions of longitude**

Similar to the latitude plots, there is another spike in counts at about -5 degrees in the PCA outlier grouping from the single ss_id with 8 anomalous curves. The others, again, have similar shapes to the distribution for all systems, though the isolation forest group has a significant spike at about -1 degrees.

<p align="center">
  <img src="images/outliers_longitude.png" alt="Distribution by Anomaly Grouping: longitude" width="650"/>
</p>

**Distributions of panel orientation**

The isolation forest group has very few instances where the panel orientation is greater than 200 degrees, especially compared to how many systems of these systems exist.

<p align="center">
  <img src="images/outliers_orientation.png" alt="Distribution by Anomaly Grouping: orientation" width="650"/>
</p>

**Distributions of panel tilt**

The isolation forest group again has another difference in the shape from the distribution of all systems - there are significant spikes in outlier counts where the panel tilt is at about 15 degrees and 20 degrees.

<p align="center">
  <img src="images/outliers_tilt.png" alt="Distribution by Anomaly Grouping: tilt" width="650"/>
</p>

# Discussion

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

### Preprocessing: Collecting Timestamp Groupings and Removing Missing Data Points
Each entry in the 30 minute dataset consists of the system ID, the timestamp, and the measured energy output. In order to analyze the power output throughout an entire day, the timestamps must first be collected into groups by both system ID and date. As the measurements are taken every 30 minutes, there should be, ideally, 48 timestamps per system ID for every date. Due to the coarse-grained nature of these measurements, any missing data points can greatly affect the shapes of the fitted models, leading to possible false flags. Thus, in order to parameterize the power generation curves as accurately as possible, we need to minimize the number of missing data points.

There are two main categories of missing data points:
1. For a given solar PV system at a given timestamp (ex: 04/24/2019 12:30:00), the energy output was reported as NULL
2. For a given solar PV system on a given day, fewer than 48 timestamps exist in the dataset

All NULL values were removed from the dataset, and out of all pairings of ID-Date, only 144,730 had fewer than 48 timestamps - these were all also removed.

## Analyzing the Principal Components of the Reconstructed Power Curves
As PC1 explains over 90% of the total variance, its value has the greatest impact on the overall shape of the curve. When PC1 is extremely high or extremely low, the resulting power generation curve has an anomalous spike which dominates the curve. After filtering out the major outliers and zooming into the normal range, an increase in PC1 is associated with a decrease in the maximum power value up until PC1 = 0. Afterward, the maximum power value appears to spike once again.

On the other hand, PC2 does not have a clear correlation with any particular property of the power generation curves. However, comparing the individual sampled curves *does* seem to imply that when PC2 is extremely high or extremely low, the resulting power generation curve is wider. Considering that these curves represent the power generated from solar PV systems, the most likely curve shape would be somewhere between a bell-curve and a square-curve, depending on environmental factors and the system configurations. However, some curves with high or low PC2 values show power being generated at night, which is highly unlikely.

## Analyzing the Anomalies Identified During the PCA Process

### Major Outliers
Though there are 3 clearly separated groups of anomalies, there actually is no significant difference between the curves from each group. In fact, 8/10 are from the same ss_id, 7635, and all have measurements taken within a fairly small timeframe (2017-11-20 to 2017-12-07). The most significant connection between all of these outlier points is that they contain extreme spikes in the maximum and/or minimum power generation values. The timings of the spike have no clear correlation, though interestingly, two of the curves have dual spikes at different times in the day.

<p align="center">
  <img src="images/plotting_major_anomalies.png" alt="Plotting Major Anomalies" width="700"/>
</p>

### Close Outliers
After filtering out the major anomalies, the next set of anomalies can be easily separated from the central cluster. Unlike the previous set of outliers though, a few patterns become clear when analyzing these curves:

* When PC1 is relatively high, the amplitude of the curve becomes extremely large. This likely also occurs when PC1 is relatively low.
* When PC2 is relatively high, the peak of the curve is offset from the center. In fact, many of these curves appear to show peak power generation at midnight. This likely also occurs when PC2 is relatively low

These properties correlate with the choice of basis vectors. As stated previously, the basis vectors consist of a single bias curve to offset the reconstructed power curves vertically as well as a series of sinusoidal curves to recreate the shape of the original curves. PC1 appears to correlate with the coefficient for the bias vector while PC2 (and likely the next PCs) appars to correlate with the coefficients for the sinusoidal basis vectors.

<p align="center">
  <img src="images/5_26_closeOutliers_visualized.png" alt="Visualizing Minor Anomalies" width="700"/>
</p>

## Analyzing the Effectiveness of Statistical Methods for Outlier Detection
Both the interquartile method and the z-score method resulted in very similar cutoff values and very similar average curve shapes for each grouping of outliers. Both methods resulted in no high PC1 outliers, and both methods had high PC2 outliers that closely resembled the normal curve. Also, both methods show the low PC2 outliers as having skinnier distributions compared to the low PC1 outlier group.

While they were very similar, though, the interquartile method extracted a significantly lower number of low outliers, especially on the PC1 axis, and a slightly higher number of high outliers on the PC2 axis. Neither method worked particularly well at identifying truly anomalous curves, and this is likely attributed to the distribution not being normal. The distribution is highly left skewed, particularly along the PC1 axis. Both methods also set a single boundary line in either direction which limits any further analysis as only a single average curve can be generated to analyze each anomaly grouping.

## Analyzing the Effectiveness of the Isolation Forest for Outlier Detection
The isolation forest was quite effective in separating out outliers from the central cluster in all directions. Most of the points with the lowest scores are located far to the left of the distribution, and the score increases when approaching the mean. These scores also help to further organize the anomalies into subcategories correlating with distance from the mean.

Plotting the average reconstruction curve as the score decreases is quite interesting - as the outlier score decreases, the height of the curve's peak increases and slowly splits into one tall curve and one slightly smaller peak to the right. Also, the variance seems to increase as the overall curve's shape becomes more jagged. The change in the mean curve reveals that the further away a point's PC1 and PC2 values are from the mean, the more irregular the power generation curve becomes.

Without extra information, it is hard to explain why the average curve seems to split in two so cleanly. One possible explanation could be the prevalence of cloudy/rainy weather in the UK causing anomalous dips and/or spikes in power generation.

<center>
<p float="left">
  <img src="images/visualizing_score_distribution_outliers_highlighted.png" alt="Distributions of scores" width="380"/>
  <img src="images/iforest_average_curves.png" alt="Change in Average Curve as Outlier Score changes" width="400"/>
</p>
</center>

## Analyzing the Effectivness of Supervised Outlier Detection Methods

## Analyzing the Distributions of System Configurations Within Anomaly Groupings
Overall, the only variable that appears to correlate strongly with the occurence of anomalies is kwp. This makes sense as many of the average "anomalous" curves were simply normal curves with higher peaks. The average outlier curves identified with both statistical methods had fairly normal shapes that had peaks at 3 kW or lower. Similarly, the average curves identified using the isolation forest followed a similar pattern until the score dropped to 0.65 or below. However, given that the kwp cutoff for solar PV systems to be included in this analysis was 3.42, these "anomalies" may actual be legitimate curves.

This result seems to suggest that, in order to ensure that all identified curves are truly anomalous, the dataset may need to be even further restricted to a smaller smaller subset of solar PV systems.

<p align="center">
  <img src="images/outliers_kwp.png" alt="Distribution by Anomaly Grouping: kwp" width="650"/>
</p>

Though the outlier frequencies for the IQR and STD methods had fairly similar distributions to the number of systems represented in the dataset, the isolation forest group did have a few notable differences in the orientation and tilt variables. Significantly fewer systems with an orientation above 200 degrees had outlier scores below 0.6. There were also a couple of spikes in outlier frequency where the panel tilt was at about 15 degrees and 20 degrees. The data suggests that there may be some kind of correlation between the frequency of anomalous power generation curves and panel tilt/orientation, but further analysis is required to confirm this suspicion.

<p align="center">
  <img src="images/outliers_orientation_tilt.png" alt="Distribution by Anomaly Grouping: iforest only" width="650"/>
</p>

# Conclusion

## Discuss more of the above methods here, incorporate more than just PCA

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

# Collaboration

