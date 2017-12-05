---
title: Neuropsychological Exam Data
notebook: Neuropsychological_Data_Summaries.ipynb
nav_include: 2
---

## Contents
{:.no_toc}
*  
{: toc}

## This file describes the data in each of the seven exams we use in our analyses

### ADAS (Alzheimer's Disease Assessment Scale)
- Primarily measures language and memory
- Scoring
    - Higher scores indicate greater dysfunction.
- Tasks:
    - Word recall
    - Naming objects and fingers
    - Following commands (i.e. simple instructions like "put the pencil in my hand")
    - Constructional Praxis (i.e. drawing increasingly complex shapes that are shown to subject)
    - Ideational Praxis (e.g. ask subject to *pretend* to write a letter, fold it, put it in envelope, and place stamp)
    - Orientation (Does subject know their name, the year, the month, where they are, etc.)
    - Word recognition (subject sees a list of words; when shown a word later, have to say whether it was on initial list)
    - Remembering test directions
    - Comprehension (assesed throughout testing; how well does subject understand words and language)
    - Word-finding difficulty (assessed through spontaneous conversation during assessment)

*source: https://www.verywell.com/alzheimers-disease-assessment-scale-98625 *

### CDR (Clinical Dementia Rating)
- Measures cognitive and functional performance
- Scoring
    - Higher scores indicate higher stages of dementia (i.e. more severe symptoms)
    - Final composite score between 0 and 3
- Six areas of testing:
    - Memory
    - Orientation
    - Judgment & problem solving
    - Community affairs
    - Home & hobbies
    - Personal care
- Notes:
    - Takes a long time to administer
    - High interrater reliability (Utility of the Clinical Dementia Rating in Asian Populations - Lim et al. 5 (1): 61 - Clinical Medicine & Research)

*source: https://en.wikipedia.org/wiki/Clinical_Dementia_Rating*

### FAQ (Functional Assessment Questionnaire)
- Measures ability to perform "activities of daily living"
- Scoring
    - Higher scores indicate greater dependence
- 10 questions
    - Writing checks, Doing taxes, shopping, etc.
- Notes:
    - Has been shown to be able to distinguish between mild cognitive impairment and "very mild alzheimer's disease" (https://www.ncbi.nlm.nih.gov/pmc/articles/PMC2997338/)
    
*source: ADNI data dictionary*

### GDS (Geriatric Depression Scale)
- 30 item self-report assessment (note: ADNI 1 only had 15 questions)
    - Each item takes a yes/no answer
- Scoring:
    - Higher scores indicate greater depression
- Example questions (from ADNI dictionary):
    - "Are you basically satisfied with your life?"
    - "Do you often get bored?"
    - "Do you feel full of energy?"
    
*source: https://en.wikipedia.org/wiki/Geriatric_Depression_Scale*

### Mini Mental-state Exam
- 30 item questionnaire (ADNI 1 has all 30)
- Measures factors similar to ADAS-cog
- Scoring:
    - Higher scores indicate better functioning
- Note:
    - Requires no training for administration
    - Affected by demographics
        - Age and education

*source: https://en.wikipedia.org/wiki/Mini%E2%80%93Mental_State_Examination*

### Modified Hachinski Scale
- Attempts to differentiate between alzheimers and dementia
- For each item, the physician indicates whether the stated condition is present or absent
- Scoring:
    - Score of 2 or less predicts alzheimers
    - Score greater than 2 predicts dementia
- Notes:
    - "Recent research has shown that the Hachinski score has no predictive value when using autopsy as the gold-standard diagnostic tool."
    - Some items are given 2 points when present
        - In the clean data, we've kept the total (weighted) score, but recoded all items to [0,1]
    
*source: http://www.gpnotebook.co.uk/simplepage.cfm?ID=-301596617*

### Neuropsychiatric Inventory Q
- Administered to caregivers of patients
- Scoring:
    - Higher scores indicate greater dysfunction
- Example questions:
    - "Does [patient] believe people are stealing from him/her?"
    - "Does [patient] appear to feel too good or act excessively happy?"
- Notes:
    - "...the NPI is able to screen for multiple types of dementia, not just Alzheimer’s Disease"
    - High interrater reliability
    
*source: http://www.apa.org/pi/about/publications/caregivers/practice-settings/assessment/tools/neuropsychiatric-inventory.aspx*
