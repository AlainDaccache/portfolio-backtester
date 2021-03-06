# portfolio-backtester

A command-line script I made to help me with making decisions with regards to choosing stocks in the stock market. To oversimplify it, it's a portfolio backtester; i.e. given a portfolio, it'll tell you how that portfolio would've done in the past.

> NOTE: I've included definitions and links for some words at the bottom, since stock terminology is used in describing some functionality. Words with definitions at the bottom are in blue and clickable. If you mousover, a short summary should appear, but you can click to navigate to the actual, longer definitions

Main features:
- Download day-by-day stock data from Google using Downloader.py
- Draw a price history chart using downloaded data
- Calculate and overlay [indicators](https://github.com/DmitryGubanov/portfolio-backtester/tree/v3.0-basic-timing-strategies#indicator "tools used to analyze trends and patterns") on the chart. Implemented indicators: [SMA](https://github.com/DmitryGubanov/portfolio-backtester/tree/v3.0-basic-timing-strategies#sma-simple-moving-average "Simple Moving Average, average stock price for last N days"), [EMA](https://github.com/DmitryGubanov/portfolio-backtester/tree/v3.0-basic-timing-strategies#ema-exponential-moving-average "Exponential Moving Average, like SMA, but the prices used in the average are given exponentially decreasing weightings going backwards"), [MACD](https://github.com/DmitryGubanov/portfolio-backtester/tree/v3.0-basic-timing-strategies#macd-moving-average-convergence-divergence "Moving Average Convergence Divergence, a set of values which seek to quantify momentum")
- Simulate past performance on a day-by-day basis for a portfolio of stocks
- Supports periodic [rebalancing](https://github.com/DmitryGubanov/portfolio-backtester/tree/v3.0-basic-timing-strategies#rebalance "Restoring the original weights for the assets in your portfolio") and contributing
- Specify conditional ratios for assets, which could depend on some relationship between stock price and/or indicators (e.g. buy stock X when it's below SMA_50, sell when it's above SMA_10)
- Summarize portfolio performance with commonly used statistics. Implemented statistics: final value, number of trades made, [(Adjusted) CAGR](https://github.com/DmitryGubanov/portfolio-backtester/tree/v3.0-basic-timing-strategies#adjusted-cagr-compound-annual-growth-rate "Adjusted Compound Annual Growth Rate, your average yearly returns"), [Sharpe Ratio](https://github.com/DmitryGubanov/portfolio-backtester/tree/v3.0-basic-timing-strategies#sharpe-ratio "A ratio quantifying how many returns you make per unit of risk; higher is better"), [Sortino Ratio](https://github.com/DmitryGubanov/portfolio-backtester/tree/v3.0-basic-timing-strategies#sortino-ratio "Similar to Sharpe, but this ratio only factors in negative volatility"), best year, worst year, maximum [drawdown](https://github.com/DmitryGubanov/portfolio-backtester/tree/v3.0-basic-timing-strategies#drawdown "A percent value representing a change from a peak to a valley, i.e. max drawdown is the maximum loss incurred along the way") and time taken to recover from it.
- Show portfolio status over time by charting some statistics. Implemented charted statistics: portfolio value history, asset allocation/ratios over time, annual returns, contributions vs growth over time.

Experimental features:
- Generating data for one stock based on data of another stock. Example: stock A is correlated to stock B, but stock A only has data back to 2009, while stock B has data going back to 1990. You can use this data generation to generate data for stock A back to 1990 based on stock B. Intended for use on [leveraged ETFs](https://github.com/DmitryGubanov/portfolio-backtester/tree/v3.0-basic-timing-strategies#leveraged-etf "To oversimplify, a stock which seeks to multiply the returns of another stock by a factor").

# 0. Table of contents

[1. Prerequisites](https://github.com/DmitryGubanov/portfolio-backtester/tree/v3.0-basic-timing-strategies#1-prerequisites)

[2. Sample usage](https://github.com/DmitryGubanov/portfolio-backtester/tree/v3.0-basic-timing-strategies#2-sample-usage)

[3. Advanced usage](https://github.com/DmitryGubanov/portfolio-backtester/tree/v3.0-basic-timing-strategies#3-advanced-usage)

[4. Current work in progress](https://github.com/DmitryGubanov/portfolio-backtester/tree/v3.0-basic-timing-strategies#4-current-work-in-progress)

[5. Changelog](https://github.com/DmitryGubanov/portfolio-backtester/tree/v3.0-basic-timing-strategies#5-version-featureschangelog)

[6. Definitions](https://github.com/DmitryGubanov/portfolio-backtester/tree/v3.0-basic-timing-strategies#6-definitions)

# 1. Prerequisites

This program was written and tested in Python 3.5.2 (https://www.python.org/downloads/release/python-352/). Use a different version at your own discretion.

Graphing requires matplotlib.
```
$ pip install matplotlib
```
> NOTE: ensure that the pip you use is installed under python 3.5 with 'pip -V'

Finally, you're probably going to want to clone this repo.

# 2. Sample usage

This will act as an example of how this program can be used to tweak a common portfolio strategy for more desirable performance. I provided some sample strategy files in the repo which we'll use.

### 2.0: Downloading the data

Download stock data for the stocks/funds with tickers SPY and TLT.
```
$ python3.5 Downloader.py --download SPY TLT
```
> NOTE: For the curious, SPY follows the S&P500 index (the stock market as a whole) while TLT follows the long-term treasury bond index (the apparent value of stable and relatively low risk investments). You invest in the stock market for growth purposes, but when the stock market is doing poorly, the viablility of more stable investments rises since they aren't as exposed to poor market conditions. In short, when the stock market is down, there is a better than random chance the bond index is up. As a result, the two are somewhat inversely correlated which makes bonds a 'natural' hedge (something you use to mitigate losses) for stocks.

### 2.1: Testing standard strategy (our benchmark)

Let's see where simply investing 10,000 in the stock market gets us:
> NOTE: If you pay attention to the command, you'll notice '--strategy stocks-only'. stocks-only is a sample strategy file I provided - we'll be using three different ones.

> ANOTHER NOTE: Your outputs will differ from mine, since time has passed and the stocks we're using in this example are real stocks which change price over time

```
$ python3.5 folio.py --portfolio 10000 --strategy stocks-only

##################################
# PERFORMANCE SUMMARY
##################################
initial: $10000.00
final:   $28673.68
trades:  1
---------------------------
Sharpe Ratio:  0.13015961697451908
Sortino Ratio: 0.2555517731631023
---------------------------
CAGR:          7.22%
Adjusted CAGR: 7.22%
---------------------------
best year:  25.13%
worst year: -35.71%
---------------------------
max drawdown: -56.26%
  between 2007-10-10 and 2009-03-09, recovered by 2013-03-14

```

Charts:
- first chart: portfolio value vs time
- second chart: asset allocation vs time (in this case, we're 100% in stocks the whole time)
- third chart: annual returns
- fourth chart: contributions and growth vs time (we start at 100% contributions and as time goes on, we grow as our assets grow - notice in 2009 we have more contributions than actual portfolio value, i.e. we've lost money overall)

<img src="http://i.imgur.com/NuzTFZ0.png" alt="chart" />

So on average we get 7.2% a year, but we would have had to weather a 56% drop during the 2008 recession (yikes).

### 2.2: Introduce bonds

Let's try to add bonds, a 'natural' hedge to stocks, to try and mitigate some of those losses.
```
$ python3.5 folio.py --portfolio 10000 --strategy stocks-and-bonds

##################################
# PERFORMANCE SUMMARY
##################################
initial: $10000.00
final:   $23500.12
trades:  2
---------------------------
Sharpe Ratio:  0.15580397381790653
Sortino Ratio: 0.29533912934209955
---------------------------
CAGR:          5.82%
Adjusted CAGR: 5.82%
---------------------------
best year:  15.67%
worst year: -17.55%
---------------------------
max drawdown: -35.72%
  between 2007-10-10 and 2009-03-09, recovered by 2012-02-24
```
<img src="http://i.imgur.com/5zhQrJv.png" alt="chart" />

By introducing bonds, we've cut down our risk by ~40% at the cost of ~20% of our gains. As a result, the [Sharpe](https://github.com/DmitryGubanov/portfolio-backtester/tree/v3.0-basic-timing-strategies#sharpe-ratio "A ratio quantifying how many returns you make per unit of risk; higher is better") and [Sortino](https://github.com/DmitryGubanov/portfolio-backtester/tree/v3.0-basic-timing-strategies#sortino-ratio "Similar to Sharpe, but this ratio only factors in negative volatility") ratios are both higher.

From the graphs, we can see our asset allocations have veered away from what we set intially (0.6 and 0.4, check the sample files).

### 2.3: Maintain ratios by rebalancing

Let's rebalance quarterly to maintain our desired ratios of 60% SPY and 40% TLT, as defined by our strategy file.

```
$ python3.5 folio.py --portfolio 10000 --strategy stocks-and-bonds --rebalance q

##################################
# PERFORMANCE SUMMARY
##################################
initial: $10000.00
final:   $25402.22
trades:  114
---------------------------
Sharpe Ratio:  0.1765820993420145
Sortino Ratio: 0.3315811452594389
---------------------------
CAGR:          6.36%
Adjusted CAGR: 6.36%
---------------------------
best year:  16.98%
worst year: -14.45%
---------------------------
max drawdown: -33.09%
  between 2007-10-30 and 2009-03-09, recovered by 2011-05-31
```
<img src="http://i.imgur.com/IIoIR5E.png" alt="chart" />

With our ratios maintained throughout the life of our portfolio, we've regained some of those lost gains and actually lost even more risk. You'll notice the Sharpe and Sortino ratios have once again increased.

### 2.4: Experiment with timing

Let's try a timing strategy based on the Simple Moving Average indicator. In this case we'll use the SMA 100, a fairly long term indicator. In short, we'll sell when there's a sharp enough negative movement to break a positive 100-day trend, but buy it back when it recovers above that trend. Theoretically, this is to avoid big negative movements; realistically, we'll see:

```
$ python3.5 folio.py --portfolio 10000 --strategy stocks-and-bonds --rebalance q

##################################
# PERFORMANCE SUMMARY
##################################
initial: $10000.00
final:   $20147.15
trades:  317
---------------------------
Sharpe Ratio:  0.16891167389583966
Sortino Ratio: 0.393798491370434
---------------------------
CAGR:          4.74%
Adjusted CAGR: 4.74%
---------------------------
best year:  11.59%
worst year: -8.71%
---------------------------
max drawdown: -12.76%
  between 2007-06-05 and 2009-03-09, recovered by 2009-09-16
```
<img src="http://i.imgur.com/Aq37jCM.png" alt="chart" />

First thing to notice is the asset allocations. The blue one (SPY) is bouncing between 0.6 and 0.2, because our strategy sells 40% below the SMA 100, and buys it back when it comes back above it.

Before moving on, it might help to visualize this:
```
$ python3.5 folio.py --draw SPY --indicators SMA_100
```

From our original, we've lost ~35% of our gains, but we've also lost ~80% of our risk. In fact, this is not immediately obvious, but the Sharpe and Sortinio ratios indicate this strategy sacrifices some upward movement to avoid a lot of downward movement. We're also making ~317 trades over the course of 15 years, which is a lot more than the original of 1 trade, but that comes out to about 20 trades a year, which really isn't that much.

### 2.5 Conclusion

I knew these tweaks would have these results ahead of time, so it's entirely possible to get worse results from your tweaks. However, the point is this program makes it fairly easy to play around with various strategies to see how they would perform in the market conditions of the past.

# 3. Advanced usage

This section is for using some of the more advanced features.

### 3.0 Advanced features

> Disclaimer: These are advanced features of a stock program, so they do require a bit more knowledge and a more involved perspective on the stock market to be understood. I'll try my best to convey the motivations, functionality, and results as succinctly as possible.

[3.1 Generating data](https://github.com/DmitryGubanov/portfolio-backtester/tree/v3.0-basic-timing-strategies#31-generating-data)

[3.2 Adjusting/creating  strategies](https://github.com/DmitryGubanov/portfolio-backtester/tree/v3.0-basic-timing-strategies#32-adjusting-timing-strategies)

### 3.1 Generating data

This demonstration will use two tickers, SPY and UPRO. UPRO tries to multiply the returns of SPY by 3, which simply means if SPY moves 1%, UPRO tries to move 3%. This seems attractive:

```
$ python3.5 Downloader.py --download SPY UPRO
$ python3.5 folio.py --draw UPRO
```

<img src="http://i.imgur.com/FmXWZ7L.png" alt="chart" />

That's more than a 10x increase in value over the last ~7 years. This is all the data there is on UPRO, so all existing data suggests that this is a good investment. However, UPRO was conveniently started after the recession in 2008, so there exists no data on how it would have performed during that time. Let's find out (theoretically speaking) by generating UPRO based on SPY.

This generation is using existing UPRO and SPY data to build a relationship between the two, then using that relationship to generate the part of UPRO that doesn't exist where SPY does exist. Luckily SPY goes back all the way to the 1990s, so we can generate UPRO that far.

```
$ python3.5 folio.py --draw UPRO --use-generated UPRO SPY
```

> NOTE: Notice the added --use-generated argument on the command-line. --use-generated simply bypasses the original data source for any feature, and replaces it with the generated data.

<img src="http://i.imgur.com/W2hU8rL.png" alt="chart" />

Anyway, we can see that UPRO dropped quite a bit (~95%) during the recession and during the dot-com crash. With this new information, it's unlikely that many would feel comfortable investing in something that lost over 90% of its value on two occasions in the last 20 years.

Let's build a portfolio using UPRO the same way we did with SPY in section 2.1.

```
python3.5 folio.py --portfolio 10000 --strategy upro-only --use-generated UPRO SPY

##################################
# PERFORMANCE SUMMARY
##################################
initial: $10000.00
final:   $503322.53
trades:  1
---------------------------
Sharpe Ratio:  0.16743168735558275
Sortino Ratio: 0.3561463844157349
---------------------------
CAGR:          17.26%
Adjusted CAGR: 17.26%
---------------------------
best year:  146.13%
worst year: -85.67%
---------------------------
max drawdown: -96.18%
  between 2000-03-27 and 2009-03-09, recovered by 2016-12-07
```

Although the yearly returns look good, starting in 2000 you would have lost money until you lost 96% and would only recover by the end of 2016. To most, this would be a deal-breaker, which is why I consider this feature handy in testing leveraged ETFs in situations to which they've not been exposed.

This is an experimental feature, in that there is no way to verify its accuracy. However, I've used this method to generate data that does exist (for verification purposes; in any other case, I wouldn't need to generate data which already exists) and it was pretty accurate.

Using the standalone generate functionality, you can compare generated data against real data:

```
$ python3.5 folio.py --generate UPRO SPY
```

<img src="http://i.imgur.com/NNSRDGc.png" alt="chart" />

At the top we see the real vs the generated, at the bottom we see the generated and what the generated is generated from.

### 3.2 Adjusting timing strategies

In each example so far, there have been strategy files used. They're in CSV format and have four columns: weight, ticker, buy signal, sell signal. Here's 'stocks-only':

```
$ cat stocks-only

1.0,SPY,ALWAYS,NEVER
0.0,TLT,ALWAYS,NEVER
```

The weight is the portion of the portfolio dedicated to that asset or position. In this case, 1.0 (or 100%) SPY and 0.0 (or 0%) TLT. The 0.0 line isn't necessary, but it's there for consistency between strategies.

A signal is like a raised flag, if the buy/sell signal is satisfied, the strategy says to buy/sell that portion of the portfolio. In this case, the buy signals are ALWAYS (always buy this position) and sell signals are NEVER (never sell this position). This just represents a buy-and-hold portfolio.

Here's 'stocks-and-bonds-timing':

```
$ cat stocks-and-bonds-timing

0.2,SPY,ALWAYS,NEVER
0.4,SPY,SPY~PRICE > SPY~SMA_100,SPY~PRICE < SPY~SMA_100
0.4,TLT,ALWAYS,NEVER
```

The buy and sell signals for the second portion is more involved now, but it's simply saying buy when SPY's price is above SPY's SMA_100 and sell when the opposite happens. Without using any fancy regex, the pattern is basically:

```
<ticker>~<indicator> <relation> <ticker>~<indicator>.
```

Ticker can be any real ticker for which you have data.

Indicator currently has to be one of the following, where X, Y, Z are positive integers:
- PRICE
- SMA_X
- EMA_X
- MACD_X-Y-Z
- MACDSIGNAL_X-Y-Z

Relation is either < or >


# 4. Current work in progress

### 4.0 Short-term (v3.0, trades based on indicators):

x create shell for Brain class, a class dedicated to making decisions based on strategies  
x hardcode a basic strategy into Brain (assesses market daily, provides shares to Trader)  
x probably need to refactor Trader by moving rebalancing into Brain  
x program Brain to handle strategies based on different indicators and periods  
x implement a way to read strategies from file in DataManager  
x implement Sharpe and Sortino ratios  
x implement previous high as indicator  
o add some sort of tolerance/adjustments to previous high to not make it useless for years after crashes (need to brainstorm)  
x initialize both ratios and shares in Brain to 0 for all assets before anything runs  
o dynamic/adjusted buy and sell signals (keyword -> filled in during simulation)  
o buy and sell signals with ANDs and ORs  
o relative strength index  
o identify peaks and valleys (draw functionality for now)  
o identify support and resistance lines (draw functionality for now)  
o logarithmic charts or daily returns instead of daily prices  
o chart pattern: head and shoulders  
o chart pattern: double top, double bottom  

### 4.1 Long-term:

o interface (e.g. web)  
o dynamic portfolio ratios depending on conditions  
o benchmarks  
o reimplement withdrawals   
o gather very short term data (minutely or less) (possibly other program)

# 5. Version features/changelog

Current version: 3.0  
WIP: 3.0

## Version 1

> Goals: get data, store data, project data, show data

#### v1.0, basic data
- download stock data given ticker
- download stock(s) data from list of stocks in file
- read CSV file with stock data and convert to arrays
- graph stock data using pyplot

#### v1.1, basic indicators
- implement some indicators (sma, ema, macd) with custom date ranges
- display indicators using pyplot

#### v1.2, playing around with data
- calculate growth of all stocks in a file
- specify time period for analysis
- implement some utils to make analysis consistent (e.g. date math, nearest date before/after given date)

#### v1.3, ETF data generation
- given two tickers, create relationship between the two and extrapolate data for one based on data in other (e.g. UPRO is 3x the S&P500, read S&P before UPRO's inception to calculate what UPRO would have been had it existed before its inception)
- tweak data generation to improve accuracy
- test generation by generating existing data and comparing

#### v1.4, cleanup
- move repeated code into functions
- rewrite some functions to be more legible and have clearer logic flow


## Version 2

> Goals: simulate a basic portfolio, create framework-esque platform

#### v2.0, basic portfolio
- create portfolio class, which has cash, holdings, and assets
- create portfolio behaviour (buy, sell, short, cover)

#### v2.1, basic market
- create market class, which has a date and stocks
- create market behaviour (query stocks on date, advance date, add stocks, inject data)

#### v2.2, basic simulation
- create simulator class, which has portfolio, market, and start/end date(s)
- create simulator simulation behaviour

#### v2.3, simulation features
- add contributions and rebalancing of portfolio holdings to simulator
- add optional commission costs
- add portfolio statistics for graphing purposes (portfolio value, asset allocation, annual return, contribution vs growth)

#### v2.4, validation, cleanup, and fixes
- separate download logic into own Downloader class
- implement downloading from google, since yahoo stopped their free/easy to use service
- separated all classes into own files and put all util classes/functions into own file
- implement Trader class for trading logic
- implement DataManager class for managing data on disk
- implement Monitor class for statistics and record keeping during simulations
- implement Calculator class for stand-alone calculations outside simulations
- rewrote all files to follow PEP-8 and Google docstrings coding style


## Version 3

> Goals: more intricate user programmed strategies

#### v3.0, basic indicator related strategies
- implement Brain class, where all decision making will happen
- Trader now has a Brain, but otherwise only executes trades based on what Brain has decided (i.e. Brain calculates needed shares, Trader then references needed shares and executes trades so their Portfolio matches said shares)
- implement custom strategies read from file (all needed data is automatically extracted from the strategies file so only the files need to be changed to test a new strategy)
- Sharpe and Sortino ratios implemented (helps compare strategy effectiveness)
- separated MACD into two indicators: MACD and MACDSIGNAL

# 6. Definitions

> NOTE: Some definitions have been pulled from or influenced by Investopedia. Terminology is also simplified to avoid using undefined terms in definitions.

#### Indicator
Indicators are statistics used to measure current conditions as well as to forecast financial or economic trends.

http://www.investopedia.com/terms/i/indicator.asp


#### SMA (Simple Moving Average)
Always has a period (number of days, X) associated with it. The average price for a stock over the last X days. Typically used to quantify trends.

http://www.investopedia.com/terms/s/sma.asp


#### EMA (Exponential Moving Average)
Always has a period (number of days, X) associated with it. Similar to the SMA, but the weight given to each price goes down exponentially as you go backwards in time. Whereas in a SMA, equal weight is given to each day.

http://www.investopedia.com/terms/e/ema.asp


#### MACD (Moving Average Convergence Divergence)
Typically has three periods (number of days, X, Y, Z) associated with it. The standard periods are 12, 26, 9, but these can be changed. The math is too complicated for this definition, but in general, it tries to quantify the momentum of a stock, rather than the trend, by subtracting a long-term trend from a short-term trend (in an attempt to see the 'net' trend).

http://www.investopedia.com/terms/m/macd.asp


#### Rebalance
When you build a portfolio of assets, a standard strategy is to specify weights for each asset (e.g. if you have 4 assets, you might give each a weight of 25% in your portfolio). However, over time asset values change and these weights/ratios might stray from what you originally specified. Rebalancing is simply buying/selling until the original weights/ratios are restored.

http://www.investopedia.com/terms/r/rebalancing.asp


#### [Adjusted] CAGR (Compound Annual Growth Rate)
Simply put, this is the average rate at which your portfolio grew every year. Adjusted CAGR is applicable only when contributions have been made to the portfolio after its inception; it doesn't include these contributions in the growth and tells you the 'net' growth per year.
> NOTE: growth is exponential, so this is not total growth divided by years.

http://www.investopedia.com/terms/c/cagr.asp


#### Sharpe Ratio
A ratio of returns:volatility. In other words, a value meant to quantify how much return you get on per unit of risk you take on. Often times risk is the variable controlled for when managing a portfolio. For example, two portfolios moved up 10% in a year, but the first moved drastically up and down along the way, while another moved in a straight line. The former is very volatile and would have a low ratio, while the latter is not volatile and would have a higher ratio. Typically, higher is better.

http://www.investopedia.com/terms/s/sharperatio.asp


#### Sortino Ratio
A ratio of returns:negative volatility. Similar to Sharpe, but this ignores volatility in the positive direction, since drastic upward moves are considered good.

http://www.investopedia.com/terms/s/sortinoratio.asp


#### Drawdown
A percent change between a peak and a valley on a chart. For our purposes, we care about maximum drawdowns, which is the biggest loss you incur along the way.

http://www.investopedia.com/terms/d/drawdown.asp


#### ETF (Exchange Traded Fund)
For all practical purposes, this is just another stock. The difference is, ETFs aren't based on spefic companies usually, but rather on and index or collections of companies/commodities/etc., usually based on some criteria.

http://www.investopedia.com/terms/e/etf.asp


#### Leveraged ETF
Assume there exists an ETF X. A leveraged ETF based on X would seek to multiply the returns of X by some factor (usually 2 or 3).
> NOTE: returns can be negative, so multiplying returns is typically considered very risky.

http://www.investopedia.com/terms/l/leveraged-etf.asp
