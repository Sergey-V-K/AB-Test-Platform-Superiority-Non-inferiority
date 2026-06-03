# A/B Test Calculations: Non-Inferiority and Superiority Cases

> **Abstract.** This document explains a common interpretation mistake in A/B testing.
> If a superiority test reaches the planned sample size but shows no statistically
> significant difference, that does **not** mean the treatment is no worse than the
> predefined MDE. Superiority and non-inferiority tests answer different questions.
> With a numerical example we show that even at the required sample size and with no
> significance, the lower bound of the confidence interval can still fall below the
> acceptable margin. So the absence of significance does not guarantee that the
> treatment drop is within the allowed limit.

**Key point:** *A non-significant superiority test does not guarantee that the treatment effect lies within the acceptable margin.*

---

## How to read this document

It answers one practical question:

> *If an A/B test reached the required sample size but showed no statistical significance, can we conclude that **treatment ≥ (control − MDE)**?*

The answer is **no**.

Structure:

- **Goal** — defines the practical question and the scenario.
- **Main principles** — key concepts: superiority vs non-inferiority, the role of confidence intervals, and MDE vs margin.
- **Decision matrix** — how to read outcomes depending on superiority and non-inferiority results.
- **Counterexample** — a numerical case that reaches the required sample size, shows no significance, yet still fails the non-inferiority condition.
- **Practical procedure** — how to find the worst-case drop and evaluate non-inferiority with confidence intervals.

---

## 1. Goal

After a two-sided A/B test with no statistical significance, find out how much worse the treatment could actually be.

| Key question | Answer |
| --- | --- |
| If we reached the required sample size but saw no significance, can we say: **treatment ≥ (control − MDE)?** | **NO** |

Below is the proof with examples.

---

## 2. Main Principles

### 2.1 Superiority and Non-Inferiority answer different questions

|  | Superiority | Non-Inferiority |
| --- | --- | --- |
| **Question** | Is there a difference? | Is the drop within the acceptable limit? |
| **H₀** | treatment = control | treatment is worse by ≥ margin |
| **H₁** | treatment ≠ control | treatment is worse by < margin |
| **Decision** | p-value < α | CI lower bound > −margin |

- Not finding an answer to the first question tells us nothing about the second.
- "We did not prove a difference" does not mean "the difference is small enough."

### 2.2 Look at the CI, not the p-value

In a non-inferiority test, the decision depends on one thing: where the lower bound of the confidence interval sits relative to the margin.

- Lower bound of 95% CI for (treatment − control) is **above −margin** → **NI is shown → we can ship**
- Lower bound is **below −margin** → **NI is NOT shown → we cannot ship**

Notes:

- The superiority p-value alone is not sufficient for this decision.
- Standard (FDA, industry): use a two-sided 95% CI. This is the same as a one-sided test with α = 0.025. Check the lower bound of the two-sided 95% CI: if it is above (−margin), NI is shown.

### 2.3 Margin is a business decision — set it BEFORE the test

The margin (acceptable drop) is not a statistical parameter. It is a product or business decision:

> *"We are OK with losing at most X p.p. of conversion in exchange for a cleaner UI."*

Set the margin before the test starts. Do not change it after you see the results.

**Important: margin ≠ MDE**

- **MDE** (minimum detectable effect) is about test power: what effect size the test can detect.
- **Margin** is about business tolerance: what drop we are willing to accept.
- In practice they often have the same number, but they mean different things.

---

## 3. Decision Matrix

| Superiority | NI shown? | Decision | Comment |
| --- | --- | --- | --- |
| Treatment better | Yes | **Ship** | Treatment is better. |
| Not significant | Yes | **Ship** | Not better, but the drop is acceptable. |
| Not significant | No | **Don't ship** | Cannot guarantee the drop is OK. |
| Treatment worse | Yes | **Ship\*** | Worse, but within margin. Product decides. |
| Treatment worse | No | **Don't ship** | Worse beyond the acceptable limit. |

**Row 3 is the case from the ticket.** "No significance" + "reached sample size" is not enough. **You must check the CI.**

---

## 4. Counterexample

We want to show that **"no significance + reached sample size" does NOT mean "we can ship."**

### 4.1 Setup

- Baseline conversion rate: **11.3%**
- MDE (for superiority test): **0.1 p.p.**
- Acceptable drop (NI margin): also **0.1 p.p.** *We set margin = MDE on purpose, to answer the ticket question.*
- Sample size for the superiority test → **~1,576,000 per group** (see [calculator](https://www.evanmiller.org/ab-testing/sample-size.html))
- "Run" the test: treatment = **11.25%** (slightly below control)

### 4.2 Result

| Parameter | Value |
| --- | --- |
| Superiority p-value | 0.16 → NOT significant |
| 95% CI of the difference | [−0.120, +0.020] p.p. |
| −Margin | −0.100 p.p. |
| CI lower bound vs −margin | **−0.120 < −0.100 → NI NOT shown** |

**Conclusion:** there is no significance, and we reached the sample size. But the lower bound of the CI (−0.120 p.p.) is below −margin (−0.100 p.p.). We **cannot** guarantee that treatment is worse than control by at most 0.1 p.p.

- The claim "treatment ≥ (control − MDE)" is **NOT** valid.
- We **cannot** automatically ship this change.

### 4.3 Why does this happen?

With standard parameters (α = 0.05, power = 80%) for a superiority test:

```
SE ≈ MDE / (Z(0.975) + Z(0.80)) ≈ MDE / (1.96 + 0.842) ≈ MDE / 2.8
```

The 95% CI goes about this far in **each direction** from the observed difference:

```
Z(0.975) × SE ≈ 1.96 × MDE / 2.8 ≈ 0.7 × MDE
```

If the observed difference is (−0.5 × MDE) (treatment is worse, but not significant), the lower bound of the CI is:

```
-0.5 × MDE − 0.7 × MDE = -1.2 × MDE
```

This is below −MDE. The NI margin is broken, even though the superiority test showed no significance.

**Key point:** a sample size for a superiority test with MDE = X **does NOT guarantee** that "no significance" means "treatment is no worse than X."

---

## 5. What to Do in Practice

### 5.1 After the test: calculate the lower bound of the 95% CI

```
lower_bound = (p_treatment − p_control) − Z(0.975) × SE
```

Then compare with (−margin):

- **lower_bound > −margin → NI is shown → we can ship**
- **lower_bound ≤ −margin → NI is NOT shown → we cannot ship**

This is the non-inferiority test.

### 5.2 Planning an NI test in advance

1. Set the margin (business decision).
2. Calculate sample size for the NI test.
3. Run the test.
4. Look at the CI, not the p-value.

### 5.3 Quick checklist

| Stage | Action |
| --- | --- |
| **Before test** | Set the margin (business decision). Calculate sample size. |
| **After test** | Calculate the 95% CI of the difference (treatment − control). |
| **Decision** | CI lower bound > −margin → Ship. Otherwise → Don't ship. |

### 5.4 Worst-case drop (by hand)

```
lower_bound = (p_treatment − p_control) − Z(0.975) × SE
```

This answers the question:

> *"With 97.5% confidence, treatment is worse than control by at most max(0, −lower_bound) p.p."*

The full calculator (`ni_calculator` function) is in the attached Jupyter Notebook.

---

## 6. Summary

- **"No significance" ≠ "we can ship."** Superiority and NI are different tests. They answer different questions.
- **Use the CI to decide.** Lower bound of two-sided 95% CI above −margin → NI is shown → ship.
- **Margin is a business decision.** Set it before the test. "We are OK with losing at most X p.p."
- **Margin ≠ MDE.** MDE is about test power. Margin is about acceptable drop. They can be the same number, but mean different things.
- **The counterexample is real.** Even with the right sample size for superiority, no significance does not mean the drop is within MDE.
- **In practice:** use `ni_calculator()` from the notebook. It gives both superiority and NI results in one place.
