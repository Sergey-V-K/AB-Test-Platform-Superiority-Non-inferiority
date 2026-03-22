# A/B Test Calculator: Design & Analysis

This calculator helps you at two stages of an A/B test:

- **Design:** before the test — figure out how many users you need and how long to run.
- **Analysis:** after the test — determine whether to ship, based on both superiority and non-inferiority.

Use the toggle at the top to switch between modes.

**Assumptions:** binary metric (conversion rate), independent observations, large-sample approximation, fixed-horizon test. This tool covers the primary metric only; guardrail metrics should be checked separately.

---

## Design Mode

Plan your experiment before launch. The calculator gives you the required sample size for both superiority and non-inferiority tests, so you can cover both with a single run.

| Field | What it means | Example |
|---|---|---|
| **Baseline CR (%)** | Current conversion rate in the control experience | 11.30 |
| **MDE (p.p.)** | Minimum detectable effect — the smallest improvement you want to be able to detect | 0.10 |
| **NI margin (p.p.)** | Maximum acceptable drop — the worst-case decline you are willing to live with. Set this as a team before the test. | 0.10 |
| **Alpha** | False positive rate. 0.05 is standard for most tests. | 0.05 |
| **Power** | Probability of detecting a real effect. 0.80 is standard. | 0.80 |
| **Ratio (treatment/control)** | How many treatment users per one control user. 1.0 = equal split. 2.0 = treatment group is twice as large. | 1.0 |
| **Daily traffic** | Total users entering the experiment per day (both groups combined). Used to estimate duration. | 10,000 |

**Output:** sample size per group, total users, and estimated days — separately for superiority and non-inferiority. The recommendation at the bottom tells you which number to use (always the larger one, so both tests have enough data).

---

## Analysis Mode

After the test is finished, enter the actual results. Groups can have different sizes — this is handled correctly.

| Field | What it means | Example |
|---|---|---|
| **Conversions control** | Number of conversions in the control group | 56,500 |
| **N control** | Total users in the control group | 500,000 |
| **Conversions treatment** | Number of conversions in the treatment group | 56,250 |
| **N treatment** | Total users in the treatment group | 500,000 |
| **NI margin (p.p.)** | Same margin you agreed on before the test | 0.10 |
| **Alpha** | Same alpha you used in the design. Must match. | 0.05 |

**Output:** two charts, a decision, and a text report.

**Top chart (Superiority):** is the treatment better or worse? The confidence interval is shown relative to zero.

**Bottom chart (Non-Inferiority):** is the drop acceptable? The same confidence interval is shown relative to the margin line. The NI decision is based on whether the lower bound of the CI stays above -margin.

**Decision** is one of five outcomes:

| # | Superiority result | NI shown? | Decision | What it means |
|---|---|---|---|---|
| 1 | Treatment better | Yes | **Ship** | Treatment is better. |
| 2 | Not significant | Yes | **Safe to ship** | No difference detected, but the CI confirms the drop is within the acceptable margin. |
| 3 | Not significant | No | **Don't ship** | Cannot guarantee the drop is acceptable. CI crosses the margin boundary. |
| 4 | Treatment worse | Yes | **Safe to ship*** | Significantly worse, but within the margin. Product team decides. |
| 5 | Treatment worse | No | **Don't ship** | Worse beyond the acceptable limit. |

The calculator highlights which row applies to your test.

**SRM check:** if the group sizes look imbalanced (more than expected), the calculator prints a warning. This may indicate an allocation bug — check before trusting the results.

---

## When to use which mode

**Design** — when planning a new test. Run it once, agree on the margin with your PM and EM, then launch.

**Analysis** — when the test is done. Works for any A/B test, not just non-inferiority. If the result is "no significant difference" and you want to know whether shipping is safe — this is exactly what you need.

---

## Two things to remember

**Margin is a product decision.** It is not a statistical parameter. It means: "we are OK with losing at most this much." Always set it before the test starts. Do not change it after you see the results.

**Alpha must be the same in Design and Analysis.** If you designed the test with alpha = 0.05, analyze it with the same value. Changing alpha after seeing results invalidates the test.
