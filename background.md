---
title: Background
notebook: background.ipynb
nav_include: 1
---

## Contents
{:.no_toc}
*  
{: toc}


### How is Alzheimer's Disease diagnosed?

Alzheimer's Disease is diagnosed post-mortem and can be difficult to distinguish from other types of dementia. 

While both dementia and Alzheimer's are characterized by problems with memory, communication, and thinking ability, patients with Alzheimer's may also show increased apathy, impaired judgement, disorientation, behavior change, and difficulty with basic functioning.

In addition, several biomarkers indicate Alzheimer's. These include hypometabolism (regions of low glucose activity, analogous to "dead" brain regions), brain shrinkage, amyloid protein plaque deposits, and tau protein tangles.

### What does a PET scan measure?

PET scans measure the concentrations of radioactive tracer molecules in the body. 

In FDG-PET scans (the type used to measure hypometabolism), a radioactive tracer is attached to glucose molecules (a key source of cellular energy) and injected into the patient. The glucose molecules aggregate in the regions of highest metabolic activity; thus, low levels of radioactivity indicate low levels of glucose, which corresponds to hypometabolism in those regions of the brain.

In the case of Alzheimer's, different tracer molecules can be used to measure amyloid plaques and tau protein tangles in the brain. Clinicians also use PET scans to diagnose other conditions in different organs.

### What is HCI?


"HCI" stands for "Hypometabolic Convergence Index", a measure developed by the Banner Alzheimer's Institute that indicates the degree to which a patient's FDG-PET brain scans correspond to those of a patient with an Alzheimer's diagnosis. 

In a 2011 paper, a team at BAI found that HCI predicted whether or not a patient with Mild Cognitive Impairment would develop Alzheimer's better than other biomarkers (hippocampal volume, amyloid and tau proteins in cerebral spinal fluid), memory test scores, and clinical assessments (Chen et al, 55-57).

After normalization, HCI values range between 0 and about 30. Chen et al found that an 8.36 threshold value for the HCI (determined using an ROC curve) resulted in a 6.55 hazard ratio under the Cox Proportional Hazard Model when controlling for other variables. This "hazard ratio" indicates that for every unit increase in HCI beyond the threshold, the patient's chance of developing Alzheimer's is 6.55 times as likely (Chen et al, 57).


### Which cognitive tests were used?  What did they ask?

We developed our models using patient's results on seven cognitive exams:

**ADAS (Alzheimer's Disease Assessment Scale)**:  ADAS primarily measures language and memory. Higher scores indicate greater dysfunction. The tasks include word recall, naming objects & fingers, remembering test directions, comprehension, word-finding difficulty, following commands, orientation (e.g. knowing the date), constructional praxis (e.g. drawing increasingly complex shapes that are shown to subject), and ideational praxis (e.g. asking the subject to *pretend* to write a letter, fold it, put it in envelope, and place the stamp).

**CDR (Clinical Dementia Rating)**: CDR measures cognitive and functional performance. Final composite CDR scores range from 0 to 3; higher scores indicate higher stages of dementia (i.e. more severe symptoms). CDR covers six areas: memory, orientation, judgement & problem-solving, community affairs, home & hobbies, and personal care. Administering the CDR takes a significant amount of time, but the test has high inter-rater reliability. 

**FAQ (Functional Assessment Questionnaire)**: FAQ measures a patient's ability to perform the "activities of daily living." Higher scores indicate greater dependence. The ten questions on FAQ cover tasks such as writing checks, doing taxes, and shopping. FAQ has been shown to be capable of distinguishing between mild cognitive impairment and "very mild alzheimer's disease." 


**GDS (Geriatric Depression Scale)**: At the time of the ADNI-1 trial, GDS was a self-report assessment of 15 yes-or-no questions (15 additional items were added in later phases). Higher scores indicate greater depression. Examples of questions include, "Are you basically satisfied with your life?", "Do you often get bored?", and "Do you feel full of energy?"

**MMSE (Mini Mental-state Exam)**: The MMSE is a 30 item questionnaire that measures factors similar to ADAS. Higher scores indicate better functioning. Unlike ADAS, the MMSE does not require a trained professional to administer. However, the test is observed to be affected by age and education demographics. 

**MHS (Modified Hachinski Scale)**: The MHS test attempts to differentiate between Alzheimer's and dementia. It must be administered by a physician. A score of 2 or less predicts Alzheimer's; a score of greater than 2 predicts dementia. However, according to the GP Notebook (see link below under "Sources"), "recent research has shown that the Hachinski score has no predictive value when using autopsy as the gold-standard diagnostic tool."

** NPI (Neuropsychiatric Inventory Questionnaire)**: The NPI is administered to caregivers rather than the patients themselves. It has high interrater reliability and, according to the American Psychological Association, is able to screen for multiple types of dementia. Higher scores indicate greater dysfunction. Example questions include "Does [the patient] believe people are stealing from him/her?" and "Does [the patient] appear to feel too good or act excessively happy?"


### How well do cognitive tests predict Alzheimer's on their own?

Chen et al found that ADAS and CDR were not able to significantly predict whether or not a patient with MCI would decline to probable Alzheimer's within 18 months (Chen et al 57). Another study followed patients with mild cognitive impairment over a five year period (Palmqvist et al, 2012). It explored predicting the development of Alzheimer's using the results of cognitive tests administered at baseline, including the MMSE. The authors found that the MMSE combined with a clock drawing test predicted the development of Alzheimer's as well as cerebrospinal fluid biomarkers.

Chapman et al (2012) used discriminant analysis on principal components from a set of cognitive test items and found that the components were statistically significant predictors of subjects with mild cognitive impairment converting to Alzheimer's.

We found no examples in the literature of attempting to predict HCI from cognitive test items.

### Where did the data come from?

We used data from the first phase of the ADNI (Alzheimer's Disease Neuroimaging Initiative) study, a North America-wide study that tracked patient progression over several years from early & late Mild Cognitive Impairment to dementia or Alzheimer's. The data is available online (see http://adni.loni.usc.edu/). 

The ADNI1 datasets originally included 2865 HCI measurements from 1420 patients. The records included measurements at a "baseline" visit as well as 6 & 12 months later. After extracting the participants for whom we had cognitive test score information, we had 605 records remaining for 220 patients (218 from baseline visits, 203 from 6-month visits, and 184 from 12-month visits). 

### References 

"About PET Scans." (n.d.). Retrieved December 5, 2017, from https://www.acrin.org/patients/aboutimagingexamsandagents/aboutpetscans.aspx

"ADNI Study Design." (n.d.). Retrieved December 5, 2017, from http://adni.loni.usc.edu/study-design/

"Alzheimer’s & Dementia Testing Advances". Alzheimer's Research Center. (n.d.). Retrieved December 5, 2017, from //www.alz.org/research/science/earlier_alzheimers_diagnosis.asp

Chapman, R., Mapstone, M., Mccrary, J., Gardner, M., Porsteinsson, A., Sandoval, T., . . . Reilly, L. (2011). Predicting conversion from mild cognitive impairment to Alzheimer's disease using neuropsychological tests and multivariate methods. Journal of Clinical and Experimental Neuropsychology, 33(2), 187-199.

Chaves, C., & MD. (n.d.). The Alzheimer’s Disease Assessment Scale-Cognitive Subscale (ADAS-Cog). Retrieved December 5, 2017, from https://www.verywell.com/alzheimers-disease-assessment-scale-98625
"Geriatric Depression Scale." (n.d.). Retrieved December 5, 2017, from http://www.minddisorders.com/Flu-Inv/Geriatric-Depression-Scale.html

K. Chen, N. Ayutyanont, J. B. Langbaum, A. S. Fleisher, C. Reschke, W. Lee, X. Liu, D. Bandy, G. E. Alexander, P. M. Thompson, L. Shaw, J. Q. Trojanowski, C. R. Jack, Jr., S. M. Landau, N. L. Foster, D. J. Harvey, M. W. Weiner, R. A. Koeppe, W. J. Jagust, and E. M. Reiman, "Characterizing Alzheimer's disease using a hypometabolic convergence index," *Neuroimage.*, vol. 56, no. 1, pp. 52-60, May 2011.

Lim, W. S., Chong, M. S., & Sahadevan, S. (2007). Utility of the Clinical Dementia Rating in Asian Populations. Clinical Medicine and Research, 5(1), 61–70. http://doi.org/10.3121/cmr.2007.693

"Modified Hachinski Ischaemic Scale." General Practice Notebook. (n.d.). Retrieved December 5, 2017, from http://www.gpnotebook.co.uk/simplepage.cfm?ID=-301596617

"Neuropsychiatric Inventory." (n.d.). Retrieved December 5, 2017, from http://www.apa.org/pi/about/publications/caregivers/practice-settings/assessment/tools/neuropsychiatric-inventory.aspx

Palmqvist, Hertze, Minthon, Wattmo, Blennow, Zetterberg, . . . Hansson. (2012). Comparison of brief cognitive tests and CSF analysis in predicting Alzheimer's disease in mild cognitive impairment: Six-year follow-up study. Alzheimer's & Dementia: The Journal of the Alzheimer's Association, 8(4), P434

Teng, E., Becker, B. W., Woo, E., Knopman, D. S., Cummings, J. L., & Lu, P. H. (2010). Utility of the Functional Activities Questionnaire for Distinguishing Mild Cognitive Impairment from Very Mild Alzheimer’s Disease. Alzheimer Disease and Associated Disorders, 24(4), 348–353. http://doi.org/10.1097/WAD.0b013e3181e2fc84

Tombaugh, T. N. and McIntyre, N. J. (1992), The Mini-Mental State Examination: A Comprehensive Review. Journal of the American Geriatrics Society, 40: 922–935. doi:10.1111/j.1532-5415.1992.tb01992.x

"What’s the Difference Between Dementia and Alzheimer’s Disease?" (2013, August 23). Retrieved December 5, 2017, from https://www.healthline.com/health/alzheimers-disease/difference-dementia-alzheimers
