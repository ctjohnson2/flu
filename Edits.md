# Edits

## Early Stopping

Models are optimized with Adam, initialized at a base learning rate $\eta_0$ (e.g., $10^{-4}$). If training loss fails to improve for 10 consecutive epochs, the learning rate is decayed by a factor of 10 and weights are reset to the best training-loss checkpoint observed so far; training terminates once the learning rate falls below $10^{-6}$.

At each epoch, the model is additionally scored on a held-out validation window using a combined criterion that balances overall fit against peak-height accuracy:

$$\mathcal{L}_{\text{combined}} = \sqrt{(y_{\text{peak}} - \hat{y}_{\text{peak}})^2 + (\alpha \cdot \mathcal{L}_{\text{train}})^2}$$

where $y_{\text{peak}}$ and $\hat{y}_{\text{peak}}$ are the true and predicted maxima over the validation window, $\mathcal{L}_{\text{train}}$ is the epoch's training loss, and $\alpha$ is a weighting hyperparameter. The checkpoint deployed for inference is the epoch minimizing $\mathcal{L}_{\text{combined}}$, rather than the final or most-trained epoch. At $\alpha = 0$, checkpoint selection is driven by peak accuracy alone; increasing $\alpha$ progressively favors epochs with lower training loss.

## Weekly Forecast Performance

Week-to-week forecast accuracy was evaluated separately for individual models and for the combined ensemble, over each season's forecast window (weeks 30 through 17 of the following calendar year). For each of the seven constituent architectures (RNN-Lin1, GRU-Lin1, LSTM, LSTM-Lin1, LSTM-LinComb, RNN-LinComb, GRU-LinComb), we report root mean squared error (RMSE), mean absolute error (MAE), and mean absolute percentage error (MAPE) against the observed weekly hospitalization rate [Table X]. MAPE is computed only over weeks whose observed rate exceeds 5% of that window's peak, since the metric is undefined (and numerically unstable) near zero; RMSE and MAE are computed over the full window.

Three ensemble combination strategies were compared at the same weekly resolution: (1) a **Peak-Aligned** ensemble, which time-shifts each model's forecast so that its individual peak coincides with the ensemble's mean peak week before averaging, avoiding the amplitude attenuation that naive pointwise averaging produces under timing disagreement; (2) a **Width-Adjusted** variant, which additionally rescales the Peak-Aligned curve's time axis around the peak by a width-correction factor $r$ derived from comparing the ensemble's apparent peak width to the true, historically observed width of a validated season ($g(t) = f(t_p + r(t-t_p))$); and (3) a **Lorentzian** ensemble, which fits a smooth single-peak curve to each model's output and combines the fitted shape parameters via Monte Carlo resampling. All three are evaluated with the same RMSE/MAE/MAPE metrics as the individual models [Table Y], allowing direct comparison of whether combination improves on the best individual architecture.

Beyond point accuracy, we assessed whether each ensemble method's stated uncertainty is trustworthy via a calibration analysis: at each of three nominal confidence levels (50%, 80%, 95%), we constructed prediction intervals ($\text{Mean} \pm z\sigma$) at every week and measured the empirical fraction of observed values falling inside, pooled across all three seasons ([Table/Figure Z], reliability diagram). A method whose empirical coverage matches its nominal level is well-calibrated; coverage below nominal indicates overconfident (too-narrow) intervals, and coverage above nominal indicates overly conservative ones.

[Summarize here: which architecture(s) had the lowest weekly RMSE/MAE/MAPE; whether ensembling improved on the best individual model; which combination method was best calibrated and which was over/under-confident.]

## Seasonal Outcome Forecasts

In addition to week-to-week accuracy, we evaluated each method's ability to recover three season-level outcomes of primary public-health relevance: the cumulative season total, the peak weekly hospitalization rate, and the timing (calendar week) of that peak. These quantities were compared against the corresponding actual season outcome for each of the three evaluated seasons (23-24, 24-25, 25-26), across the Peak-Aligned, Width-Adjusted, and Lorentzian ensembles, as well as a 6-model subset variant of each [Table W].

For each method and season, we report the point estimate alongside its associated uncertainty: for peak height, the standard deviation of the underlying curve at the estimated peak week; for the season total, half the spread between the summed lower and upper uncertainty bands. [Figure V] presents these season total vs. peak height, faceted by season and colored by method, with the actual observed outcome marked separately, allowing visual comparison of which method's estimates most closely track the true seasonal trajectory and whether any method shows a systematic over- or under-estimation bias across seasons.

[Summarize here: which method's season total / peak height / peak week estimates were closest to actual across the three seasons; whether the full model list or the 6-model subset performed better; whether any consistent bias (e.g., systematic over-prediction of peak height) is apparent.]

## Hyperparameter Selection

Training hyperparameters (learning rate, epoch budget, training window, and validation/forecast window length) were selected by manual tuning during development and held fixed across all ten architectures, so that differences in forecast performance between architectures reflect model design rather than per-model hyperparameter optimization. No systematic search (grid, random, or Bayesian) was performed; values were chosen to give stable, converging training behavior under manual inspection, not to maximize validation accuracy.

The one exception is the random seed used to initialize each architecture's weights, which is set per architecture rather than shared: certain architectures failed to leave their initialization, or diverged, under a common seed, so each was assigned the smallest seed value found to reliably initiate training for that architecture. Seeds were not tuned for downstream forecast accuracy.
