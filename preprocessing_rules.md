# Preprocessing and Distance Rules for the 65 Clustering Features

This document records the implemented preprocessing applied before the 65 clustering features enter patient-pair distance calculations in the frozen primary analysis. All statistics are estimated from observed values only. The value `-1` is the missing-value code and does not contribute to statistical estimation or feature-level distances.

## 1. Continuous variables

### 1.1 Conditional `log1p` transformation

For each continuous variable, skewness, \(P_1\), and \(P_{99}\) are first calculated from observed raw values. The transformation is applied only when all of the following conditions hold:

1. All observed values are nonnegative.
2. Sample skewness is greater than 2.
3. \(P_{99}>0\).
4. \(P_1\le 0\), or \(P_{99}/P_1\ge 5\).

The transformation is:

\[
x'=\log(1+x).
\]

The variables that triggered this rule were myoglobin, BNP, alpha-HBDH, LDH, CK, D-dimer, platelet count, phosphorus, ALT, AST, alkaline phosphatase, urea, creatinine, random glucose, triglycerides, and lipoprotein(a).

### 1.2 Robust P1–P99 range

After any conditional transformation, \(P_1\) and \(P_{99}\) are recalculated on the current scale and the values are clipped:

\[
\tilde{x}=\min\{\max(x',P_1),P_{99}\}.
\]

The robust range is:

\[
R_k=P_{99,k}-P_{1,k}.
\]

If \(P_{99}\le P_1\), the code uses the minimum and maximum observed values instead. If the resulting range is still not greater than 0, \(R_k=1\).

The distance between patients \(i,j\) for continuous feature \(k\) is:

\[
d_{ijk}=\min\left(\frac{|\tilde{x}_{ik}-\tilde{x}_{jk}|}{R_k},1\right).
\]

## 2. Binary and nominal categorical variables

Sex is treated as binary. Chief complaint category, smoking status, and alcohol-use status are treated as nominal. Numeric category codes are not interpreted as having a continuous order.

\[
d_{ijk}=
\begin{cases}
0,&x_{ik}=x_{jk},\\
1,&x_{ik}\ne x_{jk}.
\end{cases}
\]

Smoking and alcohol-use status use 0 = never, 1 = current, 2 = former, and 3 = unknown in the complete audit dataset. Code 3 is converted to `-1` in the primary clustering input, so “unknown” does not contribute as an observed category. Chief complaint codes 1–6 are matched only as nominal categories.

## 3. Circular ECG variables

The following three variables are treated as circular angles:

- QRS axis
- P axis
- T axis

Their distance is:

\[
d_{ijk}^{\mathrm{angle}}=
\frac{\min\left(|a-b|,360-|a-b|\right)}{180}.
\]

Zero degrees is valid for all three ECG axes. It is not converted to missing and is not subjected to ordinary linear P1–P99 clipping.

## 4. Derived pack-years feature

For current or former smokers, cigarettes per day and smoking duration are first checked against the source values. The original pack-years value is set to missing when any of the following conditions hold:

- Cigarettes per day are less than or equal to 0.
- Cigarettes per day are greater than 100.
- Smoking duration is less than or equal to 0 years.

The 99th percentile of valid pack-years is 100. The final primary clustering feature is therefore:

\[
x_{pack}=\log\left(1+\min(\mathrm{Pack\text{-}years},100)\right).
\]

The derived feature is subsequently processed as a continuous variable with the robust P1–P99 range, but `log1p` is not applied a second time.

## 5. Unified troponin percentile

Conventional cardiac troponin I and high-sensitivity cardiac troponin I are separately converted to mean-rank empirical percentiles within their respective assay-specific observed samples:

\[
p_i=\frac{R_i}{n},
\]

where \(R_i\) is the average rank assigned to tied values and \(n\) is the number of observed values in that assay system. The patient-level unified measure is:

\[
T_i=
\begin{cases}
p_{cTnI,i},&\text{only conventional cTnI observed},\\
p_{hs\text{-}cTnI,i},&\text{only hs-cTnI observed},\\
\left(p_{cTnI,i}+p_{hs\text{-}cTnI,i}\right)/2,&\text{both observed},\\
NA,&\text{both missing}.
\end{cases}
\]

The final feature is `unified_troponin_percentile`. The assay-type indicator is retained only for audit or sensitivity analysis and does not enter the primary clustering distance.

## 6. Unified CK-MB percentile

CK-MB activity and CK-MB mass use the same mean-rank percentile rule. The assay-specific value is \(R_i/n\). If only one assay is observed, its percentile is used; if both are observed, their mean is used; and if both are missing, the unified value is missing. The final feature is `unified_CKMB_percentile`. The assay-type indicator does not enter the primary clustering distance.

## 7. Invalid-value handling

Zero is converted to missing for the following ECG fields: PR, HR, QT, QTc, P-wave duration, and QRS duration. The same rule is not applied to QRS axis, P axis, T axis, SV1, or RV5.

Vital signs use the following prespecified conservative valid ranges:

| Variable | Accepted range | Action outside range |
| --- | ---: | --- |
| Respiratory rate | 5–60 breaths/min | Set to missing |
| Systolic blood pressure | 40–300 mmHg | Set to missing |
| Diastolic blood pressure | 20–200 mmHg | Set to missing |
| Physical-examination heart rate | 20–250 beats/min | Set to missing |

Out-of-range values are explicitly set to missing rather than clipped to the boundary.

## 8. Within-view distance and patient-pair reliability

The 65 features are grouped into four views. For patients \(i,j\) and view \(v\), only features observed in both patients are used. Let \(p_v\) denote the number of features in the view and \(O_{ij}^{(v)}\) the set of jointly observed features. Then:

\[
D_{ij}^{(v)}=
\frac{1}{|O_{ij}^{(v)}|}
\sum_{k\in O_{ij}^{(v)}}d_{ijk},
\]

\[
r_{ij}^{(v)}=\frac{|O_{ij}^{(v)}|}{p_v}.
\]

If a view has no jointly observed feature for a patient pair, it is excluded from that pair’s fused distance. All four views have a global weight of 1 in the primary analysis:

\[
D_{ij}=
\frac{\sum_v r_{ij}^{(v)}D_{ij}^{(v)}}
{\sum_v r_{ij}^{(v)}}.
\]

Missingness patterns therefore affect only patient-pair reliability in the corresponding view and do not enter the primary clustering model as a separate high-weight feature set.

## 9. Variables excluded from primary clustering

`label` denotes one-year all-cause mortality and is used only for post-clustering external validation. Heart-function category is also used only for external validation; original code 0 indicates missing and is converted to `-1` in the output, while valid categories 1–4 are interpreted nominally. Troponin assay type, CK-MB assay type, and all missingness indicators are excluded from the 65 primary clustering features.
