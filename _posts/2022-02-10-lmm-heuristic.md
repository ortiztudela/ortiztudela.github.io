---
title: "A heuristic for LMM model selection"
excerpt: "Hopefully, this will help as a guide for your thinking through the process."
date: 2022-02-10
permalink: /blog/lmm-heuristic/
tags:
  - Statistics
  - LMM
  - R
  - tutorial
---

Heuristic for using LMMs as a two-step process. This helps me conceptualise the process. 

# How to?

The two steps are: 1) determine whether the slopes of your effects differ, 2) test your main effects.

**In order to do 1):**

- Build the more complex (maximal) model.
- Check convergence with a large number of iterations.
- Build a reduced model by removing 1 element.
- Compare models in descending order: if there is a significant reduction in fit, accept the more complex model; if there is no decrease, continue descending.
- If further descending is needed, remove 1 element from the previously reduced model and repeat the process.
- Continue until a significant reduction in fit is found.
- Pro tip: the suggested reduction order is interactions > main effects.

**In order to do 2)**, you can use a Chi-square likelihood ratio test. In R, this is what `anova(reduced_model, full_model)` does (not `summary()`, which only prints the coefficient estimates).

Pro tip: if you're running a logistic (binomial) mixed model, you can get the effect size as an odds ratio by exponentiating the beta estimate — in R, `exp()` will do it for you. This only applies to logistic/binomial models; for a standard linear mixed model with a continuous outcome, the beta is already on the outcome's scale and there's no odds ratio to compute.
