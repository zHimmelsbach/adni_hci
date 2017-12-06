---
title: Modeling
notebook: models.ipynb
nav_include: 3
---

## Contents
{:.no_toc}
*  
{: toc}


```python
all_merged = pd.read_pickle("ADNIcsv/all_merged.pkl")
```




```python
def extract_fields(columns, key, is_at_start):
    match_list = []
    if is_at_start == True:
        length = len(key)
        for col in columns:
            if key in col[:length]:
                if "score" not in col:
                    match_list.append(col)
    else:
        for col in columns:
            if key in col:
                if "score" not in col:
                    match_list.append(col)
    return match_list
```




```python
total_score_names = ['adas_total_0', 'adas_total_06', 'adas_total_12', 
                     'cdglobal_sc', 'cdglobal_06', 'cdglobal_12',  
                     'faqtotal_bl', 'faqtotal_06', 'faqtotal_12', 
                     'gdtotal_sc', 'gdtotal_12',  
                     'mmscore_sc', 'mmscore_06', 'mmscore_12', 
                     'hmscore',
                     'npiscore_bl', 'npiscore_06', 'npiscore_12']
```




```python
def make_XY_sets(dataset, quad = False):
    #Create feature & reponse variable datasets split by VISCODE, with total scores removed
    all_X = dataset.drop(extract_fields(all_merged.columns, 'hci', True), axis = 1).drop(total_score_names, axis = 1).dropna()
    X_06, X_12 = all_X[extract_fields(all_X.columns, '06', False)], all_X[extract_fields(all_X.columns, '12', False)]
    X_bl = all_X.drop(X_06.columns, axis = 1).drop(X_12.columns, axis = 1)

    #Remove patients who dropped out between baseline and 12 month visits
    X_bl, X_06 = X_bl.loc[X_12.index][:],  X_06.loc[X_12.index][:]

    #Find response variables for each viscode set, dropping missing HCI values
    y_bl, y_06, y_12 = all_merged.loc[X_bl.index][:].hci_bl.dropna(), all_merged.loc[X_06.index][:].hci_m06.dropna(), all_merged.loc[X_12.index][:].hci_m12.dropna() 

    #Update predictor datasets to only include samples with HCI records
    X_bl, X_06, X_12 = X_bl.loc[y_bl.index][:], X_06.loc[y_06.index][:], X_12.loc[y_12.index][:]

    return [X_bl, X_06, X_12, y_bl, y_06, y_12]
```




```python
#Split to train and test sets
train, test = train_test_split(all_merged, test_size = 0.2)
X_bl_test, X_06_test, X_12_test, y_bl_test, y_06_test, y_12_test = make_XY_sets(test)
X_bl, X_06, X_12, y_bl, y_06, y_12 = make_XY_sets(train)
```


## Attempts to Improve the Baseline Model

### Approach 1: "Most Significant Predictors"

We fit Linear Regression & Random Forest Regressor models to the individual questions on each cognitive test. We fit each test separately and found the most significant predictors on each fit. For the Linear Regression fits, we considered predictors with a p-value of less than 0.03 to be significant (we used a stringent p-value because using less-stringent p-values such as 0.05 resulted in overfitting from too many predictors being selected). For the Random Forest fits, we simply selected the predictor that was considered most-significant in each fit. We ran 100 iterations of this process on different subsets of the training data, and then selected out the predictors that were considered significant in at least 50% of the trials.



```python
#Split up predictors according to test (ADAS, CD, FAQ, GD, HM, MM, or NPI)
bl_cols, m06_cols, m12_cols = [], [], []

#Add in ADAS columns
adas_cols_bl = [x for x in all_merged.columns if re.search(r'hci|cd|faq|gd|mm|RID|hm|npi|06|12|total', x) == None]
bl_cols.append(adas_cols_bl)
adas_cols_06_temp = [x for x in all_merged.columns if re.search(r'hci|cd|faq|gd|mm|RID|hm|npi|12|total', x) == None]
m06_cols.append([x for x in adas_cols_06_temp if x not in adas_cols_bl])
adas_cols_12_temp = [x for x in all_merged.columns if re.search(r'hci|cd|faq|gd|mm|RID|hm|npi|06|total', x) == None]
m12_cols.append([x for x in adas_cols_12_temp if x not in adas_cols_bl])

#Other tests
for key in ['cd', 'faq', 'gd', 'hm','mm','npi']:
    bl_cols.append(extract_fields(X_bl.columns, key, True))
    if key != 'hm':
        m12_cols.append(extract_fields(X_12.columns, key, True))
        
        if key != 'gd':
            m06_cols.append(extract_fields(X_06.columns, key, True))
        else:
            m06_cols.append([])
    else:
        m12_cols.append([])
        m06_cols.append([])
```




```python
#Storage for sets of significant predictors 
all_signifs_lin, all_signifs_rf = [], []

#Initialize RF model
rf = RandomForestRegressor()

#Set number of iterations for sub-set model fitting
iters = 100

for j in range(iters):
    #Make new subset of training set for each model fit
    sub_train, sub_test = train_test_split(train, test_size = 0.2)
    X_bl, X_06, X_12, y_bl, y_06, y_12 = make_XY_sets(sub_train)
    
    #Re-structure storage for viscode data subsets
    vis_cols = [bl_cols, m06_cols, m12_cols]
    X_sets, y_sets = [X_bl, X_06, X_12], [y_bl, y_06, y_12]

    signif_preds_lin, signif_preds_rf = [], []
    
    #For each viscode:
    for i in range(3):
        
        #Get the X & y datasets
        vis, X, y = vis_cols[i], X_sets[i], y_sets[i]
        vis_preds_lin, vis_preds_rf = [], []
        
        #For each list of predictors, fit model and record the significant predictors
        for col_list in vis:
            X2 = X.loc[:][col_list]

            if len(col_list) != 0:      
                #Find significant predictors in linear fit
                est = sm.OLS(y, sm.add_constant(X2))
                est2 = est.fit()
                signifs = est2.params[est2.pvalues < 0.03].index.values
                for i in signifs:
                    if i != 'const':
                        vis_preds_lin.append(i)
                
                #Find the most significant predictor from rf fit
                fit = rf.fit(X2, y)
                signifs_rf = fit.feature_importances_.reshape(-1,1)
                debug_ser = pd.DataFrame(signifs_rf, index = col_list, columns = ["imp"])
                most_imp = debug_ser.sort_values('imp', ascending = False).index[0]
                vis_preds_rf.append(most_imp)
            
        #Add each significant predictor to a list
        signif_preds_lin.append(vis_preds_lin)
        signif_preds_rf.append(vis_preds_rf)
        
    #Add list of significant predictors for each viscode to the larger storage list
    all_signifs_lin.append(signif_preds_lin)
    all_signifs_rf.append(signif_preds_rf)
    
#Restructure data storage structures
all_signifs_lin_df = pd.DataFrame(all_signifs_lin, columns = ['BL', 'm06', 'm12'])
all_signifs_rf_df = pd.DataFrame(all_signifs_rf, columns = ['BL', 'm06', 'm12'])
```


    /Applications/anaconda/lib/python3.6/site-packages/statsmodels/base/model.py:1036: RuntimeWarning: invalid value encountered in true_divide
      return self.params / self.bse
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in greater
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in less
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:1818: RuntimeWarning: invalid value encountered in less_equal
      cond2 = cond0 & (x <= self.a)
    /Applications/anaconda/lib/python3.6/site-packages/statsmodels/base/model.py:1036: RuntimeWarning: invalid value encountered in true_divide
      return self.params / self.bse
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in greater
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in less
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:1818: RuntimeWarning: invalid value encountered in less_equal
      cond2 = cond0 & (x <= self.a)
    /Applications/anaconda/lib/python3.6/site-packages/statsmodels/base/model.py:1036: RuntimeWarning: invalid value encountered in true_divide
      return self.params / self.bse
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in greater
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in less
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:1818: RuntimeWarning: invalid value encountered in less_equal
      cond2 = cond0 & (x <= self.a)
    /Applications/anaconda/lib/python3.6/site-packages/statsmodels/base/model.py:1036: RuntimeWarning: invalid value encountered in true_divide
      return self.params / self.bse
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in greater
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in less
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:1818: RuntimeWarning: invalid value encountered in less_equal
      cond2 = cond0 & (x <= self.a)
    /Applications/anaconda/lib/python3.6/site-packages/statsmodels/base/model.py:1036: RuntimeWarning: invalid value encountered in true_divide
      return self.params / self.bse
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in greater
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in less
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:1818: RuntimeWarning: invalid value encountered in less_equal
      cond2 = cond0 & (x <= self.a)
    /Applications/anaconda/lib/python3.6/site-packages/statsmodels/base/model.py:1036: RuntimeWarning: invalid value encountered in true_divide
      return self.params / self.bse
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in greater
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in less
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:1818: RuntimeWarning: invalid value encountered in less_equal
      cond2 = cond0 & (x <= self.a)
    /Applications/anaconda/lib/python3.6/site-packages/statsmodels/base/model.py:1036: RuntimeWarning: invalid value encountered in true_divide
      return self.params / self.bse
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in greater
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in less
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:1818: RuntimeWarning: invalid value encountered in less_equal
      cond2 = cond0 & (x <= self.a)
    /Applications/anaconda/lib/python3.6/site-packages/statsmodels/base/model.py:1036: RuntimeWarning: invalid value encountered in true_divide
      return self.params / self.bse
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in greater
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in less
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:1818: RuntimeWarning: invalid value encountered in less_equal
      cond2 = cond0 & (x <= self.a)
    /Applications/anaconda/lib/python3.6/site-packages/statsmodels/base/model.py:1036: RuntimeWarning: invalid value encountered in true_divide
      return self.params / self.bse
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in greater
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in less
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:1818: RuntimeWarning: invalid value encountered in less_equal
      cond2 = cond0 & (x <= self.a)
    /Applications/anaconda/lib/python3.6/site-packages/statsmodels/base/model.py:1036: RuntimeWarning: invalid value encountered in true_divide
      return self.params / self.bse
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in greater
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in less
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:1818: RuntimeWarning: invalid value encountered in less_equal
      cond2 = cond0 & (x <= self.a)
    /Applications/anaconda/lib/python3.6/site-packages/statsmodels/base/model.py:1036: RuntimeWarning: invalid value encountered in true_divide
      return self.params / self.bse
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in greater
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in less
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:1818: RuntimeWarning: invalid value encountered in less_equal
      cond2 = cond0 & (x <= self.a)
    /Applications/anaconda/lib/python3.6/site-packages/statsmodels/base/model.py:1036: RuntimeWarning: invalid value encountered in true_divide
      return self.params / self.bse
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in greater
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in less
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:1818: RuntimeWarning: invalid value encountered in less_equal
      cond2 = cond0 & (x <= self.a)
    /Applications/anaconda/lib/python3.6/site-packages/statsmodels/base/model.py:1036: RuntimeWarning: invalid value encountered in true_divide
      return self.params / self.bse
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in greater
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in less
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:1818: RuntimeWarning: invalid value encountered in less_equal
      cond2 = cond0 & (x <= self.a)
    /Applications/anaconda/lib/python3.6/site-packages/statsmodels/base/model.py:1036: RuntimeWarning: invalid value encountered in true_divide
      return self.params / self.bse
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in greater
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in less
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:1818: RuntimeWarning: invalid value encountered in less_equal
      cond2 = cond0 & (x <= self.a)
    /Applications/anaconda/lib/python3.6/site-packages/statsmodels/base/model.py:1036: RuntimeWarning: invalid value encountered in true_divide
      return self.params / self.bse
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in greater
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in less
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:1818: RuntimeWarning: invalid value encountered in less_equal
      cond2 = cond0 & (x <= self.a)
    /Applications/anaconda/lib/python3.6/site-packages/statsmodels/base/model.py:1036: RuntimeWarning: invalid value encountered in true_divide
      return self.params / self.bse
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in greater
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in less
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:1818: RuntimeWarning: invalid value encountered in less_equal
      cond2 = cond0 & (x <= self.a)
    /Applications/anaconda/lib/python3.6/site-packages/statsmodels/base/model.py:1036: RuntimeWarning: invalid value encountered in true_divide
      return self.params / self.bse
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in greater
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in less
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:1818: RuntimeWarning: invalid value encountered in less_equal
      cond2 = cond0 & (x <= self.a)
    /Applications/anaconda/lib/python3.6/site-packages/statsmodels/base/model.py:1036: RuntimeWarning: invalid value encountered in true_divide
      return self.params / self.bse
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in greater
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in less
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:1818: RuntimeWarning: invalid value encountered in less_equal
      cond2 = cond0 & (x <= self.a)
    /Applications/anaconda/lib/python3.6/site-packages/statsmodels/base/model.py:1036: RuntimeWarning: invalid value encountered in true_divide
      return self.params / self.bse
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in greater
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in less
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:1818: RuntimeWarning: invalid value encountered in less_equal
      cond2 = cond0 & (x <= self.a)
    /Applications/anaconda/lib/python3.6/site-packages/statsmodels/base/model.py:1036: RuntimeWarning: invalid value encountered in true_divide
      return self.params / self.bse
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in greater
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in less
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:1818: RuntimeWarning: invalid value encountered in less_equal
      cond2 = cond0 & (x <= self.a)
    /Applications/anaconda/lib/python3.6/site-packages/statsmodels/base/model.py:1036: RuntimeWarning: invalid value encountered in true_divide
      return self.params / self.bse
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in greater
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:879: RuntimeWarning: invalid value encountered in less
      return (self.a < x) & (x < self.b)
    /Applications/anaconda/lib/python3.6/site-packages/scipy/stats/_distn_infrastructure.py:1818: RuntimeWarning: invalid value encountered in less_equal
      cond2 = cond0 & (x <= self.a)
    



```python
#Extract higly recurring (in >50% of iterations) significant predictors for each model

#Storage structures for most-recurring predictors
vis_sigs_lin, vis_sigs_rf = [], []

#For each viscode:
for vis in ['BL', 'm06', 'm12']:
    shared_lin, shared_rf = [], []

    #Add every significant predictor from each round to a single list
    for pred_list in all_signifs_lin_df.loc[:][vis]:
        for pred in pred_list:
            shared_lin.append(pred)
    for pred_list in all_signifs_rf_df.loc[:][vis]:
        for pred in pred_list:
            shared_rf.append(pred)
    #Count the number of times each predictor was selected
    shared2_lin, shared2_rf = pd.Series(shared_lin), pd.Series(shared_rf)
    uniq_preds_lin, uniq_preds_rf = shared2_lin.unique(), shared2_rf.unique()
    
    #Extract the predictors that were significant in at least half of the model fit iterations
    sigs_lin = shared2_lin.value_counts().index[shared2_lin.value_counts() > iters/2]
    sigs_rf = shared2_rf.value_counts().index[shared2_rf.value_counts() > iters/2]
    
    #Add to storage structure
    vis_sigs_lin.append(sigs_lin.tolist())
    vis_sigs_rf.append(sigs_rf.tolist())
```




```python
#Create new datasets using only the most signficant predictors

#Linear Fit Significant Dataset
X_bl_sig_lin, X_06_sig_lin, X_12_sig_lin = X_bl.loc[:][vis_sigs_lin[0]], X_06.loc[:][vis_sigs_lin[1]], X_12.loc[:][vis_sigs_lin[2]]
X_sigs_lin = [X_bl_sig_lin, X_06_sig_lin, X_12_sig_lin]

#Random Forest Fit Significant Dataset
X_bl_rf, X_06_rf, X_12_rf = X_bl.loc[:][vis_sigs_rf[0]], X_06.loc[:][vis_sigs_rf[1]], X_12.loc[:][vis_sigs_rf[2]]
X_rfs = [X_bl_rf, X_06_rf, X_12_rf]

#Associated Test Sets
X_sigs_lin_test = [X_bl_test.loc[:][vis_sigs_lin[0]], X_06_test.loc[:][vis_sigs_lin[1]], X_12_test.loc[:][vis_sigs_lin[2]]]
X_rf_test = [X_bl_test.loc[:][vis_sigs_rf[0]], X_06_test.loc[:][vis_sigs_rf[1]], X_12_test.loc[:][vis_sigs_rf[2]]]
y_sets_test = [y_bl_test, y_06_test, y_12_test]
```




```python
#Fit selected Linear predictors on full train and full test sets, calculate R2 scores 
regr = LinearRegression()

#Initialize lists to store scores
train_scores = []
test_scores = []

#Calculate and append scores to lists
for i in range(3):
    regr.fit(X_sigs_lin[i], y_sets[i])
    train_scores.append(regr.score(X_sigs_lin[i], y_sets[i]))
    test_scores.append(regr.score(X_sigs_lin_test[i], y_sets_test[i]))

#Restructure storage of R2 scores
lin_signif_records = pd.DataFrame([train_scores, test_scores], index = ['Train', 'Test'], columns = ['BL', '6M', '12M']).T
```




```python
#Fit selected RF predictors on full train and full test sets, calculate R2 scores 

#Initialize lists to store scores
rf_train_scores = []
rf_test_scores = []

#Calculate and append scores to lists
for i in range(3):
    fit = rf.fit(X_rfs[i], y_sets[i])
    rf_train_scores.append(fit.score(X_rfs[i], y_sets[i]))
    rf_test_scores.append(fit.score(X_rf_test[i], y_sets_test[i]))

#Restructure storage of R2 scores
rf_signif_records = pd.DataFrame([rf_train_scores, rf_test_scores], index = ['Train', 'Test'], columns = ['BL', '6M', '12M']).T
```




```python
#Print results of linear fit
lin_signif_records
```





<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Train</th>
      <th>Test</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>BL</th>
      <td>0.555438</td>
      <td>-0.001134</td>
    </tr>
    <tr>
      <th>6M</th>
      <td>0.450924</td>
      <td>0.522852</td>
    </tr>
    <tr>
      <th>12M</th>
      <td>0.631418</td>
      <td>0.378470</td>
    </tr>
  </tbody>
</table>
</div>





```python
#Print results of RF fit
rf_signif_records
```





<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Train</th>
      <th>Test</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>BL</th>
      <td>0.668476</td>
      <td>0.194630</td>
    </tr>
    <tr>
      <th>6M</th>
      <td>0.718225</td>
      <td>0.367173</td>
    </tr>
    <tr>
      <th>12M</th>
      <td>0.819802</td>
      <td>0.358399</td>
    </tr>
  </tbody>
</table>
</div>



### Approach 2: Step-Forward Variable Selection

Here we used a step-forward variable selection approach. For the Linear Regression models, we used the BIC as the criterion for whether or not adding a feature improved the model; for Random Forest, we used the R2 score as the criterion. 

#### Approach 2A: Step Forward Selection for Linear Regression



```python
#STEP-WISE FORWARD SELECTION WITH OLS
selected_preds_lin = []

for k in range(3):
    #Create a list of all predictor variables
    all_preds = X_sets[k].columns.tolist()

    #Start with an empty set of model predictors
    predictors_fw = []

    #Initialize the list holding the best BIC scores
    best_score = [10e6,0]

    #Initialize the boolean tracking whether or not better solutions are still being found
    atEnd = False

    #Loop over 
    while atEnd == False:
        #track whether or not better solutions are still being found
        tracker = 0

        for i in range(0, len(all_preds)):
            #Add the predictor to the predictor set
            predictors_fw.append(all_preds[i])

            #Fit a model to the current predictor set
            trn_regr = sm.OLS(y_sets[k], X_sets[k].loc[:][predictors_fw]).fit()

            #Compute the BIC
            bic_temp = trn_regr.bic

            #If this BIC is better than the best BIC so far, store the predictor and its associated BIC in best_score
            if (bic_temp < best_score[0]):
                best_score[0] = bic_temp
                best_score[1] = all_preds[i]
                tracker += 1

            #Remove the tested predictor for the next loop
            predictors_fw.remove(all_preds[i])

        if tracker == 0:
            atEnd = True
        else:
            #Add the best predictor to the predictors set for all future loops
            predictors_fw.append(best_score[1])

            #Remove it from the all_preds set
            all_preds.remove(best_score[1])
    selected_preds_lin.append(predictors_fw)
        
```




```python
regr = LinearRegression()

#Initialize R2 Score storage structures
step_lin_train_scores = []
step_lin_test_scores = []

#Create train & test datasets per VISCODE with selected predictors
X_sigs_lin_step = [X_bl.loc[:][selected_preds_lin[0]], 
               X_06.loc[:][selected_preds_lin[1]],
               X_12.loc[:][selected_preds_lin[2]]]

X_sigs_step_lin_test = [X_bl_test.loc[:][selected_preds_lin[0]], 
                    X_06_test.loc[:][selected_preds_lin[1]], 
                    X_12_test.loc[:][selected_preds_lin[2]]]

#Get R2 Scores on Train and Test sets
for i in range(3):
    regr.fit(X_sigs_lin_step[i], y_sets[i])
    step_lin_train_scores.append(regr.score(X_sigs_lin_step[i], y_sets[i]))
    step_lin_test_scores.append(regr.score(X_sigs_step_lin_test[i], y_sets_test[i]))

#Restructure score storage structure
step_lin_records = pd.DataFrame([step_lin_train_scores, step_lin_test_scores], index = ['Train', 'Test'], columns = ['BL', '6M', '12M']).T
```




```python
#Print results of step forward linear model selection
step_lin_records
```





<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Train</th>
      <th>Test</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>BL</th>
      <td>0.495092</td>
      <td>0.089557</td>
    </tr>
    <tr>
      <th>6M</th>
      <td>0.563306</td>
      <td>0.376531</td>
    </tr>
    <tr>
      <th>12M</th>
      <td>0.721160</td>
      <td>0.276239</td>
    </tr>
  </tbody>
</table>
</div>



#### Approach 2B: Step Forward Selection for Random Forest Regression



```python
#STEP-WISE FORWARD SELECTION WITH Random Forest
selected_preds_rf = []

for k in range(3):
    #Create a list of all predictor variables
    all_preds = X_sets[k].columns.tolist()
    if 'RID' in all_preds:
        all_preds.remove('RID')

    #Start with an empty set of model predictors
    predictors_fw = []

    #Initialize the list holding the best R2 scores
    best_score = [0,0]

    #Initialize the boolean tracking whether or not better solutions are still being found
    atEnd = False

    #Loop over 
    while atEnd == False:
        #track whether or not better solutions are still being found
        tracker = 0

        for i in range(0, len(all_preds)):
            #Add the predictor to the predictor set
            predictors_fw.append(all_preds[i])

            #Fit a model to the current predictor set
            fit = rf.fit(X_sets[k].loc[:][predictors_fw], y_sets[k])

            #Compute the BIC
            temp_score = fit.score(X_sets[k].loc[:][predictors_fw], y_sets[k])
            
            #If this BIC is better than the best BIC so far, store the predictor and its associated BIC in best_score
            if (temp_score > best_score[0]):
                best_score[0] = temp_score
                best_score[1] = all_preds[i]
                tracker += 1

            #Remove the tested predictor for the next loop
            predictors_fw.remove(all_preds[i])

        if tracker == 0:
            atEnd = True
        else:
            #Add the best predictor to the predictors set for all future loops
            predictors_fw.append(best_score[1])

            #Remove it from the all_preds set
            all_preds.remove(best_score[1])
            
    selected_preds_rf.append(predictors_fw)
        
```




```python
#Get Scores from Step RF Model

#Initialize R2 Score storage structures
step_rf_train_scores = []
step_rf_test_scores = []

#Create train & test datasets per VISCODE with selected predictors
X_sigs_step_rf = [X_bl.loc[:][selected_preds_rf[0]], 
               X_06.loc[:][selected_preds_rf[1]],
               X_12.loc[:][selected_preds_rf[2]]]

X_sigs_step_rf_test = [X_bl_test.loc[:][selected_preds_rf[0]], 
                    X_06_test.loc[:][selected_preds_rf[1]], 
                    X_12_test.loc[:][selected_preds_rf[2]]]

#Get R2 Scores on Train and Test sets
for i in range(3):
    rf.fit(X_sigs_step_rf[i], y_sets[i])
    step_rf_train_scores.append(rf.score(X_sigs_step_rf[i], y_sets[i]))
    step_rf_test_scores.append(rf.score(X_sigs_step_rf_test[i], y_sets_test[i]))

#Restructure score storage structure
step_rf_records = pd.DataFrame([step_rf_train_scores, step_rf_test_scores], index = ['Train', 'Test'], columns = ['BL', '6M', '12M']).T
```




```python
#Print results from Step Forward RF
step_rf_records
```





<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Train</th>
      <th>Test</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>BL</th>
      <td>0.838665</td>
      <td>0.037376</td>
    </tr>
    <tr>
      <th>6M</th>
      <td>0.898704</td>
      <td>0.585557</td>
    </tr>
    <tr>
      <th>12M</th>
      <td>0.869852</td>
      <td>0.380147</td>
    </tr>
  </tbody>
</table>
</div>






```python
step_rf_records
```





<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Train</th>
      <th>Test</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>BL</th>
      <td>0.838665</td>
      <td>0.037376</td>
    </tr>
    <tr>
      <th>6M</th>
      <td>0.898704</td>
      <td>0.585557</td>
    </tr>
    <tr>
      <th>12M</th>
      <td>0.869852</td>
      <td>0.380147</td>
    </tr>
  </tbody>
</table>
</div>





```python
step_lin_records
```





<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Train</th>
      <th>Test</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>BL</th>
      <td>0.495092</td>
      <td>0.089557</td>
    </tr>
    <tr>
      <th>6M</th>
      <td>0.563306</td>
      <td>0.376531</td>
    </tr>
    <tr>
      <th>12M</th>
      <td>0.721160</td>
      <td>0.276239</td>
    </tr>
  </tbody>
</table>
</div>





```python
lin_signif_records
```





<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Train</th>
      <th>Test</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>BL</th>
      <td>0.555438</td>
      <td>-0.001134</td>
    </tr>
    <tr>
      <th>6M</th>
      <td>0.450924</td>
      <td>0.522852</td>
    </tr>
    <tr>
      <th>12M</th>
      <td>0.631418</td>
      <td>0.378470</td>
    </tr>
  </tbody>
</table>
</div>





```python
rf_signif_records
```





<div>
<style>
    .dataframe thead tr:only-child th {
        text-align: right;
    }

    .dataframe thead th {
        text-align: left;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>Train</th>
      <th>Test</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>BL</th>
      <td>0.668476</td>
      <td>0.194630</td>
    </tr>
    <tr>
      <th>6M</th>
      <td>0.718225</td>
      <td>0.367173</td>
    </tr>
    <tr>
      <th>12M</th>
      <td>0.819802</td>
      <td>0.358399</td>
    </tr>
  </tbody>
</table>
</div>





```python

```

