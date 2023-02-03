# Causal inference challenge project

### Machine Learning and Causal Inference. MSc in Data Science Methodology. Barcelona School of Economics.

Project idea contributed by Robin Evans (Oxford University).

## Description

In this project you should attempt to mimic part of the benchmarking
of causal inference methods reported by Dorie et al. (2019) for two
of those methods. Dorie et al. (2019) present the results of a
causal inference data analysis challenge, where the competing
methods attempt to provide a best estimate for the sample average
treatment effect of the treated (ATT).

The simulated data and scripts used to do the benchmarking are in
this GitHub [repo](https://github.com/vdorie/aciccomp), where
the authors provide 77 different simulation settings. To adapt the
project to the available time constraints of MLCI, we have simulated
the data for a particular setting (#58) as follows:

```
remotes::install_github("vdorie/aciccomp/2016")

library(aciccomp2016)

gold <- dgp_2016(input_2016, 58, 1)
gold <- do.call("data.frame", gold)
head(gold)
colnames(gold) <- c("X", "Y", "Y.0", "Y.1", "mu.0", "mu.1", "e")
write.csv(gold, "gold.csv", row.names=FALSE)

dat <- data.frame(Y=gold$Y, X=gold$X)
dat <- cbind(dat, input_2016)
colnames(dat) <- sub("x_", "Z_", colnames(dat))
write.csv(dat, "aciccomp2016_58_1.csv", row.names=FALSE)
```
Since the publication by Dorie et al. (2019) denote treatment
and covariate in the opposite way we do at MLCI, the code above
also renames them to comply with the terminology at the course.
Keep in mind that difference, however, when you read the paper.

The resulting CSV files are stored in this repo and correspond
to the following items:

* `gold.csv`: gold-standard data consisting of treatment `X`,
   observed outcome `Y`, potential outcomes Y(0) and Y(1) in
   `Y.0` and `Y.1`, expected value of the potential outcomes
   E[Y(0)] and E[Y(1)] in `mu.0` and `mu.1`, and propensity
   scores `e`. The data in this CSV file should be used only
   for benchmarking.

* `aciccomp2016_58_1.csv`: observed data consisting of
   outcome `Y`, treatment `X` and covariates `Z_1` to `Z_58`.
   The data in this CSV file should be used to estimate the
   ATE and ATT.
   
A straightforward estimate of the ATE can be obtained by using
a saturated linear regression model:

```
fit <- lm(Y ~ ., dat)
ate.hat <- coef(fit)["X"]
ate.hat
##       X
## 3.98629

## true ATE=E[Y(1)]-E[Y(0)]
ate <- mean(gold$mu.1) - mean(gold$mu.0)
ate
## [1] 3.422661
```
Had we had access to the true propensity score, the ATE estimate
would have been much closer to the actual ATE:

```
dat2 <- cbind(dat, e=gold$e)
fit2 <- lm(Y ~ X + e, dat2)
ate2.hat <- coef(fit2)["X"]
ate2.hat
##        X
## 3.406772
```
Therefore, making a good prediction of the propensity score
from the observed data can be a way to obtain a better ATE estimate.
A straightforward way to calculate the ATT with regression models
is the following one:

```
## Z_2 is a factor without observations for
## certain levels in the control group and
## we cannot use it for the estimation of
## of the ATT with regression models
dat2 <- dat[, !colnames(dat) %in% "Z_2"]

## build a model Y ~ Z from the controls
fitctl <- lm(Y ~ . - X, dat2, subset= X == 0)

## use the previous model to predict the
## counterfactual outcome in the treated
Y0.trt <- predict(fitctl, dat2[dat2$X == 1, ])

att.hat <- mean(dat2$Y[dat2$X == 1])-mean(Y0.trt)
att.hat
## [1] 4.096808

## true ATT
mean(gold[gold$X == 1, "mu.1"])-mean(gold[gold$X == 1, "mu.0"])
## [1] 3.344872
```

Try to use one of the best performing methods reported by
Dorie et al. (2019), such as [BART](https://arxiv.org/abs/0806.3286),
and see whether you achieve a better performance. If you have
time, include some other method and check whether the relative
performance you observe among these few methods matches the one
observed by Dorie et al. (2019).

## Submission procedure

This project has to be submitted using
[GitHub Classroom](https://classroom.github.com). This
means that you should have cloned the GitHub repo of this project
from the organization account for MLCI in the corresponding academic
year at https://github.com/mlciXXXX using the submission link
provided through Google Classroom.

Once you have cloned this GitHub repo, then you can work on it in
your local disk and _push_ your changes whenever you like, but make
sure that you have pushed the last version of your assignment before
the deadline; consult the Google Classroom if you are unsure about
the deadline. There is no _submit_ button or any other specific
submission procedure or action than just pushing your changes to your
GitHub assignment repo. When correcting the assignment, the version
available at the deadline will be retrieved. If the last update to
the repo is posterior to the deadline, then the mark of the
assignment will have a penalty.
