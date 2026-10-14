# Are factory measurements telling us the same thing twice?

**I compared 15 measurements from one Bosch production station. Two moved almost identically, even though neither was an exact duplicate of the other.**

**0.999994** pairwise correlation · **15/15** matrix rank · **227** complete records

That is why I looked beyond a simple rank check: a matrix can be full rank and still contain measurements with very similar patterns.

![Similarity matrix for five measurements at station L1_S25. The F1869 and F2147 pair has a normalized dot product of 0.999994.](figures/measurement_similarity.png)

## What I found

| Check | Result |
|---|---:|
| Measurements at one production station | **15 columns** |
| Rows with all 15 measurements present | **227** |
| Matrix rank | **15 of 15** |
| Directions required to pass 95% of squared variation | **9 of 15** |
| Squared variation represented by those 9 directions | **97.99%** |
| Smallest leave-one-out reconstruction residual | **0.003367** |
| Similarity of `L1_S25_F1869` and `L1_S25_F2147` | **0.999994** |

These values come from the **saved outputs of the notebook in this repository**. The analysis uses the first 100,000 rows of the public [Bosch Production Line Performance dataset](https://www.kaggle.com/competitions/bosch-production-line-performance/data), then selects complete rows for 15 measurements at station `L1_S25`.

The two measurements named above have a dot product of 0.999994 after centering and unit-length normalization. That is also their sample Pearson correlation on these 227 rows. It says they vary almost identically in this subset; it does **not** tell us whether they are physically separate sensors.

![Reconstruction errors of the 15 selected measurements, calculated from the notebook's saved output.](figures/reconstruction_residuals.png)

`L1_S25_F1869` has the smallest residual (0.003367), followed closely by `L1_S25_F2147` (0.003369). Both are reconstructed remarkably well using the other 14 columns.

## How I checked it

I stayed within one station rather than mixing unrelated measurement locations. The notebook takes the numeric Bosch data, groups features by station, and selects the 15 columns with the most non-missing values at `L1_S25`. Keeping only the rows where all 15 are present leaves a **227 × 15** matrix.

From there, the calculations answer slightly different questions:

1. **Rank:** Are any columns exactly linearly dependent? No — the original matrix has rank 15.
2. **Centering and normalization:** Can the columns be compared without their offsets or scales dominating the calculation? Each centered column is divided by its Euclidean norm.
3. **SVD:** Are there directions in the normalized matrix that contribute very little? Yes. The smallest singular values are much weaker than the largest ones.
4. **Leave-one-out least squares:** Can each column be approximated from the other 14? This points to individual redundancy candidates.
5. **Dot products and the Gram matrix:** Which measurements move together? The `F1869` / `F2147` pair stands out.

![Cumulative squared singular values calculated from the notebook's stored singular values.](figures/svd_cumulative.png)

The first **8** singular directions account for **94.96%** of the sum of squared singular values. Adding the ninth raises that to **97.99%**. So nine directions are needed to cross a 95% threshold. This is a property of the *selected, normalized matrix*, not a claim that nine physical sensors could safely replace fifteen.

## A little more on the mathematics

Let `A` contain the 227 selected rows and 15 measurement columns. I subtract each column mean and scale each centered column to unit length:

$$
\tilde A_{:,j}=\frac{A_{:,j}-\overline{A}_{:,j}}{\|A_{:,j}-\overline{A}_{:,j}\|_2}.
$$

The singular value decomposition

$$
\tilde A = U\Sigma V^\mathsf{T}
$$

shows how much of the matrix's squared norm lies along each orthogonal direction. I inspect the cumulative quantity

$$
q_k=\frac{\sum_{i=1}^{k}\sigma_i^2}{\sum_{i=1}^{15}\sigma_i^2}.
$$

That describes the overall matrix. To find *which column* overlaps with the others, I remove one normalized column `x`, put the remaining 14 into `B`, and solve

$$
\hat\beta=\arg\min_\beta \|x-B\beta\|_2,
\qquad r=\|x-B\hat\beta\|_2.
$$

The smallest residual identifies the column most easily reconstructed by a linear combination of the others **on these same rows**. Finally, because the columns are centered and unit length, their dot products are correlations. I used those dot products to check whether the overlap was spread across several columns or concentrated in a particularly close pair.

## What the result does — and doesn't — say

The Bosch feature identifiers are anonymized, so I cannot assign a physical meaning to `F1869` or `F2147`. This is a small, complete-case subset of one station, not a survey of every operating condition. The least-squares reconstruction is evaluated on the same rows used to fit it, so it is **not an out-of-sample prediction result**.

I did not use the `Response` label, train a fault classifier, or test whether removing a real sensor would be safe. The question here was narrower: *how much independent linear information do these measurements contain?*

## Notebook and data

- **Code and saved results:** [`Bosch_Measurement_Redundancy.ipynb`](Bosch_Measurement_Redundancy.ipynb) — 12 code cells, no Markdown cells.
- **Dataset:** [Bosch Production Line Performance — Kaggle](https://www.kaggle.com/competitions/bosch-production-line-performance/data). Download `train_numeric.csv.zip` and extract `train_numeric.csv`.
- **Run in Colab:** Upload `train_numeric.csv` to `/content/`, open the notebook, and run from the first cell. It reads the first 100,000 rows.
- **Python packages:** NumPy and pandas. The plots in this README were generated from the notebook's saved numerical outputs, not from a separate execution against the CSV.

The dataset is not included in this repository.
