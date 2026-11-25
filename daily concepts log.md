24th nov
## At the stage of image preprocessing where the paper does:
1. Dark current subtraction
2. Filtering
   **Median filtering**
   Median filtering is particularly effective against impulse noise, where individual pixels have unusually high or low values.
   For example:
  - A black pixel appearing unexpectedly in a bright region.
  - A white pixel appearing unexpectedly in a dark region.
  Median filtering can remove these isolated disturbances without averaging them into neighbouring pixels.

  **Adaptive weiner filtering**
  Region A: relatively uniform
    50 51 49
    50 52 49
    51 50 50
  Region B: contains an edge or texture
  20 22 18
  21 120 125
  19 118 122
Should we smooth both regions equally?

Probably not.
- In Region A, small variations are likely to be noise, so stronger smoothing is useful.
- In Region B, some variations may represent real image structure, so excessive smoothing could destroy useful information.
this is formula

$$
P_w[i,j] = \mu[i,j] +
\frac{\sigma^2[i,j]-v^2}{\sigma^2[i,j]}
\left(P[i,j]-\mu[i,j]\right)
$$

A. Local mean
The local mean tells us the average intensity around the current pixel.

$$
\mu[i,j]=\frac{1}{N}\sum_{k=-m}^{m}\sum_{l=-m}^{m}P[i+k,j+l]
$$

This is the same averaging calculation we just discussed.

It estimates the local background or typical intensity.
B. Local variance
Variance tells us how much the pixel values differ from their local mean.

$$
\sigma^2[i,j]=
\frac{1}{N}\sum_{k=-m}^{m}\sum_{l=-m}^{m}
P^2[i+k,j+l]-\mu^2[i,j]
$$

Intuition:
- Low variance: neighbouring pixels have similar intensities.
- High variance: neighbouring pixels have substantial intensity differences.
High variance could indicate noise, texture, an edge, or actual sample structure. Variance alone cannot tell us which one it is.
C. Estimate the noise variance
The paper then estimates a global noise variance:

$$
v^2=\frac{1}{L\times W}
\sum_{i=0}^{L-1}\sum_{j=0}^{W-1}\sigma^2[i,j]
$$

Here:
- \(L\) = image length (number of rows).
- \(W\) = image width (number of columns).
- \(L\times W\) = total number of pixels.
- \(\sigma^2[i,j]\) = local variance at each pixel.
The paper assumes this white-noise variance is constant across the image and estimates it by averaging the local variances. 2020- Validation of multispectral imaging for the detection of selected adulterants in turmeric samples (1).pdf

6. The actual Wiener filtering equation
The paper gives:

$$
\boxed{
P_w[i,j]=\mu[i,j]+
\frac{\sigma^2[i,j]-v^2}{\sigma^2[i,j]}
\left(P[i,j]-\mu[i,j]\right)
}
$$

Let's break this into three parts.

Part 1: Local mean

$$
\mu[i,j]
$$

This is the typical intensity of the neighbourhood.

Part 2: Deviation from the local mean

$$
P[i,j]-\mu[i,j]
$$

This tells us how far the original pixel is from its neighbourhood's average.

Part 3: Adaptive weight

$$
\frac{\sigma^2[i,j]-v^2}{\sigma^2[i,j]}
$$

This determines how much of the original pixel's deviation we preserve.

Let's understand that weight.
Case A: Low local variance
Suppose:

$$
\sigma^2[i,j]=10,\qquad v^2=8
$$

Then:

$$
\text{Weight}=\frac{10-8}{10}=0.2
$$

Only 20% of the original deviation from the local mean is retained.

The output moves closer to the local mean.

Interpretation: When local variation is close to the estimated noise level, the filter suppresses much of that variation.
Case B: High local variance
Suppose:

$$
\sigma^2[i,j]=100,\qquad v^2=8
$$

Then:

$$
\text{Weight}=\frac{100-8}{100}=0.92
$$

Now 92% of the deviation is retained.

The filter preserves most of the original pixel's deviation.

Interpretation: When local variation is much greater than the estimated noise level, the filter preserves more image detail.

This is why it is called adaptive: the amount of smoothing changes according to the local image statistics.
The key difference
Median
Uses the middle value of neighbouring pixels to suppress outliers.


Wiener
Uses local mean, local variance and estimated noise variance to decide how much of a pixel's variation to preserve.


Technical note: The printed Wiener equation needs care when local variance is zero or below the estimated noise variance. A practical implementation normally handles those cases to avoid division by zero or negative weights. We'll follow the paper's exact formulation and document any implementation safeguards separately.
7. Why did the paper choose a 3 × 3 window?
This is particularly important for your research.

The authors explain that filtering is essentially a low-pass smoothing operation.

Think of image information as having two broad types of variation:
- Low-frequency information: gradual changes across larger regions.
- High-frequency information: rapid changes between neighbouring pixels.
Random noise can create high-frequency fluctuations.

But adulterant-related information can also create high-frequency fluctuations!

Therefore, if we use a very large window, we might remove not only noise but also useful information about adulteration.

The authors experimented with larger window orders and observed that some adulterant-related information was removed.

They selected \(m=1\), giving a 3 × 3 window, as a compromise between noise reduction and preservation of useful information.
 
2. Why do we need the weight?
   Suppose the local variance is close to the estimated noise variance.

The weight becomes small, so:

$$
P_w\approx\mu
$$

We move the pixel towards its neighbourhood's mean because its deviation may largely be noise.

Situation B: Pixel variation contains useful structure

Suppose local variance is much greater than the estimated noise variance.

The weight becomes large, so:

$$
P_w\approx P
$$

We preserve more of the original pixel's deviation, helping retain texture, edges and other local 
variations.
Also, high local variance does not guarantee that the variation is genuine texture. It could also be noise. Wiener filtering makes a statistical estimate, not a perfect distinction.
Adaptive Wiener filtering smooths regions where local variation is close to the estimated noise level, while preserving more variation where the local variance is substantially higher.


### after weiner filtering they said to examine the spectral signature of the images
- firstly they selected 100 pixel in each replica and they have taken 30 replicas in each percent level adulteation
- now selecting 100 pixels in an image randomly in a replica then each pixel can be represented by 9 pixels
- so columns are 9 and 100 pixels with each 30 replica would give us total 3000 pixels
- so data matrix dim are 3000*9 for one percent level
- this been done for each percent level from 0 to 40
__spectral curve__
spectral curve is the graph of pixel intensity val at each wavelength.
Now why did they select 100 pixels:
Many pixels may belong to the background rather than turmeric.
Nearby pixels can be highly similar because they belong to the same physical region.
Processing every pixel creates a much larger dataset and computational burden.
The researchers wanted a manageable collection of spectral observations representing the sample.

### why did they prepare 30 replicates per %:
For example, different preparations can have variation due to:
- Mixing and distribution of adulterant.
- Differences in powder packing.
- Sample surface texture.
- Small differences during image acquisition.


### now why did the mean spectral signatures fail
For one concentration, they have 3,000 pixel observations. Each observation contains nine intensities.

They calculate the average intensity at each wavelength:

$$
\bar X_p =
[\bar I_{405},\bar I_{430},\ldots,\bar I_{950}]
$$

now rom here two samples have diff adulateration percentages but their average intensities are very similar.

25th nov
now paper said that -- When the data matrixes for all the adulteration levels (
Xp (m,n), for
p = 0,5,10 ... 40) were created, mean spectral intensity values along
the nine dimensions (9 spectral bands) were calculated and denoted as
Xp as shown in Fig. 5. Thus, the dimensions of the
Xp is
1 * 9 
Next, thevariation of the average spectral intensity values with the wavelength for different adulteration levels were shown in Fig. 6 however, they are not noticeably distinguished. Therefore, the identification of adulterated turmeric by the mean spectral signatures is not a viable solution, as it can lead to incorrect results. In order to reduce the data redundancy and to improve the discrimination between classes (adulterated levels),
PCA was performed on the nine-dimensional dataset. In addition, it would allow us to reduce the correlation between each spectral band and pave the way to the construction of the functional relationship between a statistical metric and the adulteration level in a more efficient manner.

now i asked some questions 
first thing i want to ask is that if they suppose got spectral sig distinguised then how would theyuse this info to classify cuase i think thrie main goal is to classify the adulateration levels right if i am not wrong so they would do this
nxt thing they said to use pca to get uncorrelated spectral bands how i get to knwo that features are diff bands and they can be correlated
next thing they said Therefore,
the PCA step was carried out in its traditional scenes to do a simplified
dimension reduction which will enable us to construct the multivariate
Gaussian distribution function in a more efficient manner.

so why they want to construct the multivariate gaussian distribution functuon at the fist place




