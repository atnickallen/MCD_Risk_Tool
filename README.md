<p align="center">
  <a href="" rel="noopener">
 <img src="https://user-images.githubusercontent.com/39813026/62629404-116a6c00-b8fb-11e9-8a95-3f48f38045ea.png"></a>
</p>
<h3 align="center">MakerDAO MCD Risk Tool</h3>

<div align="center">
 
  [![Status](https://img.shields.io/badge/status-active-success.svg)]() 
  [![GitHub Pull Requests](https://img.shields.io/github/issues-pr/kylelobo/The-Documentation-Compendium.svg)](https://github.com/kylelobo/The-Documentation-Compendium/pulls)
  [![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE.md)

</div>

---

<p align="center"> Analyzing different collateral type risk with models
    <br> 
</p>

*Code will be availible once Readme has been reviewed*

## 🗄 Table of Contents
- [The Goal](#Goal)
- [Setup](#setup)
- [Downloading The Prerequisites](#pre)
- [Cloning The Repo](#cloning)
- [Loading the Models](#loading)
- [For Analysis](#analysis)
- [Running the Models](#running)
- [MCD Risk Model 1](#model1)
- [Authors](#authors)

## Goal of the Risk Tool: <a name = "Goal"></a>
Goal of the tool is to display risk and allow users to double click on visually evaluating risk.

## 💽 Setup: <a name = "setup"></a>
If you would like to install and use this tool on your own machine make sure you have `pipenv` installed. If you need to install follow [these simple instructions](https://github.com/pypa/pipenv#installation).

### Downloading The Prerequisites <a name = "pre"></a>

The packages needed to run this tool: `pipenv`, `homebrew`, `python3`, `pandas`, and `numpy`.

The packages `pandas` and `numpy` will need to be installed in order for these repositories to work. If you do not have them installed, run these commands in your terminal:

```
pipenv install pandas
```

```
pipenv install numpy
```


### Cloning The Repo <a name = "cloning"></a>

Clone the MCD Risk Model Repo:

```
git clone https://github.com/madison1111/mcd_risk_model.git
```

Then cd into the directory:

```
cd mcd_risk_model
```

### Loading the Models <a name = "loading"></a>
To reproduce results run the following commands:

```
python3 mcd_risk_model_1.py
```

```
python3 mcd_risk_model_2.py
```

### For Analysis <a name = "analysis"></a>
Start your Jupyter notebook:

```
jupyter notebook
```

If your browser doesn't automatically open, then go to:

```
http://localhost:8888
```

## 🎛 Running the Models <a name = "running"></a>
Notebooks can be found in the `notebooks` folder

### MCD Risk Model 1: <a name = "model1"></a>

```
mcd_risk_model_1.py
```

### Step 1:
Before doing anything else this script imports the various dependencies that allow for the script to be run in python as well as creates a class for the initial variables that we need

```
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd
import time
```

And create the class:


```
class mcd_model_1:
    def __init__(self, start, end):
        self.start = start
        self.end = end
```

After, we look to call our data and will make two separate methods within our mcd_model_1 class, one for a single collateral type, and one for multiple collateral types

Data is encapsulated in Risk RESTful API

## Historical Data
Collect trading history across all exchanges

### Read in data
Data methodology is outlined in Risk API github repo

### Historical exchange data

There is no ‘master exchange’ to pull from so we pull from several BTI verified exchanges, as price is determined by supply and demand and wash trading is a real problem among cryptocurrency exchanges

We pull data from 6 major exchanges: `Coinbase`, `Bitfinex`, `Poloniex`, `Bitstamp`, and `Binance` and `Kraken` to calculate an aggregate price index.

For example we can inspect the last 4 rows of the dataframe using the `tail()` method

<img width="483" alt="Screen Shot 2019-08-02 at 11 00 15 AM" src="https://user-images.githubusercontent.com/39813026/62380064-5d419d80-b516-11e9-8f09-dee21f5f825f.png">


`Close` - reversion benchmark

- Reversion metrics give us an indication of our temporary impact

`Open` - momentum benchmark

`Previous Close` - momentum benchmark

- Momentum metrics give us an indication of the direction of the price drift 

### Two Separate Methods

Define single collateral type:

```
def get_single_collateral(self, symbol):
        #dates
        start = self.start 
        end = self.end 
        
        prices = ETH['Close']
        returns = prices.pct_change()
        
        self.returns = returns
        self.prices = prices
```

Define multiple collateral types:

```
def get_multiple_collateral(self, symbols, weights):
        start = self.start
        end = self.end
        
        #Get exchange price data
        df = ETH['Close']
       
        #percent Change
        returns = df.pct_change()
        returns += 1
        
        #define allocation of each asset 
        portfolio_value = returns * weights
        portfolio_value['Portfolio Value'] = portfolio_value.sum(axis=1)
        
        #Portfolio prices
        prices = portfolio_value['Portfolio Value']
        
        #portfolio returns 
        returns = portfolio_value['Portfolio Value'].pct_change()
        returns = returns.replace([np.inf, -np.inf], np.nan)
                
        self.returns = returns
        self.prices = prices

```

### Checking out momentum:

```
#Moving averages are a great way to determine the momentum of the cryptocurrency
ma_days = [10,20,50]
for ma in ma_days:
    column_name = "MA %s days" %(str(ma))
    ETH[column_name] = ETH['Adj Close'].rolling(window=ma,center=False).mean()
ETH[['Adj Close','MA 10 days','MA 20 days','MA 50 days']].plot(legend=True)

```

### Moving average representation:



Moving average representation expresses any time series Yt as:

<img width="143" alt="Screen Shot 2019-08-02 at 1 17 27 PM" src="https://user-images.githubusercontent.com/39813026/62387176-e8775f00-b527-11e9-8328-eec62b4acbc0.png">

ETH output:

<img width="418" alt="Screen Shot 2019-08-02 at 11 23 36 AM" src="https://user-images.githubusercontent.com/39813026/62380820-0dfc6c80-b518-11e9-891f-5c00d9270726.png">

Once we have historical returns, we can gauge their relative dispersion with a measure called `variance`

Now we will calculate volatility and historical time-to-revert for collateralization ratio

Volatility: is calculated as the standard deviation of returns 

### Close to Close Volatility Method:

<img width="137" alt="Screen Shot 2019-08-02 at 1 16 45 PM" src="https://user-images.githubusercontent.com/39813026/62387138-d5648f00-b527-11e9-8ce7-e90d5e677ffc.png">

```
def close_to_close_volatility(close_ret, mean_ret): 
    return np.sqrt(np.sum((close_ret-mean_ret)**2)/390)

```

### OHLC Volatility Estimation Method:

<img width="309" alt="Screen Shot 2019-08-02 at 1 20 15 PM" src="https://user-images.githubusercontent.com/39813026/62387334-4efc7d00-b528-11e9-94cf-1b03e8a75e15.png">


```
def crypto_volatility(open, high, low, close, close_tm_): 
    return np.log(open/close_tm1)**2 + 0.5*(np.log(high/low)**2) \
        - (2*np.log(2)-1)*(np.log(close/open)**2)

```

### CDP States:

- `bite` = debt-tranche that has been liquidated
- `wipe/shut` = debt-tranche that has been voluntarily closed by owner
- `unsafe` = CDP is below 150% collateralization ratio
- `safe` = CDP is not below 150% collateralization ratio
- `open` = CDP is safe and is still open

The time to revert for collateralization ratio looks at the CDP states and every state in the sequence has a timestamp (number of seconds the CDP stayed in that state)

1. Calculate Transition state (infinitesimal generator matrix or intensity matrix) 
2. Look at the sequence distribution
3. Answer: What fraction of all dai-time is spent in state ‘m’ before moving to state ‘k’?
4. A matrix of the number of transition states are produced 
5. The next component of the code multiplies 2 matrices to get ‘draw_time’ in seconds 
6. The next component categories into safe and unsafe 
7. A final matrix is produced 

```
sequesnces_with_next_state = utils.dataframe_to_sequences_with_end_state(df) 
time_spent_matrix = utils.time_spent_before_state_change_distribution(sequence_with_next_state)
time_spent_matrix

```

### Geometric Brownian Motion data: Volatility modeled in GBM function

Geometric Brownian Motion assumes that a constant drift is accompanied by random shocks and the period returns are normally distributed under GBM, the consequent multi-period price levels are lognormally distributed 

Brownian motion models the random behavior of our collateral type over time 


```
logarithmic_return = np.log(ETH.close)-np.log(close.shift(1))
mean_return = np.mean(returns)
volatility = returns.std()
```

GBM is composed of `drift` and `shock`

<img width="208" alt="Screen Shot 2019-08-02 at 1 31 36 PM" src="https://user-images.githubusercontent.com/39813026/62387915-f4fcb700-b529-11e9-9064-231046efa2e5.png">

Where:

<img width="335" alt="Screen Shot 2019-08-02 at 1 34 35 PM" src="https://user-images.githubusercontent.com/39813026/62388175-9f74da00-b52a-11e9-9900-7e3b102412a9.png">

`drift` <img width="43" alt="Screen Shot 2019-08-02 at 1 38 48 PM" src="https://user-images.githubusercontent.com/39813026/62388299-ed89dd80-b52a-11e9-8c7e-3b63ea97f475.png"> a long term trend in the positive or negative direction; derived from collaterals historical performance
`shock` <img width="59" alt="Screen Shot 2019-08-02 at 1 40 32 PM" src="https://user-images.githubusercontent.com/39813026/62388363-2033d600-b52b-11e9-9cf9-1649ca9616a5.png"> added or subtracted to the drift; function of the collaterals standard deviation 

For each timestep, our model assumes the price will ‘drift’ up by the expected return 

The drift will be ‘shocked’ (+/-) by a random shock and the random shock will be the standard deviation multiplied by a random number; this is a way of scaling the standard deviation 

Pseudocode for GBM of ETH:

<img width="307" alt="Screen Shot 2019-08-02 at 1 42 29 PM" src="https://user-images.githubusercontent.com/39813026/62388469-6721cb80-b52b-11e9-97b8-aba85f39d6f1.png">

### GBM without jump diffusion parameters:

`Xo`     initial collateral type price
`mu`     returns (drift coefficient)
`sigma`  volatility (diffusion coefficient)
`W`      brownian motion
`T`      time period
`N`      number of increments


```
def GBM(Xo, mu, sigma, W, T, N):    
    t = np.linspace(0.,1.,N+1)
    X = []
    X.append(Xo)
    for i in xrange(1,int(N+1)):
        drift = (mu - 0.5 * sigma**2) * t[i]
        diffusion = sigma * W[i-1]
        X_temp = Xo*np.exp(drift + diffusion)
        X.append(X_temp)
    return X, t
```

### To capture drift and diffusion:

```
def daily_return(adj_close):
    returns = []
    for i in xrange(0, len(adj_close)-1):
        today = adj_close[i+1]
        yesterday = adj_close[i]
        daily_return = (today - yesterday)/yesterday
        returns.append(daily_return)
    return returns

returns = daily_return(adj_close)
```

### We use these simulations and visualize them:


```
while (simulation < number_simulations):
    #list for each new simulation 
    prices = [] 
    #reset the counter 
    days = 0 
    #input initial price 
    prices.append(last_price) 
    time_step = 1

    while (days < predicted_days):
        drift = (mean_return - volatility**2/2)*time_step
        shock = volatility * np.random.normal() * time_step**0.5
        price = prices[days] * np.exp(drift+shock)
        prices.append(price) 
        #increment days counter 
        days = days + 1
    
    results[str(simulation)] = pd.Series(prices).values
    #increment simulation counter 
    simulation = simulation + 1
```


## Behavioral Data 
Analyzing CDP behavior and jump diffusion 

To tune our model we are going to want to input: `volatility`, `probability_of_jump`, and `mean_jump_size`.

In jump diffusion the idea is that price movements underlie sudden changes:

<img width="180" alt="Screen Shot 2019-08-02 at 1 49 00 PM" src="https://user-images.githubusercontent.com/39813026/62388833-5a51a780-b52c-11e9-99ba-ce19cacf5c06.png">

Jump diffusion will consider downtrends and captures the true picture of the collateral behavior:

<img width="445" alt="Screen Shot 2019-08-02 at 1 50 25 PM" src="https://user-images.githubusercontent.com/39813026/62388911-84a36500-b52c-11e9-9859-dff0578d60b6.png">

We can calculate <img width="56" alt="Screen Shot 2019-08-02 at 1 51 22 PM" src="https://user-images.githubusercontent.com/39813026/62388976-a4d32400-b52c-11e9-8c62-a6defe1ae942.png"> by:

```
sigma*np.randn()+mu
```

We can then calculate the jump size J by:

```
def jump(probability, mean_size, volatility):
    jump_size = mean_size + volatility*np.random.randn()
    return decision(probability)*jump_size
```

After, we are going to want to map the instance of the event under consideration of a probability using:


```
def decision_collateral(probability):
   return int(np.random.random() < probability)
```

Taking the input data:
- `date`
- `collateral_type`
- `last_price`
- `volatility`
- `simulated_time_window`
- `Time_window`




```
results = pd.DataFrame()

simulation = 0
while (simulation < number_simulations):
    prices = []
    days = 0 
    prices.append(last_price) 
    time_step = 1
    while (days < predicted_days):
        drift = (mean_return - volatility**2/2)*time_step
        shock = volatility * np.random.normal() * time_step**0.5
        c_jump = jump(probability, mu, sigma)
        price = prices[days] * np.exp(drift+shock)
        price = price + c_jump
        prices.append(price)
        days = days + 1
    if(prices[-1] > 0):
        results[str(simulation)] = pd.Series(prices).values 
    simulation = simulation + 1
```


### How we model GBM and Collateralization Ratio


We take this GBM with jump diffusion and the inertia reversion function to fluctuating collateralization ratio come in at ETH price path based on GBM with jump diffusion

ETH price path over time and second pass reads each day movement of ETH prices, ETH price moved x% and I am going to change collateralization ratio by y% and y is a function of x and the previous collateralization ratio and time 

Evaluate liquidity by analyzing:
- % of day's volume
- intraday volume curve
- cumulative intraday volume curve, etc.

Traditionally, liquidity is highest as we approach the close and second highest at open. 

### Parameters for ‘expected amount of loss’:
crashes that are atypical;outright collateralization or significant enough sale through liquidation 

The Value at Risk (VaR) for coverage  is the maximum amount we could expect to lose with likelihood <img width="69" alt="Screen Shot 2019-08-02 at 1 59 23 PM" src="https://user-images.githubusercontent.com/39813026/62389434-d1d40680-b52d-11e9-860a-a9d5d807eeed.png">

- VaR is very similar to confidence intervals
- Conditional Value at Risk takes into account the shape of the returns distribution 

### Defining CVaR:

```
def cvar(invested, returns, weights, alpha=0.95, lookback=500):
    var = value_at_risk(invested, returns, weights, alpha, lookback=lookback)
    returns = returns.fillna(0.0)
    portfolio_returns = returns.iloc[-lookback:].dot(weights)
    
    # Get back to a return
    var_pct_loss = var / invested
    
    return invested * np.nanmean(portfolio_returns[portfolio_returns < var_pct_loss])
```

- CVaR captures the moments of the distribution; if the tails have more mass CVaR will capture this 
- Following calculating CVaR check for convergence

### Tail Risk:

- CVaR will take care of tail risk or fat tailedness 
- Tail risk: autoregressive processes tend to have more extreme values than data taken from a normal distribution and this is  because the value at each time point is influenced by recent values
- Fat tail distribution: if the series randomly jumps up, it is more likely to stay up than a non-autoregressive series, the extremes on the pdf will be fatter than the normal distribution 


```
def calc_unadjusted_interval(X):
    T = len(X)
    mu = np.mean(X)
    sigma = np.std(X)
    lower = mu - 1.96 * (sigma/np.sqrt(T))
    upper = mu + 1.96 * (sigma/np.sqrt(T))
    return lower, upper
```

## Statistical Rigor
Checking out visualizations of your data is not enough; we need a stronger degree of statistical rigor ie confidence intervals, pacf, etc. 

## Terms:

- `Volatility` is calculated as the standard deviation of returns 
- `Slippage` is when the price 'slips' before the trade is fully executed, leading to the fill price being different from the price at the time of the order. Slippage is where our backtester calculates the realistic impact that your orders have on the execution price we receive. 
 - The attributes that have the most influence on slippage are:
 - `Volatility`
 - `Liquidity`
 - `Relative order size`
 - `Bid / Ask spread`



## ✍️ Authors <a name = "authors"></a>
- [Madison Piercy](https://github.com/madison1111/)
- [Nick Allen](https://github.com/atnickallen/)
