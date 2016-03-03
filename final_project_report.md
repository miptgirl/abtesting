# Experiment: Udacity Free Trial Screener
##Experiment Overview

At the time of this experiment, Udacity courses currently have two options on the home page: "start free trial", and "access course materials". If the student clicks "start free trial", they will be asked to enter their credit card information, and then they will be enrolled in a free trial for the paid version of the course. After 14 days, they will automatically be charged unless they cancel first. If the student clicks "access course materials", they will be able to view the videos and take the quizzes for free, but they will not receive coaching support or a verified certificate, and they will not submit their final project for feedback.

In the experiment, Udacity tested a change where if the student clicked "start free trial", they were asked how much time they had available to devote to the course. If the student indicated 5 or more hours per week, they would be taken through the checkout process as usual. If they indicated fewer than 5 hours per week, a message would appear indicating that Udacity courses usually require a greater time commitment for successful completion, and suggesting that the student might like to access the course materials for free. At this point, the student would have the option to continue enrolling in the free trial, or access the course materials for free instead. This screenshot shows what the experiment looks like.

![alt text][logo]

[logo]: https://github.com/miptgirl/abtesting/blob/master/data/experiment_screen.png "Experiment screenshot"

The hypothesis was that this might set clearer expectations for students upfront, thus reducing the number of frustrated students who left the free trial because they didn't have enough time—without significantly reducing the number of students to continue past the free trial and eventually complete the course. If this hypothesis held true, Udacity could improve the overall student experience and improve coaches' capacity to support students who are likely to complete the course.

The unit of diversion is a cookie, although if the student enrolls in the free trial, they are tracked by user-id from that point forward. The same user-id cannot enroll in the free trial twice. For users that do not enroll, their user-id is not tracked in the experiment, even if they were signed in when they visited the course overview page.

## Experiment Design
### Metric Choice
The following metrics will be used as invariant metrics for sanity check, they shouldn't be affected by experiment (to be sure that experiment is set up well and there is not pre-exisiting differences between control and experiment groups):
  * __Number of cookies__: number of unique cookies to view the course overview page. (`dmin=3000`)
  * __Number of clicks__: number of unique cookies to click the "Start free trial" button (which happens before the free trial screener is trigger). (`dmin=240`)
  *  __Click-through-probability__: number of unique cookies to click the "Start free trial" button divided by number of unique cookies to view the course overview page. (`dmin=0.01`)

The following metrics will be used as evaluation metrics:
  *  __Gross conversion__: number of user-ids to complete checkout and enroll in the free trial divided by number of unique cookies to click the "Start free trial" button. (`dmin= 0.01`)
  *  __Retention__: That is, number of user-ids to remain enrolled past the 14-day boundary (and thus make at least one payment) divided by number of user-ids to complete checkout. (`dmin=0.01`) !!this metric won't be used since it needs to much traffic to be significant!!
  *  __Net conversion__: That is, number of user-ids to remain enrolled past the 14-day boundary (and thus make at least one payment) divided by the number of unique cookies to click the "Start free trial" button. (`dmin= 0.0075`)

I am expecting in experiment group to have lower __gross conversion__ (some students will decide to have a free access to material instead of free trial because they won't be committed enough) and higher __retention__ and __net conversion__ (students have correct expectations on course commitment and it's more likely that they will finish the course).

###Measuring Standard Deviation
Let's estimate standard deviation for evaluation metrics given the sample size 5000 unique cookies visiting HomePage per day.
I will be using the following baseline values:
  * _Number of cookies (unique cookies to view page per day)_ = 5000
  * _Click-through-probability on "Start free trial"_ = 0.08
  * _Probability of enrolling, given click_ = 0.20625
  * _Probability of payment, given enroll_ = 0.53
  * _Probability of payment, given click_ = _Probability of enrolling, given click_ * _Probability of payment, given enroll_ = 0.1093125
  * _Unique cookies who click "Start free trial" per day_ = _# cookies_ * _Click-through-probability on "Start free trial"_ = 400
  * _Unique cookies who enroll_ = _Unique cookies who click "Start free trial" per day_ * _Probability of enrolling, given click_ = 82.5

All evaluation metrics (__gross conversion__, __retention__, __net conversion__) are probability metrics so they follow binomial distribution and SE (standard error) = sqrt(p*(1-p)/N).

  * __Gross conversion__: p is probability of enrolling, given click, N is unique cookies who click "Start free trial", SE = sqrt(0.20625*(1-0.20625)/400) = 0.0202
  * __Retention__: p is probability of payment, given enroll, N is unique cookies who enroll, SE = sqrt(0.53*(1-0.53)/82.5) = 0.0549
  * __Net conversion__: p is probability of payment, given click, N is unique cookies who click "Start free trial", SE = sqrt(0.1093125*(1-0.1093125)/400) = 0.0156

###Sizing
####Number of Samples vs. Power
Since we are using multiple metrics let's use Benferroni correction: alpha_common = alpha/3 = 0.05/3.
Let's calculate the needed  sample sizes in users using [online calculator](http://www.evanmiller.org/ab-testing/sample-size.html) for each individual metric (since online calculator allows to insert only round alpha values let's the closest one: alpha = 2%).
  * __Gross conversion__: 33014 unique cookies that clicked the "Start free trial" button for each group, _total number of unique cookies on the home page for 2 groups_ = 2 * 33014 / _Click-through-probability on "Start free trial"_ = 2*33014/0.08 = 825 350
  * __Retention__: 50013 enrolled user-ids for each group, _total number of unique cookies on the home page for 2 groups_ = 2 * 50013 / _Probability of enrolling, given click_ / _Click-through-probability on "Start free trial"_ = 2*50013/0.20625/0.08 = 6 062 182
  * __Net conversion__: 19747 unique cookies that clicked the "Start free trial" button for each group, _total number of unique cookies on the home page for 2 groups_ = 2 * 19747 / _Click-through-probability on "Start free trial"_ = 2*19747/0.08 = 493 675

####Duration vs. Exposure

We need to have enough power for all evaluation metrics, so we need to have 6062182 unique cookies on HomePage (the maximum value from previous task). Let's assume that we divert to experiment 100% of traffic (40 000 unique cookies a day), so we need 6062182/40000 = 152 days = 5 months. 

It will be too long for experiment, so we should revise our decision about evaluation metrics and consider only ____gross conversion__ and __net conversion__ then we will need conduct experiment 825350/40000 = 21 days (this looks much more plausible estimation).

I've decided to divert 100% of users to experiment, since it's not considered to be very risky and it allows us to have quicker results.

##Experiment Analysis
Experiment results are stored [here](https://docs.google.com/spreadsheets/d/1j62fHO68B7rxpgoVYsr0Iqy2CJuCM7LuTQ9QBxcQKlc/edit#gid=154400404). Let's consider only data from October, 11 to November, 2.

###Sanity Checks

First of all we need to perform sanity checks to be sure, that there is no problems in experiment set up.
Let's determine confidence interval for each invariant metric and observed value:
For number of cookies and cliks we have a binomial distribution with probability = 50% (probability of assigning user to control group) and SE = sqrt(p(1-p)/N) = sqrt(1/N)/2
  * __Number of cookies__: # cookies in control = 345543, # cookies in experiment = 344660, total = 690203, SE = sqrt(1/690203)/2 = 0.0006, m (margin of error) = Z*SE = 1.96*0.0006 = 0.0012, then confidence interval equals to (0.4988, 0.5012), observed value = 345543/690203 = 0.5006 is within confidence interval, so check is passed.
  * __Number of clicks__: # cookies in control = 28378, # cookies in experiment = 28325, total = 56703, SE = sqrt(1/56703)/2 = 0.0021, m (margin of error) = Z*SE = 1.96*0.0021 = 0.0041, then confidence interval equals to (0.4959, 0.5041), observed value = 28378/56703 = 0.5005 is within confidence interval, so check is passed.

One more check we have is on __CTP for "Start free trial button"__:
N_cont = 345543, X_cont = 28378

N_exp = 344660, X_exp = 28325

p_pool = (X_cont + X_exp)/(N_cont + N_exp) = 0.0822

SE_pool = sqrt(p_pool(1-p_poll)(1/N_exp + 1/N_cont)) = 0.0007

d = p_exp - p_cont = 0.00006

m = Z*SE_pool = 1.96*0.0007 = 0.0013

d < m -> there's no difference in CTP values in control and experiment, sanity check passed.

###Result Analysis
####Effect Size Tests
For each of your evaluation metrics, give a 95% confidence interval around the difference between the experiment and control groups. Indicate whether each metric is statistically and practically significant. (These should be the answers from the "Effect Size Tests" quiz.)

####Sign Tests
For each of your evaluation metrics, do a sign test using the day-by-day data, and report the p-value of the sign test and whether the result is statistically significant. (These should be the answers from the "Sign Tests" quiz.)

####Summary
State whether you used the Bonferroni correction, and explain why or why not. If there are any discrepancies between the effect size hypothesis tests and the sign tests, describe the discrepancy and why you think it arose.

###Recommendation
Make a recommendation and briefly describe your reasoning.

##Follow-Up Experiment
Give a high-level description of the follow up experiment you would run, what your hypothesis would be, what metrics you would want to measure, what your unit of diversion would be, and your reasoning for these choices.
