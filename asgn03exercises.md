# Plotting mechanics and EDA

**Run the code below, and read along.**

In the last assignment, I gave you a list of firms from 2020 with variables
- "gvkey", "lpermno", "lpermno" = different datasets use different identifiers for firms
- "fyear" = the fiscal year the remaining variable apply to 
- "gsector" = gsector, an industry classification (see [the wiki article on GICS](https://en.wikipedia.org/wiki/Global_Industry_Classification_Standard))
- "state" = of headquarters
- "tic" = ticker
- various accounting statistics

This data is a small slice of Compustat, which is a professional grade dataset that contains accounting data from SEC filings. 

We downloaded it and found a subsample of firms that we were interested in:


```python
import pandas as pd
import numpy as np
import seaborn as sns
import pandas_datareader as pdr  # to install: !pip install pandas_datareader
from datetime import datetime
import matplotlib.pyplot as plt 
import yfinance as yf

plt.rcParams['patch.edgecolor'] = 'none' 

# this file can be found here: https://github.com/LeDataSciFi/ledatascifi-2021/tree/main/data
# if you click on the file, then click "raw", you'll be at the url below,
# which contains the raw data. pandas can download/load it without saving it locally!
url = 'https://github.com/LeDataSciFi/data/raw/main/Firm%20Year%20Datasets%20(Compustat)/firms2020.csv'
firms_df = pd.read_csv(url).drop_duplicates('tic') 

# add leverage 
firms_df['leverage'] = (firms_df['dlc']+firms_df['dltt'])/firms_df['at']

# high_lev = 1 if firm is above the median lev in its industry
firms_df['ind_med_lev'] = firms_df.groupby('gsector')['leverage'].transform('median')
firms_df.eval('high_leverage = leverage > ind_med_lev',inplace=True)

# problem: if lev is missing, the boolean above is false, so 
# high_lev = false... even if we don't know leverage!
# let's set those to missing (nan)
mask = (firms_df["leverage"].isnull()) | (firms_df["ind_med_lev"].isnull())
firms_df.loc[mask,"high_leverage"] = None

# reduce to subsample: (has leverage value and in our sectors)
subsample = firms_df.query('gsector in [45,20] & (high_leverage == 1 | high_leverage == 0)') 
ticker_list = subsample['tic'].to_list()
```

    C:\Users\rzhan\AppData\Local\Temp\ipykernel_12052\2128127218.py:1: DeprecationWarning: 
    Pyarrow will become a required dependency of pandas in the next major release of pandas (pandas 3.0),
    (to allow more performant data types, such as the Arrow string type, and better interoperability with other libraries)
    but was not found to be installed on your system.
    If this would cause problems for you,
    please provide us feedback at https://github.com/pandas-dev/pandas/issues/54466
            
      import pandas as pd
    C:\Users\rzhan\AppData\Local\Temp\ipykernel_12052\2128127218.py:28: FutureWarning: Setting an item of incompatible dtype is deprecated and will raise an error in a future version of pandas. Value 'nan' has dtype incompatible with bool, please explicitly cast to a compatible dtype first.
      firms_df.loc[mask,"high_leverage"] = None
    

Now, 
1. I will download their daily stock returns, 
2. Compute (EW) portfolio returns,
3. Using (2), compute weekly total returns (cumulate the daily returns within each week). 
    - _Note: These are not buy-and-hold-returns, but rather **daily rebalancing!**_
    - _If you want buy-and-hold returns, you'd compute the weekly firm level returns, then average those compute the portfolio returns._


```python
##################################################################
# get daily firm stock returns
##################################################################

# I called this first df "stock_prices" in the last assignment
import warnings

# Suppress future warnings
warnings.simplefilter(action='ignore', category=FutureWarning)
firm_rets   = (yf.download(ticker_list, 
                           start=datetime(2020, 2, 2), 
                           end=datetime(2020, 4, 30),
                           show_errors=False)
                .filter(like='Adj Close') # reduce to just columns with this in the name
                .droplevel(0,axis=1) # removes the level of the col vars that said "Adj Close", leaves symbols
               
                # reshape the data tall (3 vars: firm, date, price, return)
                .stack().swaplevel().sort_index().reset_index(name='Adj Close')
                .rename(columns={'Ticker':'Firm'})
 
                # create ret vars and merge in firm-level info
                .assign(ret = lambda x: x.groupby('Firm')['Adj Close'].pct_change())
                .merge(subsample[['tic','gsector','high_leverage']],
                       left_on='Firm',
                       right_on='tic')
               
               )
 
##################################################################
# get daily portfolio returns
##################################################################

# these portfolio returns are EQUALLY weighted each day (the .mean())
# this is as if you bought all the firms in equal dollars at the beginning 
# of the day, which means "daily rebalancing" --> each day you rebalance
# your portfolio so that it's equally weighted at the start of the day

daily_port_ret = (firm_rets
                  # for each portfolio and for each day
                  .groupby(['high_leverage','gsector','Date']) 
                  ['ret'].mean()                       # avg the return for that day for the firms in the port
                  .reset_index()                       # you can work with high_leverage/sector/date as index or vars
                                                       # I decided to convert them to variables and sort                                                    
                  .sort_values(['high_leverage','gsector','Date'])
                 )

##################################################################
# get weekly portfolio returns
##################################################################

# we will cumulate the daily portfolio returns so now we have a  
# dataframe that contains weekly returns for a few different portfolios

weekly_port_ret = (daily_port_ret
                   # compute gross returns for each asset (and get the week var)
                   .assign(R = 1+daily_port_ret['ret'],
                           week = daily_port_ret['Date'].dt.isocalendar().week.astype("int64"))
                   
                   # sidenote: dt.isocalander creates a variable with type "UInt32"
                   # this doesn't play great with sns, so I turned it into an integer ("int64")
                   
                   # for each portfolio and week...
                   .groupby(['high_leverage','gsector','week'])
                   # cumulate the returns
                   ['R'].prod()
                   # subtract one
                   -1
                  ).to_frame()
                   # this last line above (to_frame) isn't strictly necessary, but 
                   # the plotting functions play nicer with frames than series objs

```

    yfinance: download(show_errors=False) argument is deprecated and will be removed in future version. Do this instead to suppress error messages: logging.getLogger('yfinance').setLevel(logging.CRITICAL)
    

    [*********************100%%**********************]  388 of 388 completed
    

## Our first plot

We can plot the weekly potfolio returns easily. 

I use `weekly_port_ret.squeeze().unstack().T` to reshape the data like pandas wants (wide). Some students will benefit in understanding from running these lines:

```python
print(weekly_port_ret)
print('-'*40)
print(weekly_port_ret.squeeze())
print('-'*40)
print(weekly_port_ret.squeeze().unstack())
print('-'*40)
print(weekly_port_ret.squeeze().unstack().T)
print('-'*40)
```

Running those will help you see why each next thing is used. 


```python
print(weekly_port_ret)
print('-'*40)
print(weekly_port_ret.squeeze())
print('-'*40)
print(weekly_port_ret.squeeze().unstack())
print('-'*40)
print(weekly_port_ret.squeeze().unstack().T)
print('-'*40)
```

                                       R
    high_leverage gsector week          
    False         20.0    6     0.013198
                          7     0.007645
                          8     0.005191
                          9    -0.111730
                          10   -0.010177
                          11   -0.123325
                          12   -0.133580
                          13    0.095817
                          14   -0.036033
                          15    0.147299
                          16    0.007942
                          17   -0.021583
                          18    0.102075
                  45.0    6     0.022533
                          7     0.019730
                          8    -0.016280
                          9    -0.105738
                          10   -0.014103
                          11   -0.152096
                          12   -0.118700
                          13    0.139054
                          14   -0.036842
                          15    0.151415
                          16    0.030923
                          17    0.012018
                          18    0.090502
    True          20.0    6     0.017848
                          7     0.025557
                          8    -0.005370
                          9    -0.120835
                          10   -0.024542
                          11   -0.166683
                          12   -0.191554
                          13    0.196561
                          14   -0.094056
                          15    0.198754
                          16   -0.039980
                          17    0.016235
                          18    0.120481
                  45.0    6     0.014527
                          7     0.036803
                          8    -0.009913
                          9    -0.101916
                          10   -0.024710
                          11   -0.142963
                          12   -0.130672
                          13    0.130414
                          14   -0.053730
                          15    0.155221
                          16    0.027395
                          17    0.005330
                          18    0.086888
    ----------------------------------------
    high_leverage  gsector  week
    False          20.0     6       0.013198
                            7       0.007645
                            8       0.005191
                            9      -0.111730
                            10     -0.010177
                            11     -0.123325
                            12     -0.133580
                            13      0.095817
                            14     -0.036033
                            15      0.147299
                            16      0.007942
                            17     -0.021583
                            18      0.102075
                   45.0     6       0.022533
                            7       0.019730
                            8      -0.016280
                            9      -0.105738
                            10     -0.014103
                            11     -0.152096
                            12     -0.118700
                            13      0.139054
                            14     -0.036842
                            15      0.151415
                            16      0.030923
                            17      0.012018
                            18      0.090502
    True           20.0     6       0.017848
                            7       0.025557
                            8      -0.005370
                            9      -0.120835
                            10     -0.024542
                            11     -0.166683
                            12     -0.191554
                            13      0.196561
                            14     -0.094056
                            15      0.198754
                            16     -0.039980
                            17      0.016235
                            18      0.120481
                   45.0     6       0.014527
                            7       0.036803
                            8      -0.009913
                            9      -0.101916
                            10     -0.024710
                            11     -0.142963
                            12     -0.130672
                            13      0.130414
                            14     -0.053730
                            15      0.155221
                            16      0.027395
                            17      0.005330
                            18      0.086888
    Name: R, dtype: float64
    ----------------------------------------
    week                         6         7         8         9         10  \
    high_leverage gsector                                                     
    False         20.0     0.013198  0.007645  0.005191 -0.111730 -0.010177   
                  45.0     0.022533  0.019730 -0.016280 -0.105738 -0.014103   
    True          20.0     0.017848  0.025557 -0.005370 -0.120835 -0.024542   
                  45.0     0.014527  0.036803 -0.009913 -0.101916 -0.024710   
    
    week                         11        12        13        14        15  \
    high_leverage gsector                                                     
    False         20.0    -0.123325 -0.133580  0.095817 -0.036033  0.147299   
                  45.0    -0.152096 -0.118700  0.139054 -0.036842  0.151415   
    True          20.0    -0.166683 -0.191554  0.196561 -0.094056  0.198754   
                  45.0    -0.142963 -0.130672  0.130414 -0.053730  0.155221   
    
    week                         16        17        18  
    high_leverage gsector                                
    False         20.0     0.007942 -0.021583  0.102075  
                  45.0     0.030923  0.012018  0.090502  
    True          20.0    -0.039980  0.016235  0.120481  
                  45.0     0.027395  0.005330  0.086888  
    ----------------------------------------
    high_leverage     False               True           
    gsector            20.0      45.0      20.0      45.0
    week                                                 
    6              0.013198  0.022533  0.017848  0.014527
    7              0.007645  0.019730  0.025557  0.036803
    8              0.005191 -0.016280 -0.005370 -0.009913
    9             -0.111730 -0.105738 -0.120835 -0.101916
    10            -0.010177 -0.014103 -0.024542 -0.024710
    11            -0.123325 -0.152096 -0.166683 -0.142963
    12            -0.133580 -0.118700 -0.191554 -0.130672
    13             0.095817  0.139054  0.196561  0.130414
    14            -0.036033 -0.036842 -0.094056 -0.053730
    15             0.147299  0.151415  0.198754  0.155221
    16             0.007942  0.030923 -0.039980  0.027395
    17            -0.021583  0.012018  0.016235  0.005330
    18             0.102075  0.090502  0.120481  0.086888
    ----------------------------------------
    


```python
ax = weekly_port_ret.squeeze().unstack().T.plot()
# can access customization via matplotliab methods on ax 
plt.show()
```


    
![png](output_7_0.png)
    


Doing this in seaborn is easy too, and I don't need to modify the data at all.


```python
ax = sns.lineplot(data = weekly_port_ret,
             x='week',y='R',hue='high_leverage',style='gsector')
# can access customization via matplotlib methods on ax 
plt.show()

```


    
![png](output_9_0.png)
    


## Part 1 - Plot formatting

Insert cell(s) below this one as needed to finish this Part.

Improve the plot above.
- Q1: set the title to "Weekly Portfolio Returns - Daily Rebalancing"
- Q2: set the x-axis title to "Week in 2020"
- Q3: set the y-axis title to "Weekly Return"
- Q4: Ungraded bonus challenge: change the legend so it says the industry names, not the numbers 



```python
#Q1-3
ax = sns.lineplot(data = weekly_port_ret,
             x='week',y='R',hue='high_leverage',style='gsector').set(title="Weekly Portfolio Returns - Daily Rebalancing", xlabel = "Week in 2020", ylabel = "Weekly Return")
# can access customization via matplotlib methods on ax 
plt.show()
```


    
![png](output_11_0.png)
    


## Part 2 - Replicate/Imitate

Insert cell(s) below each bullet point and create as close a match as you can. This includes titles, axis numbering, everything you see. 

- Q5: Replicate F1.png. Notice the x-axis has no label - it's in the title.



```python
new_ax = sns.displot(data=weekly_port_ret, x="R", kde=True).set(title="Weekly Portfolio Returns", xlabel = "")
plt.show()
```


    
![png](output_13_0.png)
    


![](data/F1-b.png)

- Q6: Replicate F2.png. Notice the bin sizes are 5%. (From 0-5%, 5-10%, ...)


```python
bin_width = 0.05
bin_edges = [i * bin_width for i in range(-8, 7)]
new_ax = sns.displot(data=weekly_port_ret, x="R", bins=bin_edges).set(title="Weekly Portfolio Returns", xlabel="")
plt.show()
```


    
![png](output_16_0.png)
    


![](data/F2-b.png)

- Q7: Replicate F3.png. Pay attention to the header for a clue! 
    


```python
box_ax = sns.boxplot(y='ret', data=firm_rets).set(title = "Firm Daily Returns", ylabel = "")
plt.show()

```


    
![png](output_19_0.png)
    


![](data/F3.png)  

- Q8: Replicate F4.png. Pay attention to the header for a clue!
    


```python
box_ax = sns.boxplot(x = 'high_leverage', y='ret', data=firm_rets).set(title = "Firm Daily Returns", ylabel = "")
plt.show()
```


    
![png](output_22_0.png)
    


![](data/F4.png)  

- Q9: Replicate this figure, using this `total` dataset:

```python
total = pd.DataFrame() # open an empty dataframe
total['ret'] = (firm_rets.assign(ret=firm_rets['ret']+1) # now we have R(t) for each observation
                       .groupby('tic')['ret']    # for each firm,
                       .prod()                      # multiple all the gross returns
                       -1                           # and subtract one to get back to the total period return
)
total['cnt'] = firm_rets.groupby('tic')['ret'].count()
total['std'] = firm_rets.groupby('tic')['ret'].std()*np.sqrt(total['cnt'])
total = total.merge(firm_rets.groupby('tic')[['high_leverage','gsector']].first(), 
                    left_index=True, right_index=True)
```


```python
total = pd.DataFrame() # open an empty dataframe
total['ret'] = (firm_rets.assign(ret=firm_rets['ret']+1) # now we have R(t) for each observation
                       .groupby('tic')['ret']    # for each firm,
                       .prod()                      # multiple all the gross returns
                       -1                           # and subtract one to get back to the total period return
)
total['cnt'] = firm_rets.groupby('tic')['ret'].count()
total['std'] = firm_rets.groupby('tic')['ret'].std()*np.sqrt(total['cnt'])
total = total.merge(firm_rets.groupby('tic')[['high_leverage','gsector']].first(), 
                    left_index=True, right_index=True)

scatter_ax = sns.scatterplot(data=total, x="std", y="ret", hue="high_leverage", style="gsector").set(title = "Risk and Return(Feb, Mar, Apr 2020)", xlabel = "STD of return", ylabel = "Return")
plt.show()
```


    
![png](output_25_0.png)
    


![](data/F5.png)

## Q10: Choose your adventure.

Make one cool plot. Some ideas:
- Use a pairplot, jointplot, or heatmap on any data already loaded on this page (including the original `firms_df`). 
- Convert any of the stock price datasets to a "wide" format and then use the pair_hex_bin code from the communiy codebook.
- Do something fun with the parameters of the function you choose. 
- [Or adapt this](https://seaborn.pydata.org/examples/timeseries_facets.html) to improve  our portfolio returns plot from Part 1, because Part 1 created a tough to interpret "spaghetti" plot!
- Plot the _cumulative_ returns over the sample period for the four portfolios' weekly returns from Part 1
- [Adapt this](https://seaborn.pydata.org/examples/timeseries_facets.html) to improve  our weekly portfolio returns plot from Part 1.

Then save the figure as a png file and [share it here on the discussion board](https://github.com/orgs/LeDataSciFi/teams/classmates-2023/discussions/21).


```python
heatmap_data = total.pivot_table(index='high_leverage', columns='gsector', values='ret')

plt.figure(figsize=(10, 6)) 
sns.heatmap(heatmap_data, annot=True, fmt=".2f", cmap="coolwarm")
plt.title("Total Returns by Sector and High Leverage")
plt.xlabel("Sector")
plt.ylabel("High Leverage")
plt.show()
```


    
![png](output_28_0.png)
    


## Q11 Optional problems - using ChatGPT to fix and improve a function

### 11a - The error

I included a file in this folder called `factor_loading_simple_fcn.py`. It's adapted from the similar-named version in the handouts folder, but I made it a function.

You'll notice that the next cell, working on AAPL and MFST, works (though it provides a `FutureWarning` indicating that we should update it to prevent it from breaking in the future).

The cell below, using just AAPL, however, breaks it. One line of code doesn't work with a single stock. We need to replace it with <something> that handles a single stock request. 

Try to fix it using GPT: Copy the function code into GPT (inside triple ticks), and then copy the error output into GPT, asking for a fix. 

If it works, both of the cells below will work. If you fix it, please put the fix below in the placeholder. 

### 11b - Improving the function's code style

Ask GPT to completely redo all the comments and documentation - to improve it stylistically and for clarity. 

### 11c - Suggestions

- Ask ChatGPT for 5 feature requests that could improve the function (don't implement them), and to concisely explain how it might work
- Ask ChatGPT to identify 3 ways the function could fail and to concisely explain how the function can be adjusted to deal with these errors (don't fix them)

---

Put its suggestions here, and bold your favorites:
- a
- b
- c

```python
Copy your new code that fixed the function here.  
```


```python
from factor_loading_simple_fcn import estimate_factor_loadings
from datetime import datetime

# choose your firms and years 
stocks = ['SBUX','AAPL','MSFT']
start  = datetime(2016, 1, 1)
end    = datetime(2016, 12, 31)

estimate_factor_loadings(['AAPL','MSFT'],datetime(2016,1,1),datetime(2016,12,31))

```

    [*********************100%%**********************]  2 of 2 completed
    


    ---------------------------------------------------------------------------

    ModuleNotFoundError                       Traceback (most recent call last)

    Cell In[13], line 9
          6 start  = datetime(2016, 1, 1)
          7 end    = datetime(2016, 12, 31)
    ----> 9 estimate_factor_loadings(['AAPL','MSFT'],datetime(2016,1,1),datetime(2016,12,31))
    

    File ~\Desktop\FIN377\asgn-03-alz425\factor_loading_simple_fcn.py:138, in estimate_factor_loadings(stocks, start, end, formula)
        121 assets_and_factors
        124 # ## Estimate CAPM
        125 # 
        126 # So the data’s basically ready. _(We need to do two quick things below.)_
       (...)
        135 # 
        136 # We just need to write a reg function that works on groupby objects.
    --> 138 import statsmodels.api as sm
        140 def reg_in_groupby(df, formula="ret_excess ~ mkt_excess + SMB + HML"):
        141     """
        142     Want to run regressions after groupby? E.g., repeat the regression 
        143     for each firm-year?
       (...)
        152         df.groupby(<whatever>).apply(reg_in_groupby,formula=<whatever>)
        153     """
    

    ModuleNotFoundError: No module named 'statsmodels'



```python
estimate_factor_loadings(['AAPL'],datetime(2016,1,1),datetime(2016,12,31))

```


```python

```
