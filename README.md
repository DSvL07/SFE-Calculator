# SFE-Calculator

Predicts the stacking fault energy of an austenitic steel from its composition and temperature,
with an uncertainty range that has been validated against held-out laboratories, and a flag when
the alloy falls outside the limits of the training data.

Wei Xiong and XinHang Shi · Physical Metallurgy and Materials Design Laboratory,
Department of Mechanical Engineering and Materials Science, University of Pittsburgh

---

## Inputs and outputs

Temperature (K) and wt.% of C, Cr, Mn, Mo, N, Ni, Si, Al, with iron as the balance. Returns SFE
in mJ/m² with 50% and 90% intervals and a three-level applicability flag. Spreadsheets
(`.xlsx`, `.csv`, `.tsv`, `.txt`) are accepted with automatic column matching; see
`examples/example_alloys.csv`.

Five models are exposed side by side — a linear equation, extra trees, random forest, gradient
boosting, and a single decision tree — each walked in the browser rather than approximated.

## Validation

The training rows are not independent. Each comes from a published paper, and a paper is one
laboratory, one instrument, one calibration. Splitting rows at random puts measurements from the
same paper on both sides of the split, which lets a sufficiently flexible model recover that
laboratory's systematic offset and replay it at test time. The result is an accuracy figure that
does not survive contact with a new laboratory.

Every figure below therefore holds out **whole source papers**.

| Model | Papers held out | Rows shuffled | Gap |
|---|---|---|---|
| **Lab-corrected equation** | **7.84** | 7.77 | **+0.07** |
| Extra Trees (200) | 8.37 | 5.55 | +2.82 |
| Random Forest (200) | 8.54 | 5.76 | +2.78 |
| Gradient Boosting (100) | 8.72 | 5.80 | +2.92 |
| Decision Tree (depth 5) | 9.89 | 7.16 | +2.73 |

MAE, mJ/m². The gap column is the part of each model's apparent skill that was laboratory
recognition rather than chemistry. A 16-term linear equation with per-paper intercepts —
estimated during fitting, discarded at prediction — outperforms every ensemble under the
honest protocol, and is the only model whose advantage does not evaporate when the split
changes.

Intervals were built the same way: each of the 59 source papers was removed in turn, the model
refitted without it, and its alloys predicted blind. The nominal 90% interval achieved **90.3%**
empirical coverage. Conditional coverage holds at each flag level (90.6 / 89.5 / 89.5%); without
widening outside the tested composition limits it degrades to 74% while still claiming 90%.

### On the width of the interval

Across 181 pairs of near-identical steels measured in *different* laboratories, the two reported
values differ by 6.5 mJ/m² (median). The equation's own typical miss is 7.2. The model is at the
reproducibility floor of the source literature, and the interval width reflects that rather than
slack in the fit.

## Data

Training data is the compilation published with Wang & Xiong (2020), CC BY 4.0: 349 measurements
from 52 papers, plus 33 rows from the same workbook de-duplicated against it and filtered for
elements the model does not carry — 382 measurements from 59 papers.

## Limitations

- Inputs are limited to C, Cr, Mn, Mo, N, Ni, Si and Al. Ti, Nb, Cu, Co, W and V are **treated as
  iron**, so grades carrying them (321, 347, 904L) are approximated.
- Aluminium appears in 32 of 382 rows. Its coefficient is large and the least well constrained in
  the model.
- Calibrated over 94–598 K. The interface flags temperatures outside that.
- Quoted accuracy is for alloys from laboratories absent from the training set. Predictions for
  rows already in the compilation will appear better than the model is.
- Fitted on 382 measurements. This is a screening and design tool, not a replacement for
  measurement.

## Citation

Please cite the source dataset:

> Wang, X. and Xiong, W. Stacking fault energy prediction for austenitic steels: thermodynamic
> modeling vs. machine learning. *Science and Technology of Advanced Materials* **21**(1), 626–634
> (2020). https://doi.org/10.1080/14686996.2020.1808433

and this calculator, via the *Cite this repository* button (reads `CITATION.cff`).

## Licensing

Code and interface: **MIT** (`LICENSE`).

The embedded training data is **CC BY 4.0**, © the authors of Wang & Xiong (2020). `index.html`
contains both, so reuse of the file carries the attribution requirement that MIT alone would not
impose. See `NOTICE.md`.
