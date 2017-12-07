---
title: Modeling
notebook: models.ipynb
---

## Contents
{:.no_toc}
*  
{: toc}





## Attempts to Improve the Baseline Model

### Approach 1: "Most Significant Predictors"

We fit Linear Regression & Random Forest Regressor models to the individual questions on each cognitive test. We fit each test separately and found the most significant predictors on each fit. For the Linear Regression fits, we considered predictors with a p-value of less than 0.03 to be significant (we used a stringent p-value because using less-stringent p-values such as 0.05 resulted in overfitting from too many predictors being selected). For the Random Forest fits, we simply selected the predictor that was considered most-significant in each fit. We ran 100 iterations of this process on different subsets of the training data, and then selected out the predictors that were considered significant in at least 50% of the trials.











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


