<p align="center">
  <a href="" rel="noopener">
 <img src="https://github.com/atnickallen/MCD_Doc/blob/master/mcd%20backend%20-%20tool3.png"></a>
</p>
<h3 align="center">MakerDAO MCD Risk Tool</h3>

<div align="center">
 
  [![Status](https://img.shields.io/badge/status-active-success.svg)]() 
  [![GitHub Issues](https://img.shields.io/github/issues/kylelobo/The-Documentation-Compendium.svg)](https://github.com/kylelobo/The-Documentation-Compendium/issues)
  [![GitHub Pull Requests](https://img.shields.io/github/issues-pr/kylelobo/The-Documentation-Compendium.svg)](https://github.com/kylelobo/The-Documentation-Compendium/pulls)
  [![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE.md)

</div>

---

<p align="center"> Analyzing different collateral type risk with models
    <br> 
</p>

## 🗄 Table of Contents
- [The Goal](#problem_statement)
- [Setup](#idea)
- [Downloading The Prerequisites](#limitations)
- [Cloning The Repo](#future_scope)
- [Loading the Models](#getting_started)
- [For Analysis](#usage)
- [Running the Models](#tech_stack)
- [MCD Risk Model 1](../CONTRIBUTING.md)
- [Authors](#authors)
- [Acknowledgments](#acknowledgments)

## Goal of the Risk Tool: <a name = "Goal"></a>
Goal of the tool is to display risk and allow users to double click on visually evaluating risk.

## 💽 Setup: <a name = "setup"></a>
If you would like to install and use this tool on your own machine make sure you have `pipenv` installed. If you need to install follow [these simple instructions](https://github.com/pypa/pipenv#installation).

### Downloading The Prerequisites

The packages needed to run this tool: `pipenv`, `homebrew`, `python3`, `pandas`, and `numpy`.

The packages `pandas` and `numpy` will need to be installed in order for these repositories to work. If you do not have them installed, run these commands in your terminal:

```
pipenv install pandas
```

```
pipenv install numpy
```


### Cloning The Repo

Clone the MCD Risk Model Repo:

```
git clone https://github.com/madison1111/mcd_risk_model.git
```

Then cd into the directory:

```
cd mcd_risk_model
```

### Loading the Models
To reproduce results run the following commands:

```
python3 mcd_risk_model_1.py
```

```
python3 mcd_risk_model_2.py
```

### For Analysis
Start your Jupyter notebook:

```
jupyter notebook
```

If your browser doesn't automatically open, then go to:

```
http://localhost:8888
```

## 🎛 Running the Models
Notebooks can be found in the `notebooks` folder

### MCD Risk Model 1:

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

We pull data from 5 major exchanges: `Coinbase`, `Bitfinex`, `Poloniex`, `another`, and `another` to calculate an aggregate price index.

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

Checking out momentum:

```
#Moving averages are a great way to determine the momentum of the cryptocurrency
ma_days = [10,20,50]
for ma in ma_days:
    column_name = "MA %s days" %(str(ma))
    ETH[column_name] = ETH['Adj Close'].rolling(window=ma,center=False).mean()
ETH[['Adj Close','MA 10 days','MA 20 days','MA 50 days']].plot(legend=True)

```

### Moving average representation:



Moving average representation expresses any time series Ytas:

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



Once the ideal, reality, and consequences sections have been 
completed, and understood, it becomes easier to provide a solution for solving the problem.

## ⛓️ Dependencies / Limitations <a name = "limitations"></a>
- What are the dependencies of your project?
- Describe each limitation in detailed but concise terms
- Explain why each limitation exists
- Provide the reasons why each limitation could not be overcome using the method(s) chosen to acquire.
- Assess the impact of each limitation in relation to the overall findings and conclusions of your project, and if 
appropriate, describe how these limitations could point to the need for further research.

## 🚀 Future Scope <a name = "future_scope"></a>
Write about what you could not develop during the course of the Hackathon; and about what your project can achieve 
in the future.

## 🏁 Getting Started <a name = "getting_started"></a>
These instructions will get you a copy of the project up and running on your local machine for development 
and testing purposes. See [deployment](#deployment) for notes on how to deploy the project on a live system.

### Prerequisites

What things you need to install the software and how to install them.

```
Give examples
```

### Installing

A step by step series of examples that tell you how to get a development env running.

Say what the step will be

```
Give the example
```

And repeat

```
until finished
```

## 🎈 Usage <a name="usage"></a>
Add notes about how to use the system.

## ⛏️ Built With <a name = "tech_stack"></a>
- [MongoDB](https://www.mongodb.com/) - Database
- [Express](https://expressjs.com/) - Server Framework
- [VueJs](https://vuejs.org/) - Web Framework
- [NodeJs](https://nodejs.org/en/) - Server Environment

## ✍️ Authors <a name = "authors"></a>
- [@kylelobo](https://github.com/kylelobo) - Idea & Initial work

See also the list of [contributors](https://github.com/kylelobo/The-Documentation-Compendium/contributors) 
who participated in this project.

## 🎉 Acknowledgments <a name = "acknowledgments"></a>
- Hat tip to anyone whose code was used
- Inspiration
- References
