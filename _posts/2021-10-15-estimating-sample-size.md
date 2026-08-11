---
title: "Estimating sample size"
date: 2021-10-15
permalink: /blog/estimating-sample-size/
tags:
  - Statistics
  - power analysis
  - tutorial
---

We have all been there: you want to fill in your pre-data-collection registration for your new study and you need to input a number for the targeted sample size. Plus, you need to specify how you came to that number. You turn to your favourite software (most likely G\*Power) and, unless the design of the study is identical to ones you have used before, a thousand questions come to your head.

What does "number of groups" mean? The number of actual groups of participants? The number of levels in your variables? Also, what does "number of measurements" mean? The number of variables? The levels in them? Should I sum the levels of all variables? Should I multiply them? And, what is even worse, which measure of effect size should I use? d, eta, eta squared, F, the other weird Greek letter... Should I leave the one set by default in the software? I guess that must be the correct one, but then... how do I decide how much is a medium effect size?

If you want to dive deep and learn about all of that, you can go [here](https://osf.io/9bt5s/) and [here](https://www.frontiersin.org/articles/10.3389/fpsyg.2013.00863/full). But if you only want to know which numbers to use, rest assured, I got you covered. You just need to know the size of the effect you would care about and which one is your key analysis. You can use the table below as an orientation on effect sizes.

**Cohen's d** ([definition and calculation](https://www.statisticshowto.com/cohens-d/))
- 0.2 = Small effect size
- 0.5 = Medium effect size
- 0.8 = Large effect size

**Cohen's f** ([definition and calculation](https://www.statisticshowto.com/cohens-f-statistic-definition-formulas/))
- .10 = Small effect size
- .25 = Medium effect size
- .40 = Large effect size

*From: Cohen, J. (1988). Statistical power analysis for the behavioral sciences (2nd ed.). Hillsdale, NJ: Erlbaum.*

## Quick calculators by design

The original version of this post had interactive calculators embedded for each design below. Those widgets were tied to my old Google Sites page and didn't survive the move, but the designs they covered — and where to get the numbers instead — are still useful:

- **One-tailed t-test (matched samples)**
- **Two-tailed t-test (matched samples)**
- **One-way repeated-measures ANOVA (3 levels)**
- **Mixed design: 3 (within) by 3 (between)**

For any of these, [G\*Power](https://www.psychologie.hhu.de/arbeitsgruppen/allgemeine-psychologie-und-arbeitspsychologie/gpower) remains the standard free tool — pick the matching test family and design under "Test family → t tests / F tests" and plug in the effect size from the table above. If you prefer to stay in R, the [`pwr` package](https://cran.r-project.org/web/packages/pwr/index.html) covers all four designs (`pwr.t.test()`, `pwr.anova.test()`, and `pwr.f2.test()` for the mixed design via the general linear model), and [WebPower](https://webpower.psychstat.org/wiki/) has free online calculators for the same tests if you'd rather not install anything.
