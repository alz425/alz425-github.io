 ---
 layout: wide_default
 ---
## Part 1: EDA

_Insert cells as needed below to write a short EDA/data section that summarizes the data for someone who has never opened it before._ 
- Answer essential questions about the dataset (observation units, time period, sample size, many of the questions above) 
- Note any issues you have with the data (variable X has problem Y that needs to get addressed before using it in regressions or a prediction model because Z)
- Present any visual results you think are interesting or important


```python
import pandas as pd
import numpy as np
housing_train = pd.read_csv('input_data2/housing_train.csv')
```

    C:\Users\rzhan\AppData\Local\Temp\ipykernel_16292\3131767995.py:1: DeprecationWarning: 
    Pyarrow will become a required dependency of pandas in the next major release of pandas (pandas 3.0),
    (to allow more performant data types, such as the Arrow string type, and better interoperability with other libraries)
    but was not found to be installed on your system.
    If this would cause problems for you,
    please provide us feedback at https://github.com/pandas-dev/pandas/issues/54466
            
      import pandas as pd
    

- Parcel is the unit of observation
- There are 1941 rows in the dataset
- The time period is from 2006-2008 according to v_Yr_Sold.
- There are 81 variables
- Sale Price is from 13100 to 755000
- Sale Price has outliers
- There are 27 columns with missing values
- Categorical vars include: v_Overall_Qual, v_Overall_Cond


```python
import seaborn as sns
# Display sale price boxplot with outliers
sns.boxplot(housing_train['v_SalePrice'])
```




    <Axes: ylabel='v_SalePrice'>




    
![png](output_3_1.png)
    


## Part 2: Running Regressions

**Run these regressions on the RAW data, even if you found data issues that you think should be addressed.**

_Insert cells as needed below to run these regressions. Note that $i$ is indexing a given house, and $t$ indexes the year of sale._ 

1. $\text{Sale Price}_{i,t} = \alpha + \beta_1 * \text{v\_Lot\_Area}$
1. $\text{Sale Price}_{i,t} = \alpha + \beta_1 * log(\text{v\_Lot\_Area})$
1. $log(\text{Sale Price}_{i,t}) = \alpha + \beta_1 * \text{v\_Lot\_Area}$
1. $log(\text{Sale Price}_{i,t}) = \alpha + \beta_1 * log(\text{v\_Lot\_Area})$
1. $log(\text{Sale Price}_{i,t}) = \alpha + \beta_1 * \text{v\_Yr\_Sold}$
1. $log(\text{Sale Price}_{i,t}) = \alpha + \beta_1 * (\text{v\_Yr\_Sold==2007})+ \beta_2 * (\text{v\_Yr\_Sold==2008})$
1. Choose your own adventure: Pick any five variables from the dataset that you think will generate good R2. Use them in a regression of $log(\text{Sale Price}_{i,t})$ 
    - Tip: You can transform/create these five variables however you want, even if it creates extra variables. For example: I'd count Model 6 above as only using one variable: `v_Yr_Sold`.
    - I got an R2 of 0.877 with just "5" variables. How close can you get? I won't be shocked if someone beats that!
    

**Bonus formatting trick:** Instead of reporting all regressions separately, report all seven regressions in a _single_ table using `summary_col`.



```python
# Q1: 
from statsmodels.formula.api import ols as sm_ols
from statsmodels.iolib.summary2 import summary_col 

model1   = sm_ols('v_SalePrice ~ v_Lot_Area',  # specify model (you don't need to include the constant!)
                  data=housing_train)
results1 = model1.fit()               # estimate / fit
print(results1.summary())             # view results ... identical to before
y_predicted1 = results1.predict()     # get the predicted results
residuals1 = results1.resid           # get the residuals
print('\n\nParams:')
print(results1.params)
```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:            v_SalePrice   R-squared:                       0.067
    Model:                            OLS   Adj. R-squared:                  0.066
    Method:                 Least Squares   F-statistic:                     138.3
    Date:                Sun, 31 Mar 2024   Prob (F-statistic):           6.82e-31
    Time:                        16:49:44   Log-Likelihood:                -24610.
    No. Observations:                1941   AIC:                         4.922e+04
    Df Residuals:                    1939   BIC:                         4.924e+04
    Df Model:                           1                                         
    Covariance Type:            nonrobust                                         
    ==============================================================================
                     coef    std err          t      P>|t|      [0.025      0.975]
    ------------------------------------------------------------------------------
    Intercept   1.548e+05   2911.591     53.163      0.000    1.49e+05     1.6e+05
    v_Lot_Area     2.6489      0.225     11.760      0.000       2.207       3.091
    ==============================================================================
    Omnibus:                      668.513   Durbin-Watson:                   1.064
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):             3001.894
    Skew:                           1.595   Prob(JB):                         0.00
    Kurtosis:                       8.191   Cond. No.                     2.13e+04
    ==============================================================================
    
    Notes:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    [2] The condition number is large, 2.13e+04. This might indicate that there are
    strong multicollinearity or other numerical problems.
    
    
    Params:
    Intercept     154789.550207
    v_Lot_Area         2.648935
    dtype: float64
    


```python
# Q2:

model2   = sm_ols("v_SalePrice ~ np.log(v_Lot_Area)", 
       data=housing_train)
results2 = model2.fit()               # estimate / fit
print(results2.summary())             # view results ... identical to before
y_predicted2 = results2.predict()     # get the predicted results
residuals2 = results2.resid           # get the residuals
print('\n\nParams:')
print(results2.params)


```

                                OLS Regression Results                            
    ==============================================================================
    Dep. Variable:            v_SalePrice   R-squared:                       0.128
    Model:                            OLS   Adj. R-squared:                  0.128
    Method:                 Least Squares   F-statistic:                     285.6
    Date:                Sun, 31 Mar 2024   Prob (F-statistic):           6.95e-60
    Time:                        16:49:44   Log-Likelihood:                -24544.
    No. Observations:                1941   AIC:                         4.909e+04
    Df Residuals:                    1939   BIC:                         4.910e+04
    Df Model:                           1                                         
    Covariance Type:            nonrobust                                         
    ======================================================================================
                             coef    std err          t      P>|t|      [0.025      0.975]
    --------------------------------------------------------------------------------------
    Intercept          -3.279e+05   3.02e+04    -10.850      0.000   -3.87e+05   -2.69e+05
    np.log(v_Lot_Area)  5.603e+04   3315.139     16.901      0.000    4.95e+04    6.25e+04
    ==============================================================================
    Omnibus:                      650.067   Durbin-Watson:                   1.042
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):             2623.687
    Skew:                           1.587   Prob(JB):                         0.00
    Kurtosis:                       7.729   Cond. No.                         164.
    ==============================================================================
    
    Notes:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    
    
    Params:
    Intercept            -327915.80232
    np.log(v_Lot_Area)     56028.16996
    dtype: float64
    


```python
# Q3:

model3   = sm_ols("np.log(v_SalePrice) ~ v_Lot_Area", 
       data=housing_train)
results3 = model3.fit()               # estimate / fit
print(results3.summary())             # view results ... identical to before
y_predicted3 = results3.predict()     # get the predicted results
residuals3 = results3.resid           # get the residuals
print('\n\nParams:')
print(results3.params)
```

                                 OLS Regression Results                            
    ===============================================================================
    Dep. Variable:     np.log(v_SalePrice)   R-squared:                       0.065
    Model:                             OLS   Adj. R-squared:                  0.064
    Method:                  Least Squares   F-statistic:                     133.9
    Date:                 Sun, 31 Mar 2024   Prob (F-statistic):           5.46e-30
    Time:                         16:49:44   Log-Likelihood:                -927.19
    No. Observations:                 1941   AIC:                             1858.
    Df Residuals:                     1939   BIC:                             1870.
    Df Model:                            1                                         
    Covariance Type:             nonrobust                                         
    ==============================================================================
                     coef    std err          t      P>|t|      [0.025      0.975]
    ------------------------------------------------------------------------------
    Intercept     11.8941      0.015    813.211      0.000      11.865      11.923
    v_Lot_Area  1.309e-05   1.13e-06     11.571      0.000    1.09e-05    1.53e-05
    ==============================================================================
    Omnibus:                       75.460   Durbin-Watson:                   0.980
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):              218.556
    Skew:                          -0.066   Prob(JB):                     3.48e-48
    Kurtosis:                       4.639   Cond. No.                     2.13e+04
    ==============================================================================
    
    Notes:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    [2] The condition number is large, 2.13e+04. This might indicate that there are
    strong multicollinearity or other numerical problems.
    
    
    Params:
    Intercept     11.894073
    v_Lot_Area     0.000013
    dtype: float64
    


```python
#Q4:

model4   = sm_ols("np.log(v_SalePrice) ~ np.log(v_Lot_Area)", 
       data=housing_train)
results4 = model4.fit()               # estimate / fit
print(results4.summary())             # view results ... identical to before
y_predicted4 = results4.predict()     # get the predicted results
residuals4 = results4.resid           # get the residuals
print('\n\nParams:')
print(results4.params)
```

                                 OLS Regression Results                            
    ===============================================================================
    Dep. Variable:     np.log(v_SalePrice)   R-squared:                       0.135
    Model:                             OLS   Adj. R-squared:                  0.135
    Method:                  Least Squares   F-statistic:                     302.5
    Date:                 Sun, 31 Mar 2024   Prob (F-statistic):           4.38e-63
    Time:                         16:49:44   Log-Likelihood:                -851.27
    No. Observations:                 1941   AIC:                             1707.
    Df Residuals:                     1939   BIC:                             1718.
    Df Model:                            1                                         
    Covariance Type:             nonrobust                                         
    ======================================================================================
                             coef    std err          t      P>|t|      [0.025      0.975]
    --------------------------------------------------------------------------------------
    Intercept              9.4051      0.151     62.253      0.000       9.109       9.701
    np.log(v_Lot_Area)     0.2883      0.017     17.394      0.000       0.256       0.321
    ==============================================================================
    Omnibus:                       84.067   Durbin-Watson:                   0.955
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):              255.283
    Skew:                          -0.100   Prob(JB):                     3.68e-56
    Kurtosis:                       4.765   Cond. No.                         164.
    ==============================================================================
    
    Notes:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    
    
    Params:
    Intercept             9.405051
    np.log(v_Lot_Area)    0.288263
    dtype: float64
    


```python
# Q5:

model5   = sm_ols("np.log(v_SalePrice) ~ v_Yr_Sold", 
       data=housing_train)
results5 = model5.fit()               # estimate / fit
print(results5.summary())             # view results ... identical to before
y_predicted5 = results5.predict()     # get the predicted results
residuals5 = results5.resid           # get the residuals
print('\n\nParams:')
print(results5.params)
```

                                 OLS Regression Results                            
    ===============================================================================
    Dep. Variable:     np.log(v_SalePrice)   R-squared:                       0.000
    Model:                             OLS   Adj. R-squared:                 -0.000
    Method:                  Least Squares   F-statistic:                    0.2003
    Date:                 Sun, 31 Mar 2024   Prob (F-statistic):              0.655
    Time:                         16:49:45   Log-Likelihood:                -991.88
    No. Observations:                 1941   AIC:                             1988.
    Df Residuals:                     1939   BIC:                             1999.
    Df Model:                            1                                         
    Covariance Type:             nonrobust                                         
    ==============================================================================
                     coef    std err          t      P>|t|      [0.025      0.975]
    ------------------------------------------------------------------------------
    Intercept     22.2932     22.937      0.972      0.331     -22.690      67.277
    v_Yr_Sold     -0.0051      0.011     -0.448      0.655      -0.028       0.017
    ==============================================================================
    Omnibus:                       55.641   Durbin-Watson:                   0.985
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):              131.833
    Skew:                           0.075   Prob(JB):                     2.36e-29
    Kurtosis:                       4.268   Cond. No.                     5.03e+06
    ==============================================================================
    
    Notes:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    [2] The condition number is large, 5.03e+06. This might indicate that there are
    strong multicollinearity or other numerical problems.
    
    
    Params:
    Intercept    22.293213
    v_Yr_Sold    -0.005114
    dtype: float64
    


```python
# Q6: 

model6   = sm_ols("np.log(v_SalePrice) ~ v_Yr_Sold==2007 + v_Yr_Sold==2008", 
       data=housing_train)
results6 = model6.fit()               # estimate / fit
print(results6.summary())             # view results ... identical to before
y_predicted6 = results6.predict()     # get the predicted results
residuals6 = results6.resid           # get the residuals
print('\n\nParams:')
print(results6.params)
```

                                 OLS Regression Results                            
    ===============================================================================
    Dep. Variable:     np.log(v_SalePrice)   R-squared:                       0.001
    Model:                             OLS   Adj. R-squared:                  0.000
    Method:                  Least Squares   F-statistic:                     1.394
    Date:                 Sun, 31 Mar 2024   Prob (F-statistic):              0.248
    Time:                         16:49:45   Log-Likelihood:                -990.59
    No. Observations:                 1941   AIC:                             1987.
    Df Residuals:                     1938   BIC:                             2004.
    Df Model:                            2                                         
    Covariance Type:             nonrobust                                         
    =============================================================================================
                                    coef    std err          t      P>|t|      [0.025      0.975]
    ---------------------------------------------------------------------------------------------
    Intercept                    12.0229      0.016    745.087      0.000      11.991      12.055
    v_Yr_Sold == 2007[T.True]     0.0256      0.022      1.150      0.250      -0.018       0.069
    v_Yr_Sold == 2008[T.True]    -0.0103      0.023     -0.450      0.653      -0.055       0.035
    ==============================================================================
    Omnibus:                       54.618   Durbin-Watson:                   0.989
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):              127.342
    Skew:                           0.080   Prob(JB):                     2.23e-28
    Kurtosis:                       4.245   Cond. No.                         3.79
    ==============================================================================
    
    Notes:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    
    
    Params:
    Intercept                    12.022869
    v_Yr_Sold == 2007[T.True]     0.025590
    v_Yr_Sold == 2008[T.True]    -0.010282
    dtype: float64
    


```python
# Q7

model7   = sm_ols("np.log(v_SalePrice) ~ v_Yr_Sold==2007 + v_Yr_Sold==2008 + np.log(v_Lot_Area) + v_Year_Built + v_Total_Bsmt_SF + v_TotRms_AbvGrd", 
       data=housing_train)
results7 = model7.fit()               # estimate / fit
print(results7.summary())             # view results ... identical to before
y_predicted7 = results7.predict()     # get the predicted results
residuals7 = results7.resid           # get the residuals
print('\n\nParams:')
print(results7.params)
```

                                 OLS Regression Results                            
    ===============================================================================
    Dep. Variable:     np.log(v_SalePrice)   R-squared:                       0.654
    Model:                             OLS   Adj. R-squared:                  0.653
    Method:                  Least Squares   F-statistic:                     608.6
    Date:                 Sun, 31 Mar 2024   Prob (F-statistic):               0.00
    Time:                         16:49:45   Log-Likelihood:                 38.861
    No. Observations:                 1940   AIC:                            -63.72
    Df Residuals:                     1933   BIC:                            -24.73
    Df Model:                            6                                         
    Covariance Type:             nonrobust                                         
    =============================================================================================
                                    coef    std err          t      P>|t|      [0.025      0.975]
    ---------------------------------------------------------------------------------------------
    Intercept                    -1.3975      0.407     -3.431      0.001      -2.196      -0.599
    v_Yr_Sold == 2007[T.True]     0.0074      0.013      0.561      0.575      -0.018       0.033
    v_Yr_Sold == 2008[T.True]     0.0129      0.013      0.959      0.338      -0.013       0.039
    np.log(v_Lot_Area)            0.1118      0.012      9.503      0.000       0.089       0.135
    v_Year_Built                  0.0059      0.000     29.874      0.000       0.005       0.006
    v_Total_Bsmt_SF               0.0003   1.46e-05     18.252      0.000       0.000       0.000
    v_TotRms_AbvGrd               0.0822      0.004     21.867      0.000       0.075       0.090
    ==============================================================================
    Omnibus:                      665.488   Durbin-Watson:                   1.531
    Prob(Omnibus):                  0.000   Jarque-Bera (JB):            10047.114
    Skew:                          -1.191   Prob(JB):                         0.00
    Kurtosis:                      13.891   Cond. No.                     1.70e+05
    ==============================================================================
    
    Notes:
    [1] Standard Errors assume that the covariance matrix of the errors is correctly specified.
    [2] The condition number is large, 1.7e+05. This might indicate that there are
    strong multicollinearity or other numerical problems.
    
    
    Params:
    Intercept                   -1.397496
    v_Yr_Sold == 2007[T.True]    0.007358
    v_Yr_Sold == 2008[T.True]    0.012920
    np.log(v_Lot_Area)           0.111805
    v_Year_Built                 0.005880
    v_Total_Bsmt_SF              0.000266
    v_TotRms_AbvGrd              0.082173
    dtype: float64
    

## Part 3: Regression interpretation

_Insert cells as needed below to answer these questions. Note that $i$ is indexing a given house, and $t$ indexes the year of sale._ 

1. If you didn't use the `summary_col` trick, list $\beta_1$ for Models 1-6 to make it easier on your graders.
1. Interpret $\beta_1$ in Model 2. 
1. Interpret $\beta_1$ in Model 3. 
    - HINT: You might need to print out more decimal places. Show at least 2 non-zero digits. 
1. Of models 1-4, which do you think best explains the data and why?
1. Interpret $\beta_1$ In Model 5
1. Interpret $\alpha$ in Model 6
1. Interpret $\beta_1$ in Model 6
1. Why is the R2 of Model 6 higher than the R2 of Model 5?
1. What variables did you include in Model 7?
1. What is the R2 of your Model 7?
1. Speculate (not graded): Could you use the specification of Model 6 in a predictive regression? 
1. Speculate (not graded): Could you use the specification of Model 5 in a predictive regression? 



```python
# Q1: summary_col, can also look at results above for each model
info_dict={'R-squared' : lambda x: f"{x.rsquared:.2f}",
           'Adj R-squared' : lambda x: f"{x.rsquared_adj:.2f}",
           'No. observations' : lambda x: f"{int(x.nobs):d}"}
print('='*108)
print('                  y = sale price if not specified, log(sale price else)')
print(summary_col(results=[results1, results2, results3, results4, results5, results6, results7], # list the result obj here
                  float_format='%0.2f',
                  stars = True, # stars are easy way to see if anything is statistically significant
                  model_names=['1','2',' 3 (log)','4 (log)','5 (log)','6 (log)','7 (log)','8 (log)'], # these are bad names, lol. Usually, just use the y variable name
                  info_dict=info_dict,
                  regressor_order=[ 'Intercept','v_Lot_Area','np.log(v_Lot_Area)','v_Yr_Sold', 'v_Yr_Sold==2007', 'v_Yr_Sold==2008', 'v_Yr_Built', 'v_Total_Bsmt_SF', 'v_TotRms_AbvGrd']
                  )
     )
```

    ============================================================================================================
                      y = sale price if not specified, log(sale price else)
    
    ===============================================================================================
                                   1             2        3 (log) 4 (log) 5 (log) 6 (log)  7 (log) 
    -----------------------------------------------------------------------------------------------
    Intercept                 154789.55*** -327915.80*** 11.89*** 9.41*** 22.29   12.02*** -1.40***
                              (2911.59)    (30221.35)    (0.01)   (0.15)  (22.94) (0.02)   (0.41)  
    v_Lot_Area                2.65***                    0.00***                                   
                              (0.23)                     (0.00)                                    
    np.log(v_Lot_Area)                     56028.17***            0.29***                  0.11*** 
                                           (3315.14)              (0.02)                   (0.01)  
    v_Yr_Sold                                                             -0.01                    
                                                                          (0.01)                   
    v_Total_Bsmt_SF                                                                        0.00*** 
                                                                                           (0.00)  
    v_TotRms_AbvGrd                                                                        0.08*** 
                                                                                           (0.00)  
    v_Yr_Sold == 2007[T.True]                                                     0.03     0.01    
                                                                                  (0.02)   (0.01)  
    v_Yr_Sold == 2008[T.True]                                                     -0.01    0.01    
                                                                                  (0.02)   (0.01)  
    v_Year_Built                                                                           0.01*** 
                                                                                           (0.00)  
    R-squared                 0.07         0.13          0.06     0.13    0.00    0.00     0.65    
    R-squared Adj.            0.07         0.13          0.06     0.13    -0.00   0.00     0.65    
    Adj R-squared             0.07         0.13          0.06     0.13    -0.00   0.00     0.65    
    No. observations          1941         1941          1941     1941    1941    1941     1940    
    R-squared                 0.07         0.13          0.06     0.13    0.00    0.00     0.65    
    ===============================================================================================
    Standard errors in parentheses.
    * p<.1, ** p<.05, ***p<.01
    

Q2: A one percent increase in lot area, is associated with a 560.2817 p.p. increase in sale price

Q3: v_Lot_Area in full is 0.000013. A one unit increase in lot area, is associated with a .0013% increase in sale price 

Q4: I think Model 4 best explains the data given that its R2 and adjusted R2 is the highest at .135 out of all of the models, which indicates better fit. Its AIC and BIC metrics are lower than the other models as well, which indicates better fit as well since lower = better.

Q5: v_Yr_Sold in full is -0.005114. A one unit increase in yr sold, is associated with a 0.5114% decline in sale price. 

Q6: A house with years sold of 2007 and 2008 has a sales price of High (meaningless) on avg in the data


```python
np.exp(12.02)
```




    166042.65630144285



Q7: v_Yr_Sold == 2007 in full is 0.025590. A one unit increase in years sold == 2007, is associated with a 2.559% increase in sale price, holding all other variables (v_Yr_Sold == 2008) constant. 

Q8: The R2 of Model 6 is higher than the R2 of Model 5 because specifying the years sold as only 2007 and 2008 allows the model to cover potential differences between the yr sold and sale price. If the relationship differs from yr to yr, Model 6 accounts for this better. 

Q9: I included lot area, year sold 2007 and 2008, total basement SF, total rooms above ground, and year built as the 5 variables.

Q10: The R2 of Model 7 is 0.65



```python

```
