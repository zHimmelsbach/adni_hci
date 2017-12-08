---
title: Data Exploration
notebook: EDA_Revised_Project_Statement.ipynb
nav_include: 2
---

## Contents
{:.no_toc}
*  
{: toc}

## Data Exploration

#### Process of Determining the Response Variable

Before settling on using the HCI as a response variable, we considered the Amyloid Convergence Index (a similar measurement developed by BAI that considers amyloid plaque build-up) as well as attempting to create a Tau Convergence Index (which would take the same approach as ACI or HCI, but instead look at tau protein tangles) to use as response variables for additional models. However, creating a TCI would have been much more demanding than our time allowed, and when we reviewed the dataset, we realized that the ACI data had not been computed for many patients in ADNI1 and lacked published documentation, so we decided to predict the HCI alone. 

We saw an opportunity to contribute to existing work on the HCI because, while the BAI team did consider the ADAS, MMSE, and CD assessments, they did not attempt to parse out which questions from the cognitive tests are most relevant for predicting HCI, nor did they consider the FAQ and NPI tests. Furthermore, our results may add robustness to those of the BAI team because they are constrained to a smaller sample of patients (~130) for whom the hippocampal volume was recorded.

#### Rationale for Regression Rather Than Classification

Once we settled on the HCI as an outcome, we considered the question of using regression to predict the HCI or using classification to predict whether or not a patient would be over the threshold HCI value that Chen et al identified as indicating a high risk of Mild Cognitive Impairment developing into Alzheimer's. One approach would have been to predict whether a patient's HCI fell in the 95% confidence interval that Chen et al identified for the threshold value. However, their confidence interval was very large, ranging from 2 to 20, and included 94% of the observations in one training set. We felt that categorizing almost every patient as likely to decline to Alzheimer's would not achieve our goal of reducing costs by better targeting PET scans. In addition, we thought that being able to predict the exact HCI value would be more useful to clinicians because higher vaues correspond with greater risks. Chen et al's work was the first predictive study using HCI, and other researchers may narrow the confidence interval for threshold HCI values in the future, at which time a classification model may become more useful.

#### Data Cleaning Process

The dataset we downloaded from LONI's Image Analysis sets included the HCI, date of exam, whether the exam was at 6, 12, 18, etc. months past a patient's baseline visit, and patient ID. We originally had 2865 HCI measurements from 1420 patients, but after extracting ADNI 1 participants for whom we had cognitive test score information, we had 605 records remaining for 220 patients (218 from baseline visits, 203 from 6-month visits, and 184 from 12-month visits). 

From the neuropsychological test datasets on LONI, we selected those which were included in ADNI 1. We excluded one exam that had a high level of missingness. After tabulating how many unique patients had scores and sub-task scores for each exam, we found that that the number of records dropped precipitously for exams administered more than 12 months after baseline, so we decided to constrain ourselves to baseline, 6 month, and 12 month records. For each of the seven exams we cleaned data and merged them together by patient ID, checking how many subjects had records for all tests. We then hot-encoded the categorical questions with multiple levels so we had only binary categorical predictor columns. 

## Data Visualization & Exploration


### HCI Exploration

We first examined the distribution of HCI over the progression of visits, and observed that the mean and variation of HCI scores increased over time. In the plot below, we see that the distributions broadened but the central peak did not shift noticeably. The shapes of the plots remained normally distributed, with perhaps a slight right skew.

<div style="text-align:center"><img src ="EDA_Revised_Project_Statement_files/EDA_Revised_Project_Statement_10_0.png" /></div>

We looked at correlations between HCI and the individual test sets. We started off by checking that the relationship between ADAS & HCI did not noticeably vary over time, which was true. In the plots below, it's clear that as time progressed HCI scores tended to increase, but the scatterplots of ADAS composite scores over HCI remained linear.

<p style="text-align: center;"> <strong> Relationship Between ADAS and HCI at Different Periods of the Study </strong> </p>
<div style="text-align:center"><img src ="EDA_Revised_Project_Statement_files/EDA_Revised_Project_Statement_13_0.png" /></div>


We then plotted the composite scores for the other cognitive tests over HCI scores (see plots below). We observed clear linear trends for the FAQ, MM, and CD tests, and a less definitive trend for the NPI score. The HM and GD tests did not seem to be obviously correlated. However, we believed that there still may be individual items from these tests which would add to the predictive power of our models, so we retained those questions in our modeling stage.

<p style="text-align: center;"> <strong> Correlations of Composite Test Scores and HCI </strong> </p>
<div style="text-align:center"><img src ="EDA_Revised_Project_Statement_files/EDA_Revised_Project_Statement_15_0.png" /></div>

### Exploration of Correlations Among Exam Items

We expected feature selection to be the central challenge in our model fitting, so we examined the degree to which items within each exam were correlated. While we noticed high correlations between some items, this was not true in general. We also observed that some of the items were more correlated with the composite exam scores than others, suggesting that a subset of these items may be more predictive of HCI than the overall scores. This encouraged us to continue with our research question.

<div style="text-align:center"><img src ="EDA_Revised_Project_Statement_files/EDA_Revised_Project_Statement_19_0.png" /></div>

<div style="text-align:center"><img src ="EDA_Revised_Project_Statement_files/EDA_Revised_Project_Statement_20_0.png" /></div>




