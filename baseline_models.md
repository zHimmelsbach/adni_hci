---
title: Predictive Modeling
notebook: baseline_models.ipynb
nav_include: 3
---

## Contents
{:.no_toc}
*  
{: toc}



```python
import pandas as pd
import numpy as np
import matplotlib
import matplotlib.pyplot as plt
import sklearn.metrics as metrics
from sklearn.model_selection import cross_val_score, train_test_split
import seaborn as sns
import time
import re
import statsmodels.api as sm
from statsmodels.api import OLS
from sklearn.linear_model import LinearRegression

%matplotlib inline
```




```python
#Set styles
sns.set_style('white')
sns.set_context('talk')
```




```python
all_merged = pd.read_pickle("ADNIcsv/all_merged.pkl")
total_score_names = ['adas_total_0', 'adas_total_06', 'adas_total_12', 
                     'cdglobal_sc', 'cdglobal_06', 'cdglobal_12',  
                     'faqtotal_bl', 'faqtotal_06', 'faqtotal_12', 
                     'gdtotal_sc', 'gdtotal_12',  
                     'mmscore_sc', 'mmscore_06', 'mmscore_12', 
                     'hmscore',
                     'npiscore_bl', 'npiscore_06', 'npiscore_12']
hci_fields = ['hci_bl', 'hci_m06', 'hci_m12']
total_scores = all_merged.loc[:, total_score_names + hci_fields]

```




```python
X_cols = [x for x in total_score_names if re.search(r'0|bl|sc', x) != None and re.search(r'hci',x) == None]
temp = total_scores[X_cols + ['hci_bl']].dropna()
X = temp[X_cols]
y = temp['hci_bl']

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=.2)

baseline = LinearRegression()
print(cross_val_score(baseline, X_train, y_train, cv=5, n_jobs=-1))
baseline.fit(X_train, y_train)

print(metrics.r2_score(y_train, baseline.predict(X_train)))
print(metrics.r2_score(y_test, baseline.predict(X_test)))
```


    [ 0.40918137  0.1717127   0.1198882   0.49760678  0.2856557 ]
    0.488407988516
    0.429232383372
    



```python
adas_cols = [x for x in all_merged.columns if re.search(r'hci|cd|faq|gd|mm|RID|hm|npi|06|12|total', x) == None]
adas_temp = all_merged[adas_cols + ['hci_bl']].dropna()
X_adas = adas_temp[adas_cols]
y_adas = adas_temp['hci_bl']

X_train_adas, X_test_adas, y_train_adas, y_test_adas = train_test_split(X_adas, y_adas, test_size=.2)

baseline = LinearRegression()
print(cross_val_score(baseline, X_train_adas, y_train_adas, cv=5, n_jobs=-1))
baseline.fit(X_train_adas, y_train_adas)
print(metrics.r2_score(y_train_adas, baseline.predict(X_train_adas)))
print(metrics.r2_score(y_test_adas, baseline.predict(X_test_adas)))
```


    [ 0.23354186  0.17460298  0.72799779  0.13840559  0.27949847]
    0.527257354732
    0.2994252453
    



```python
def baseline_on_exam(model, exam_prefix):
    exam_cols = [x for x in all_merged.columns if re.search(r'hci|RID|06|12|total|global|score', x) == None and re.search(f'{exam_prefix}', x) != None]
    exam_temp = all_merged[exam_cols + ['hci_bl']].dropna()
    X_exam = exam_temp[exam_cols]
    y_exam = exam_temp['hci_bl']
    
    # Split into training and test
    X_train_exam, X_test_exam, y_train_exam, y_test_exam = train_test_split(X_exam, y_exam, test_size=.2)
    
    model = model()
    print(cross_val_score(model, X_train_exam, y_train_exam, cv=5, n_jobs=-1))
    print('Number of items: ', X_train_exam.shape[1])
    model.fit(X_train_exam, y_train_exam)
    train_score = metrics.r2_score(y_train_exam, model.predict(X_train_exam))
    test_score = metrics.r2_score(y_test_exam, model.predict(X_test_exam))
    return train_score, test_score
    
```




```python
for i in range(10):
    print(baseline_on_exam(LinearRegression, 'npi'))
    print('\n')
```


    [-0.03729281 -0.01943956  0.33033794 -0.0287696  -0.17177546]
    Number of items:  13
    (0.21009420475508045, 0.079031324135709125)
    
    
    [ 0.01973059 -0.46552873  0.14002071 -0.04382561 -0.01463751]
    Number of items:  13
    (0.16492048471070864, 0.16565190703653376)
    
    
    [-0.19703708  0.0903938   0.00208488 -0.08109694  0.16381247]
    Number of items:  13
    (0.16075682678578285, 0.15936683966012133)
    
    
    [ 0.15832311 -0.01943761  0.16012343 -0.47504338  0.20548312]
    Number of items:  13
    (0.22665088032836089, 0.013223451147024501)
    
    
    [-0.03704874 -0.18364146 -0.00984692 -0.15251972 -0.09160641]
    Number of items:  13
    (0.17528472710256526, 0.17065651050226616)
    
    
    [-0.66386138 -0.01177791  0.08235142 -0.1833735   0.09841256]
    Number of items:  13
    (0.18799864474222283, 0.014947148188055426)
    
    
    [-0.08763533 -0.32778175  0.06726909  0.0649801   0.21553317]
    Number of items:  13
    (0.19277384784097262, 0.042878743845860323)
    
    
    [ 0.25518355  0.04532645  0.07567336  0.06626183 -0.42231124]
    Number of items:  13
    (0.21894168426986815, -0.079797576454879726)
    
    
    [-0.08601555 -0.22581929  0.16932434  0.06738606 -0.11075164]
    Number of items:  13
    (0.18713366258685682, 0.15054852613515812)
    
    
    [-0.10977865 -0.12845382  0.08461125 -0.02142047 -0.07976568]
    Number of items:  13
    (0.17258475762488834, 0.16490896305574976)
    
    
    



```python
adas_cols
```





    ['word_recall_0',
     'construction_0',
     'delayed_word_recall_0',
     'naming_0',
     'ideational_praxis_0',
     'orientation_0',
     'word_recognition_0',
     'recall_instructions_0',
     'spoken_language_0',
     'word_finding_0',
     'comprehension_0',
     'number_cancellation_0',
     'adas_total_0']



1. Build model with significant items from each exam
2. Stepforward selection
3. PCA
4. Random forest with limited depth (CV for depth and for p)
5. AdaBoost

Notebook
1. Cleaning
2. EDA
3. Models

Things to do
1. Check that all the items are binary
2. Figure out how to make the website
3. PCA
4. Random Forest
5. Make nice looking version of cleaning notebook 
6. Make nice version of baseline models

Send parts by Monday night. Meet Wednesday at 5:15



```python

```

