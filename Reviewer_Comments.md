# Reviewer Comments

## Reviewer #1

In general, the revision has substantially improved the technical quality of the manuscript thanks to the inclusion of the 2025-2026 season, the thorough evaluation of hyperparameters (α), and a more complete presentation of the error and uncertainty metrics.

However, I recommend a couple of minor observations to refine the discussion.

First, the text should clarify the observed limitation in the time lag of the epidemic peak prediction (such as the lag of approximately 7 weeks in 2024-2025). Second, it should briefly discuss how the model could, in the future, automate the selection of the curve width correction factor (w_adj) without relying on ad hoc data from the previous season.

Congratulations on your work.

## Reviewer #2

This manuscript presents an interesting and potentially useful approach for extending seasonal influenza forecasting from the conventional short-term horizon toward approximately five months. The central idea—using Southern Hemisphere influenza surveillance as an early predictor for subsequent Northern Hemisphere influenza activity—is relevant to public-health preparedness and resource planning. The use of RNN/GRU/LSTM architectures, dimensionality reduction, ensemble aggregation, and Lorentzian representations is technically interesting.

The revised manuscript has improved substantially in several areas. The authors have added a third evaluated season, expanded the discussion of limitations and potential predictors, clarified the distinction between predictors and confounders, and added conventional forecast metrics.

Nevertheless, I recommend major revision because several issues remain before the work can be considered methodologically robust enough for publication.

### Major Comments

**1. The number of independent evaluation seasons remains very limited.**

The revised manuscript now evaluates the 2023–2024, 2024–2025, and 2025–2026 seasons, which is an improvement. However, three independent test seasons are still insufficient to establish generalizability across influenza seasons with different epidemiological characteristics.

The authors should clearly frame the work as a proof-of-concept or preliminary long-range forecasting framework, rather than implying that the proposed method has been sufficiently validated for operational public-health deployment.

Where possible, the authors should perform a rolling-origin or leave-one-season-out retrospective evaluation using all eligible pre-COVID seasons. Even if the authors cannot use all historical seasons as independent tests, they should explain precisely why each season is excluded and quantify how many independent seasonal observations remain.

**2. Quantitative comparison with baseline forecasting methods is still missing.**

This is the most important remaining methodological concern.

The manuscript compares several RNN-family architectures and aggregation procedures, but does not adequately compare the proposed framework with established forecasting baselines. The previous reviewer specifically raised this issue, and the response essentially states that transformers were excluded because of limited data.

Excluding transformers may be justified, but this does not eliminate the need for baseline comparisons.

At minimum, the authors should consider comparing against:
- seasonal-naïve forecasting;
- previous-season alignment;
- moving-average or autoregressive baseline;
- a simple statistical time-series model such as ARIMA/ETS;
- a simple regression model using the same Southern Hemisphere predictors;
- the proposed RNN ensemble;
- Lorentzian post-processing versus the non-parametric/peak-aligned alternative.

This would establish whether the improvement comes from the neural-network architecture or simply from exploiting the strong seasonal structure.

**3. The adjusted validation loss requires stronger justification and sensitivity analysis.**

The manuscript uses a customized $MSE_{adj}$ containing peak height, peak timing, and training MSE, with $\alpha=0.1$. The authors provide some justification and Table 1 explores different values of α.

However, the selection of the criterion and the threshold used for accepting/rejecting models still appear somewhat empirical.

The authors should provide a more systematic sensitivity analysis covering:
- α values;
- the model acceptance threshold;
- peak-height weighting;
- peak-location weighting;
- early-stopping threshold;
- random seeds.

It would be particularly useful to demonstrate whether the final conclusions remain unchanged under reasonable alternative parameter choices.

**4. The Lorentzian post-processing procedure needs an ablation analysis.**

The Lorentzian representation is an interesting component of the manuscript, and the authors show that it can reduce the dimensionality of the seasonal curve. However, the transformation from potentially bimodal influenza seasons to a single-peaked Lorentzian introduces a substantial modeling assumption.

The authors acknowledge that many seasons contain two distinct peaks and that the proposed single-peak representation cannot reproduce multi-peak behavior.

A direct ablation experiment would strengthen the paper considerably:

Raw RNN → peak-aligned ensemble → width-adjusted ensemble → Lorentzian ensemble

with RMSE, MAE, MAPE, total seasonal burden, peak height, and peak timing reported for each stage.

This would demonstrate whether each additional processing step actually contributes to predictive performance.

**5. Prediction intervals require formal calibration analysis.**

The manuscript reports nominal 50%, 80%, and 95% prediction intervals derived from model variation. However, the authors explicitly acknowledge that the intervals can appear overconfident, particularly for 2024–2025 when peak timing is incorrect.

This is a major issue for a public-health forecasting application.

The manuscript should report empirical coverage, for example:
- nominal 50% interval vs observed coverage;
- nominal 80% interval vs observed coverage;
- nominal 95% interval vs observed coverage;

and preferably interval width as well.

If formal calibration is impossible because only three seasons are available, this limitation should be explicitly stated and the intervals should not be presented as conventional statistically calibrated confidence/prediction intervals without qualification.

Moreover, the authors should distinguish carefully between confidence intervals, prediction intervals, and uncertainty bands generated from model variation, as these are not necessarily equivalent.

**6. Model-derived uncertainty may underestimate total predictive uncertainty.**

The reported uncertainty appears to be based primarily on variation across selected model architectures. This does not necessarily capture:
- data uncertainty;
- reporting uncertainty;
- country-specific surveillance bias;
- parameter uncertainty;
- random initialization uncertainty;
- structural/model uncertainty;
- uncertainty in the Southern-to-Northern Hemisphere relationship.

The authors should clarify exactly what their uncertainty estimates represent and avoid interpreting them as comprehensive predictive uncertainty.

**7. The Southern Hemisphere–U.S. transferability assumption requires deeper epidemiological justification.**

The manuscript assumes that dominant influenza strains and related dynamics in the Southern Hemisphere provide useful information about the subsequent U.S. season. This is plausible, but the relationship is not deterministic.

The manuscript already acknowledges differences in vaccination, population behavior, healthcare utilization, viral evolution, and reporting practices.

I recommend adding a more explicit discussion of why the selected five countries are epidemiologically informative for the U.S. and whether the relationship is stable across years.

A useful additional analysis would be to report the lagged correlation distributions by country and season rather than primarily presenting aggregate evidence over 2013–2022.

**8. Candidate-country selection may introduce selection bias and should be described more carefully.**

The five countries—Australia, Chile, Paraguay, Argentina, and Uruguay—are selected based on lagged correlations with U.S. hospitalization data.

The authors should explicitly clarify:
- whether the country selection was performed only using training-era data;
- whether any information from the test seasons was used during candidate selection;
- whether the selected countries remain optimal when candidate selection is repeated within each training window.

A rolling candidate-selection procedure would provide stronger evidence that the five-country set is genuinely predictive rather than optimized for the available historical data.

**9. The normalization and denormalization procedure should be documented more rigorously.**

The revision notes that an error was previously identified in the denormalization step and has been corrected. The authors state that the error affected the absolute scale but not relative model comparisons.

Given that the manuscript is a public-health forecasting paper where the absolute hospitalization burden is important, this correction deserves especially transparent documentation.

The authors should provide:
- the exact normalization equation;
- the population factor used;
- the precise unit of the final target;
- an explicit statement confirming that all reported tables and figures use the corrected values;
- preferably a reproducible example showing transformation from normalized output back to hospitalization rate/count.

**Response:** We agree this required more explicit documentation and have added it directly alongside the code that performs the scaling. The normalization is a MinMax scaling of the raw weekly CDC FluView Phase 5 hospitalization count, `INF_ALL`, fit only on data through a fixed cutoff date:

$$X_{\text{norm}}(t) = \frac{X(t) - X_{\min}}{X_{\max} - X_{\min}}$$

For our data $X_{\min}=0$ (true zero-count weeks exist in the fitting window), so this reduces to $X_{\text{norm}}(t) = X(t)/X_{\max}$.

The population (catchment) factor is 0.09: the FluView Phase 5 hospitalization surveillance network does not cover the full U.S. population, and is assumed to capture approximately 9% of it. The scale constant used to convert normalized model output back to an estimated national count extrapolates the raw sentinel peak by this assumed catchment fraction:

$$\texttt{max\_} = \frac{X_{\max}}{0.09}, \qquad \hat{X}(t) = \hat{X}_{\text{norm}}(t) \times \texttt{max\_}$$

Every reported season total, peak height, and weekly forecast in the manuscript — including all RMSE/MAE/MAPE tables, calibration intervals, and the α-sensitivity results — is therefore in units of **estimated national hospitalization counts**, not raw sentinel-surveillance counts and not a per-capita rate. As a reproducible example: a model output of 0.05 in normalized units corresponds to $0.05 \times \texttt{max\_}$ estimated hospitalizations for that week. We confirm all tables and figures in this revision use this corrected scaling consistently.

We also note as a limitation that the 9% catchment assumption is applied as a single fixed constant across the entire study period (2013–2026); its year-to-year stability has not been independently verified, and this uncertainty is not propagated into the reported confidence/prediction intervals, which capture disagreement across models only.

**10. The distinction between hospitalization rates and hospitalization counts should be consistently maintained.**

The manuscript alternates between terms such as "cases," "incidence," "hospitalizations," "hospitalization rate," and "total cases."

Because the input consists of documented influenza-positive cases while the U.S. target is hospitalization data, these are not interchangeable quantities. The authors themselves acknowledge this fundamental difference.

The terminology should therefore be standardized throughout the abstract, Results, tables, and Conclusion. In particular, the phrase "total case count" should be reconsidered if the model actually estimates cumulative hospitalization burden/rates.

### Minor Comments

- Revise the abstract for grammar and precision.
- Clearly state the exact forecasting origin and horizon in weeks/months.
- Clarify the meaning of the 26-week input sequence and the 21-week shift. The rationale for these specific values should be explicitly justified.
- Define all symbols immediately below each equation and ensure units are provided where meaningful.
- Ensure consistency between $MSE_{adj}$, $\sqrt{MSE_{adj}}$, and the reported thresholds of approximately 0.2 and 0.3.
- Explain why one random seed is sufficient for each architecture. Ideally, multiple seeds should be evaluated, particularly because uncertainty is subsequently derived from model variation.
- Provide the number of trainable parameters for each architecture and explain whether model capacity is comparable across architectures.
- Report the number of models initially trained and the number retained after the validation criterion for each season. This information would make the model-selection process much more transparent.
- The statement that RNNs are preferable for small datasets should be presented more cautiously. Model suitability depends on the effective sample size, temporal structure, architecture, regularization, and baseline performance.
- The statement that the 2023–2024 peak was correctly predicted because the previous season had a similar peak is interesting, but it should be presented as an observation/hypothesis rather than a demonstrated causal mechanism.
- The conclusion should be moderated. The manuscript demonstrates promising retrospective forecasting performance, but the evidence is not yet sufficient to conclude that the method can reliably support operational public-health resource allocation.
- The repository should contain sufficient documentation to reproduce preprocessing, country selection, model training, model selection, ensemble construction, and all reported figures/tables. The manuscript currently states that models and data are available but would benefit from a more explicit reproducibility description.
- The authors should carefully proofread the manuscript for typographical artifacts and duplicated/incorrect phrases remaining from revision.
