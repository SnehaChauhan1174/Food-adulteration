# Adaptive Wiener Filtering – Implementation

## 1. Objective

Implement the adaptive Wiener filtering technique described in the 2020 turmeric MSI paper using a 3×3 neighbourhood window.

**Input:** Dark-current-corrected spectral image (`corrected_images[3]`).

- Image dimensions: 1024 × 1280
- Input data type: `float32`

```python
one_img = corrected_images[3]
```

## 2. Local Mean Calculation

**Equation:**

$$
\mu(i,j)=\frac{1}{N}
\sum_{k=-1}^{1}\sum_{l=-1}^{1}
P(i+k,j+l)
$$

The local mean is the average intensity of pixels within the 3×3 neighbourhood.

**Implementation:**

```python
mean_img = np.zeros((1024, 1280))

for i in range(1024):
    for j in range(1280):
        mean = 0
        count = 0

        for k in range(i-1, i+2):
            for l in range(j-1, j+2):
                if 0 <= k < 1024 and 0 <= l < 1280:
                    mean += one_img[k][l]
                    count += 1

        mean_img[i][j] = mean / count
```

**Boundary handling:** Only valid neighbouring pixels are considered. Thus, the neighbourhood contains 9 pixels for interior locations, 6 for edges and 4 for corners.

**Results:**

| Measurement | Original | Mean filtered |
|---|---:|---:|
| Mean | 2.671634 | 2.671643 |
| Standard deviation | 6.081753 | 5.979446 |

**Observation:** Mean intensity remains almost unchanged, while standard deviation decreases, indicating reduced overall intensity variation.

## 3. Local Variance Calculation

**Equation:**

$$
\sigma^2(i,j)=
\frac{1}{N}\sum_{k=-1}^{1}\sum_{l=-1}^{1}
P^2(i+k,j+l)-\mu^2(i,j)
$$

Local variance measures the intensity variation within each neighbourhood.

**Implementation:**

```python
var_img = np.zeros((1024, 1280))

for i in range(1024):
    for j in range(1280):
        var = 0
        count = 0

        for k in range(i-1, i+2):
            for l in range(j-1, j+2):
                if 0 <= k < 1024 and 0 <= l < 1280:
                    var += one_img[k][l]**2
                    count += 1

        var_img[i][j] = var / count - mean_img[i][j]**2
```

**Numerical results:**

| Measurement | Value |
|---|---:|
| Minimum variance | 0.00 |
| Maximum variance | 85.73 |
| Mean variance | 1.234 |
| Centre pixel variance | 2.889 |
| Corner pixel variance | 0.00 |

**Visual observations:**
- Higher variance occurs around the sample boundary and in certain textured regions.
- Most background regions exhibit low variance.
- Higher variance may represent noise, texture or edges; it is not necessarily noise alone.

## 4. Global Noise Variance Estimation

The paper estimates a constant noise variance by averaging the local variance values across the image.

**Equation:**

$$
v^2=\frac{1}{LW}
\sum_{i=0}^{L-1}\sum_{j=0}^{W-1}
\sigma^2(i,j)
$$

**Implementation:**

```python
noise_var = np.mean(var_img)
```

**Result:**

$$
v^2 = 1.2337357
$$

This value is used as the global noise variance estimate in the Wiener filter.

## 5. Adaptive Wiener Filtering

**Equation from the paper:**

$$
P_w(i,j)=\mu(i,j)+
\frac{\sigma^2(i,j)-v^2}{\sigma^2(i,j)}
\left(P(i,j)-\mu(i,j)\right)
$$

Where:

- $P(i,j)$: Original pixel intensity
- $\mu(i,j)$: Local mean
- $\sigma^2(i,j)$: Local variance
- $v^2$: Estimated global noise variance
- $P_w(i,j)$: Filtered pixel intensity

The filter adjusts the contribution of the original pixel deviation according to the local variance relative to the estimated noise variance.

**Implementation:**

```python
weiner_img = np.zeros_like(one_img, dtype=np.float64)

for i in range(1024):
    for j in range(1280):

        if var_img[i][j] > 0:
            weiner_img[i][j] = mean_img[i][j] + (
                (var_img[i][j] - noise_var) / var_img[i][j]
            ) * (one_img[i][j] - mean_img[i][j])

        else:
            weiner_img[i][j] = mean_img[i][j]
```

Zero local variance is handled by assigning the local mean to avoid division by zero.

## 6. Results and Comparison

### Numerical comparison

| Measurement | Original corrected | Mean filter | Wiener filter |
|---|---:|---:|---:|
| Mean | 2.671634 | 2.671643 | 2.673683 |
| Standard deviation | 6.081753 | 5.979446 | 6.076581 |

### Average absolute pixel differences

| Comparison | Value |
|---|---:|
| Original vs Mean | 0.441834 |
| Original vs Wiener | 0.618549 |
| Mean vs Wiener | 0.568343 |

**Observations:**

1. Mean filtering produces a greater reduction in standard deviation than Wiener filtering.
2. Wiener filtering produces a larger average absolute pixel change from the original image.
3. Both filters preserve the overall circular sample structure.
4. Wiener filtering retains more of the overall intensity variation than mean filtering in this experiment.

These measurements describe changes in image statistics; they do not independently establish filtering quality or noise-removal effectiveness.

## 7. Implementation Notes and Limitations

1. **Boundary handling:** Variable neighbourhood sizes (9, 6 and 4 pixels) are used at interior, edge and corner locations. The paper's exact boundary convention has not been established.
2. **Zero variance:** Local mean is assigned when local variance is zero to prevent division by zero.
3. **Negative weights:** The paper's printed equation can produce negative weights when local variance is below global noise variance. These are retained in the current implementation; any clipping behaviour in the original implementation remains unverified.
4. **Numerical precision:** Floating-point arrays are used to preserve precision during subtraction, squaring and division.
5. **Validation:** The implementation has been tested on one spectral band from one turmeric sample, not the complete dataset.
