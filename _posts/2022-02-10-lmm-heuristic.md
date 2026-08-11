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

**In order to do 2)**, you can use a Chi-square likelihood ratio test. In R, this is what the `summary()` function does.

Pro tip: you can get the effect size (odds ratio) by exponentiating the beta estimate. In R, `exp()` will do it for you.
