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
\[
\mu[i,j]=\frac{1}{N}\sum_{k=-m}^{m}\sum_{l=-m}^{m}P[i+k,j+l]
\]

This is the same averaging calculation we just discussed.

It estimates the local background or typical intensity.
B. Local variance
Variance tells us how much the pixel values differ from their local mean.
\[
\sigma^2[i,j]=
\frac{1}{N}\sum_{k=-m}^{m}\sum_{l=-m}^{m}
P^2[i+k,j+l]-\mu^2[i,j]
\]

Intuition:
- Low variance: neighbouring pixels have similar intensities.
- High variance: neighbouring pixels have substantial intensity differences.
High variance could indicate noise, texture, an edge, or actual sample structure. Variance alone cannot tell us which one it is.
C. Estimate the noise variance
The paper then estimates a global noise variance:
\[
v^2=\frac{1}{L\times W}
\sum_{i=0}^{L-1}\sum_{j=0}^{W-1}\sigma^2[i,j]
\]

Here:
- \(L\) = image length (number of rows).
- \(W\) = image width (number of columns).
- \(L\times W\) = total number of pixels.
- \(\sigma^2[i,j]\) = local variance at each pixel.
The paper assumes this white-noise variance is constant across the image and estimates it by averaging the local variances. 2020- Validation of multispectral imaging for the detection of selected adulterants in turmeric samples (1).pdf

6. The actual Wiener filtering equation
The paper gives:
\[
\boxed{
P_w[i,j]=\mu[i,j]+
\frac{\sigma^2[i,j]-v^2}{\sigma^2[i,j]}
\left(P[i,j]-\mu[i,j]\right)
}
\]

Let's break this into three parts.

Part 1: Local mean
\[
\mu[i,j]
\]

This is the typical intensity of the neighbourhood.

Part 2: Deviation from the local mean
\[
P[i,j]-\mu[i,j]
\]

This tells us how far the original pixel is from its neighbourhood's average.

Part 3: Adaptive weight
\[
\frac{\sigma^2[i,j]-v^2}{\sigma^2[i,j]}
\]

This determines how much of the original pixel's deviation we preserve.

Let's understand that weight.
Case A: Low local variance
Suppose:
\[
\sigma^2[i,j]=10,\qquad v^2=8
\]

Then:
\[
\text{Weight}=\frac{10-8}{10}=0.2
\]

Only 20% of the original deviation from the local mean is retained.

The output moves closer to the local mean.

Interpretation: When local variation is close to the estimated noise level, the filter suppresses much of that variation.
Case B: High local variance
Suppose:
\[
\sigma^2[i,j]=100,\qquad v^2=8
\]

Then:
\[
\text{Weight}=\frac{100-8}{100}=0.92
\]

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
\[
P_w\approx\mu
\]

We move the pixel towards its neighbourhood's mean because its deviation may largely be noise.

Situation B: Pixel variation contains useful structure

Suppose local variance is much greater than the estimated noise variance.

The weight becomes large, so:
\[
P_w\approx P
\]

We preserve more of the original pixel's deviation, helping retain texture, edges and other local variations.
Also, high local variance does not guarantee that the variation is genuine texture. It could also be noise. Wiener filtering makes a statistical estimate, not a perfect distinction.
Adaptive Wiener filtering smooths regions where local variation is close to the estimated noise level, while preserving more variation where the local variance is substantially higher.


   
