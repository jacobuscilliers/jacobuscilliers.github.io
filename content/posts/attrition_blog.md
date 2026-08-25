---
title: "You are not testing for attrition correctly"
date: 2026-08-24
description: "Two joint tests for attrition bias in field experiments, and simulation evidence on which version to run in stratified, clustered designs."
draft: true
tags: ['RCTs', 'attrition', 'econometrics', 'methods']
---

It is a Wild West out there when it comes to testing for attrition in field experiments. Researchers use different tests, describe them with different terms, and somehow always find a way to conclude that attrition is not a problem in their case. So I was happy to read [Testing Attrition Bias in Field Experiments](https://jhr.uwpress.org/content/early/2023/08/02/jhr.0920-11190r2) by Dalia Ghanem, Sarojini Hirshleifer and Karen Ortiz-Becerra (Journal of Human Resources, 2023; [ungated version](https://escholarship.org/uc/item/4ck087v3)). It turns out I was doing it wrong. Chances are, so are you.

The authors review 96 published field experiments with baseline data. The most common test — reported in 79 percent of the experiments with attrition above 1 percent — checks whether attrition *rates* differ between treatment and control. But equal rates are neither necessary for an unbiased estimate nor, without extra assumptions, sufficient: rates can differ while your estimate is fine, and identical rates can hide serious bias. The second most common test, in 61 percent, checks whether treatment and control *respondents* are balanced at baseline. Closer, but only half of the correct test.

## The two tests

To see this, split your baseline sample by treatment status and by endline response:

|               | **Respond** | **Attrite** |
|---------------|-------------|-------------|
| **Treatment** | TR          | TA          |
| **Control**   | CR          | CA          |

Ghanem and co-authors distinguish two questions:

**Test 1: Is the comparison of treatment and control respondents unbiased, for the respondents?** The paper calls this internal validity for respondents (IV-R). The identifying assumption is that treatment is as good as randomly assigned *conditional on response status*. Its testable implication is that **both** columns are balanced at baseline, jointly: TR = CR *and* TA = CA. I used to test each separately.

**Test 2: Is that effect also the effect for your original sample?** The paper calls this internal validity for the study population (IV-P). The testable implication is that **all four cells are equal**. I will call them the respondent test and the representativeness test.

In regression form, using the baseline outcome and the full baseline sample:

*Y*<sub>0</sub> = α + β<sub>T</sub> Treat + β<sub>A</sub> Attrite + β<sub>TA</sub> Treat × Attrite + γ + ε,

where γ are strata fixed effects (more on those below); for clustered designs, standard errors are clustered at the unit of randomization. The respondent test is the joint test β<sub>T</sub> = β<sub>TA</sub> = 0. The representativeness test adds one restriction: β<sub>A</sub> = β<sub>T</sub> = β<sub>TA</sub> = 0. Strictly, the paper's tests compare entire baseline *distributions*; the regression tests the difference in means, which is what most of us will run, and the two coincide for binary variables.

The authors provide a Stata command that does all of this: `ssc install attregtest`.

## Why also test balance among attritors?

My instinct was that we only care about balance in the sample of respondents, since that is our estimating sample. That is wrong for two reasons. First, the assumption you need, treatment as good as random given response status, has implications for attritors too. Testing respondents only uses half of the testable implications. Second, the respondent-only test is often badly underpowered. Suppose your program keeps some weak students enrolled who would otherwise have dropped out and vanished from the sample. With low attrition, these program-induced responders might be 2 percent of your treatment respondents, far too few to detect in a balance test. But their counterparts in the control group are *attritors*, where they can be a large proportion of a small group. 

## What about stratified randomization?

If you randomized within strata, there are three ways to run the test. The first two work *within each stratum*, interacting everything with strata dummies: either one joint test across all strata, or a separate test per stratum followed by a multiple-testing correction that controls the family-wise error rate (FWER) or the false discovery rate. The authors motivate the per-stratum version for the case where response problems hit some strata but not others: you stratified by region, say, and one region is remote or flood-prone, so your sample there is hard to reach. The third way is simpler: the regression above, with strata fixed effects and no interactions.

I have two concerns with the within-strata versions.

**1. The joint test over-rejects the null, badly so in clustered trials.** Interacting the test with strata multiplies the number of restrictions you test at once: with ten strata, the representativeness test already stacks thirty. Joint tests with many restrictions are known to over-reject when standard errors are clustered (see [MacKinnon, Nielsen and Webb 2023](https://www.sciencedirect.com/science/article/pii/S0304407622000781) and [Kerwin, Rostom and Sterck](https://www.iza.org/publications/dp/17217/striking-the-right-balance-why-standard-balance-tests-over-reject-the-null-and-how-to-fix-it)). I asked Claude to run some simulations (first note at the end; I did not verify the code, but it is consistent with the theory). With 100 clusters, even four strata is too many for the test to hold its size, and with ten strata the within-strata representativeness test rejects 86 percent of the time when nothing is wrong. The same problem arises with heteroskedasticity-robust standard errors under individual-level randomization ([Anatolyev and Sølvsten 2023](https://arxiv.org/abs/2003.07320)), but there it is far less severe: in the simulations, it only starts to bite once strata fall below roughly 200 observations each.

**2. Lower statistical power.** The joint test has lower power because of many degrees of freedom adjustments, and the FWER correction is conservative. That hands researchers a way to falsely conclude that attrition is not a problem. A second set of simulations (second note at the end), with individual-level randomization so that over-rejection is not the issue, shows that the ranking depends on where the violation sits. If it is diffuse — slight bias in every stratum — the fixed-effects test has the most power and the per-stratum approach with an FWER correction the least. The ranking flips when the violation is concentrated in one stratum, the case the per-stratum approach was designed for.

So I believe it is more sensible to just use the fixed-effects test; the authors themselves used it in their empirical application whenever there were more than ten strata. `attregtest` reports both versions.

## What about multiple outcomes?

The authors propose that you run the test separately for each endline outcome, using that outcome's own baseline value and its own response indicator, and correct for multiple testing across outcomes. The paper uses Progresa to show why the per-outcome approach matters. School enrollment fails the representativeness test and adult employment passes it, even though both come from the same survey: households with more school-age children were more likely to skip the enrollment questions.

## What if you lack baseline data for an outcome?

Then you test on covariates instead, but not on any covariates. The paper is specific about which ones are admissible: determinants of the outcome, or proxies driven by the same unobservables as the outcome. So pre-specify a small set of outcome-specific determinants, and test them jointly with the baseline outcome, if you have it, in one system of equations. `attregtest` takes the whole list and reports one joint p-value per test. At this point the exercise looks a lot like a standard baseline balance table — except with a different omnibus test.

The same warning as above applies, though: with k variables, the respondent test stacks 2k restrictions and the representativeness test 3k, and joint tests with many restrictions over-reject in clustered designs. In the simulations, with 100 clusters, a joint representativeness test over ten variables falsely rejects 37 percent of the time (first note at the end). If you have more than two or three variables in a clustered design, you need another test. The most viable alternative is to adjust for multiple tests, using FWER or FDR. You reject the null if p or q is below your desired level of significance for at least one of the variables. Simulations show that it falsely rejects roughly 0.07-0.08 of the time for a 5 percent test, so a little bit more conservative. 

(Aside: my first instinct was using randomization inference instead, as [Kerwin, Rostom and Sterck](https://www.iza.org/publications/dp/17217/striking-the-right-balance-why-standard-balance-tests-over-reject-the-null-and-how-to-fix-it) recommend for balance tests, but it is not theoretically allowable since attrition is itself a post-treatment outcome: re-shuffling schools into treatment and control shuffles their response rates too, so the randomization-inference benchmark implicitly holds attrition unaffected by treatment (this is not a problem for individual-level randomization). Simulations show that they are not balanced, but power is actually lower than adjusting for MHTs using Anderson q-values. 

## An example from my own work

In our long-run follow-up of the [Early Grade Reading Study](https://jhr.uwpress.org/content/55/3/926) in South Africa, we found and assessed 67 percent of the treatment group (655 of 981 learners) and 66 percent of the control group (1,035 of 1,575). So attrition was high, but similar across arms. Here are the two tests for three baseline variables:

| Baseline variable | β<sub>A</sub> | β<sub>T</sub> | β<sub>TA</sub> | N | IV-R | IV-P |
|---|---|---|---|---|---|---|
| Female            | −0.10\*\*\* (0.02) | −0.02 (0.02) | 0.02 (0.04)  | 2,556 | 0.704 | <0.001 |
| Age               | 0.29\*\*\* (0.05)  | 0.03 (0.05)  | −0.07 (0.08) | 2,188 | 0.423 | <0.001 |
| Learning index    | −0.21\*\*\* (0.06) | 0.08 (0.16)  | −0.02 (0.11) | 2,556 | 0.887 | <0.001 |
| Joint (all three) |                    |              |              | 2,188 | 0.727 | <0.001 |

*Notes: β<sub>A</sub> is the coefficient on Attrite (the gap between control attritors and control respondents), β<sub>T</sub> the coefficient on Treat (the treatment–control gap among respondents), and β<sub>TA</sub> their interaction, from the regression above with strata fixed effects; standard errors, in parentheses, are clustered at the school level, the unit of randomization. The last two columns report p-values from `attregtest` for the respondent test (IV-R) and the representativeness test (IV-P), both in the strata-fixed-effects version. The Joint row tests all three variables in one system of equations. \* p<0.10, \*\* p<0.05, \*\*\* p<0.01.*

The respondent (IV-R) test never rejects, for any variable or jointly: treatment and control are balanced within both columns of the two-by-two table. So we cannot reject that our estimate is internally valid for the learners we found, at least for these baseline variables. But the representativeness (IV-P) test rejects for every variable, and jointly: the learners we lost are about a third of a year older, 10 percentage points less likely to be girls, and 0.2 standard deviations weaker on the baseline learning index — and equally so in both arms. Our estimates only speak for the two-thirds of the original sample we could still find. 

The within-strata version of the test tells a different story: it rejects the respondent test for the learning index (p = 0.004) and jointly (p < 0.001). Had we quoted those numbers, we would have concluded that the study is not internally valid even for respondents. But this is exactly the fragile test from my first concern above: with ten strata and 130 schools, the joint within-strata test stacks 60 (10 × 3 × 2) restrictions on 130 clusters — and even the single-variable version stacks 20. The simulations say such tests reject many times too often under a true null.

## The bottom line

Report attrition rates by arm with their cell sizes, but as a description of the response process, not as a test of validity. Report the four cell means and the two joint p-values, per outcome, from one regression on data you already have. Pre-specify the variables and, if you stratified, the version of the test — and in a clustered design with many strata or many variables, that version should be the fixed-effects one. If you jointly test many baseline variables in a clustered design, consider randomization inference instead. If the representativeness test fails, say plainly which population your estimate covers. If the respondent test fails, no test will rescue you: you are in the territory of attrition corrections and [bounds](https://academic.oup.com/restud/article-abstract/76/3/1071/1590712).

---

## Simulation note 1: joint tests with many restrictions in clustered designs

A cluster-randomized experiment with G schools of 20 learners, half of the schools in each stratum treated, 30 percent attrition unrelated to anything, and one baseline variable that is pure noise (intra-cluster correlation 0.15). Both tests should therefore reject 5 percent of the time. The tests are the standard regression tests as Stata computes them (cluster-robust standard errors, F critical values with G − 1 denominator degrees of freedom), in two versions: the strata-fixed-effects regression in the post, and the fully interacted within-strata version, whose restriction count grows with the number of strata (2S for the respondent test, 3S for the representativeness test). One thousand replications per cell.

{{< figure src="/images/attrition/wald_size_strata_fig.png" alt="False rejection rates of the fully interacted within-strata test, by number of strata and number of clusters" >}}

The fixed-effects version approximately holds its size everywhere: 4 to 10 percent for every combination of strata and clusters (the gray band). The within-strata version deteriorates as restrictions accumulate relative to clusters. With 100 clusters: at two strata the representativeness test falsely rejects 9 percent of the time, at four strata 20 percent, at ten strata 86 percent, and at twenty strata always. Even 400 clusters are not safe at ten strata (19 percent). In the extreme — two treated clusters per stratum — the within-stratum representativeness test is not even defined, and Stata silently drops constraints.

The problem compounds with a joint test on multiple baseline variables. With k variables, the respondent test has 2 × k restrictions and the representativeness test 3 × k; the within-strata versions multiply both by the number of strata. The next figure repeats the exercise in an *unstratified* design, varying the number of variables and the number of clusters (variables correlated at 0.3), and the trajectory is the same: with 100 clusters, a joint representativeness test over ten variables falsely rejects 37 percent of the time, and over twenty variables 97 percent. At the *same* number of restrictions, the strata version is worse — a within-stratum comparison draws only on the clusters in that stratum, while a variable's restrictions draw on all of them — but the practical rule is identical: keep the number of restrictions small relative to the number of clusters.

{{< figure src="/images/attrition/wald_size_vars_fig.png" alt="False rejection rates of the joint test across baseline variables, by number of variables and number of clusters" >}}

The same exercise with *individual-level* randomization and heteroskedasticity-robust standard errors (the outcome variance differs across strata) shows the same pattern in a much milder form. Size stays near 5 percent unless strata become small: with 2,000 observations the tests are close to nominal through ten strata, but with 800 observations and 16 strata the representativeness test rejects 36 percent of the time, and at 20 strata it frequently cannot be computed at all.

{{< figure src="/images/attrition/hc_strata_fig.png" alt="False rejection rates with individual-level randomization and heteroskedasticity-robust standard errors" >}}

Design caveats for these exercises: equal-sized strata, and variables correlated at 0.3 in the many-variables case.


## Simulation note 2: joint versus per-stratum tests

A stratified experiment with 10 strata of 200 units (n = 2,000), treatment assigned at the *individual* level within strata, a baseline outcome with a standard deviation of 15, and 10 percent never-responders. The respondent assumption is violated by adding treatment-only responders (10 percent of the sample) with a lower baseline mean, which contaminates both columns of the two-by-two table. Two geometries with the same total signal: *concentrated*, where the whole violation sits in one stratum, and *diffuse*, where it is spread evenly over all ten. Four tests of the respondent null, all regression-based: the fixed-effects regression in the post (2 restrictions); the fully interacted joint test across strata (20 restrictions); and per-stratum tests with a Bonferroni–Holm or a Benjamini–Hochberg correction. With individual-level assignment, all four hold size at 5 percent — the clustering problem from the first note does not arise. Power at a total violation of 30 baseline points, or two standard deviations (3,000 replications):

| Geometry                | Fixed effects | Joint test, all strata | Per-stratum + FWER | Per-stratum + FDR |
|-------------------------|---------------|------------------------|--------------------|-------------------|
| Diffuse (all 10 strata) | 0.74          | 0.33                   | 0.18               | 0.19              |
| Concentrated (1 of 10)  | 0.14          | 0.33                   | 0.46               | 0.47              |

{{< figure src="/images/attrition/ivr_stratified_power.png" alt="Power of the four tests against violation strength, concentrated versus diffuse" >}}

Caveats: individual-level randomization only, mean tests only, equal-sized strata.

## Simulation note 3: randomization inference

Setup: 100 schools of 20 learners in ten strata, half of the schools in each stratum treated. The test statistic is the strata-fixed-effects test over k baseline variables jointly (variables correlated at 0.3). Randomization inference re-assigns treatment across schools within strata, exactly as the original lottery did, recomputes the statistic 199 times, and takes the p-value from that distribution instead of the F table. Attrition is caused by treatment: 60 percent of learners respond in control schools and 70 percent in treated schools, so response rates differ by arm by construction — the case where re-randomization is not exactly justified in theory, because attrition is a post-treatment outcome (see the discussion in the post). Three hundred replications.

Under the null, attritors are otherwise identical to respondents, so a correct test rejects 5 percent of the time:

| Variables tested jointly | Standard test (IV-R / IV-P) | Randomization inference (IV-R / IV-P) |
|--------------------------|-----------------------------|---------------------------------------|
| 3                        | 0.08 / 0.09                 | 0.05 / 0.05                           |
| 10                       | 0.28 / 0.49                 | 0.07 / 0.06                           |

Under the violation, learners who respond only when treated score 0.5 standard deviations lower on one of the k variables. Rejection rates for randomization inference:

| Variables tested jointly | Respondent test | Representativeness test |
|--------------------------|-----------------|-------------------------|
| 3                        | 0.30            | 0.31                    |
| 10                       | 0.19            | 0.16                    |

The same violation is detected about half as often with ten variables as with three. The standard test rejects more often under the violation (0.72 at ten variables), but it also rejects 0.49 of the time when nothing is wrong.

## References

Anatolyev, Stanislav, and Mikkel Sølvsten. 2023. "Testing Many Restrictions Under Heteroskedasticity." *Journal of Econometrics* 236(1), 105473. [[journal]](https://ideas.repec.org/a/eee/econom/v236y2023i1s0304407623001677.html) [[ungated]](https://arxiv.org/abs/2003.07320)

Cilliers, Jacobus, Brahm Fleisch, Cas Prinsloo and Stephen Taylor. 2020. "How to Improve Teaching Practice? An Experimental Comparison of Centralized Training and In-Classroom Coaching." *Journal of Human Resources* 55(3): 926–962. [[link]](https://jhr.uwpress.org/content/55/3/926)

Ghanem, Dalia, Sarojini Hirshleifer and Karen Ortiz-Becerra. 2023. "Testing Attrition Bias in Field Experiments." *Journal of Human Resources*, published online August 11, 2023. [[journal]](https://jhr.uwpress.org/content/early/2023/08/02/jhr.0920-11190r2) [[ungated]](https://escholarship.org/uc/item/4ck087v3)

Ghanem, Dalia, Sarojini Hirshleifer and Karen Ortiz-Becerra. "attregtest: Stata module to implement regression-based attrition tests." Statistical Software Components, Boston College. Install with `ssc install attregtest`. [[link]](https://ideas.repec.org/c/boc/bocode/s459125.html)

Kerwin, Jason, Nada Rostom and Olivier Sterck. 2024. "Striking the Right Balance: Why Standard Balance Tests Over-Reject the Null, and How to Fix It." IZA Discussion Paper 17217. [[link]](https://www.iza.org/publications/dp/17217/striking-the-right-balance-why-standard-balance-tests-over-reject-the-null-and-how-to-fix-it)

Lee, David S. 2009. "Training, Wages, and Sample Selection: Estimating Sharp Bounds on Treatment Effects." *Review of Economic Studies* 76(3): 1071–1102. [[link]](https://academic.oup.com/restud/article-abstract/76/3/1071/1590712)

MacKinnon, James G., Morten Ørregaard Nielsen and Matthew D. Webb. 2023. "Cluster-Robust Inference: A Guide to Empirical Practice." *Journal of Econometrics* 232(2): 272–299. [[link]](https://www.sciencedirect.com/science/article/pii/S0304407622000781)
