---
title: "Berkshire Hathaway Valuation Jan 2026"
author: Rishabh Bhatia
categories: [business]
tags: business valuation brk.a brk.b
date: 2026-01-20 04:00:00 -0700
---

The best way to learn business valuation is by doing it. To kick off this series, I am evaluating the Berkshire 
Hathaway. To do this, I’ll be using two distinct frameworks:
1. The Private Business Owner Perspective: Treating the company as a collection of individual businesses.
2. The Aesop Framework: Using the classic "bird in the hand" proverb to determine value.

**Disclaimer**: This is not a financial advice, and only for educational purpose.

## Intrinsic value
The present value of all cash flows it can generate from now until its "judgment day" (forever), discounted back to 
today at a proper rate.

**Note:** Our goal is to be directionally correct and not precisely wrong.

### Valuating it like a private business deal
This is essentially valuating the company like a private business owner will do. He will first list all the main business 
units (within the company) that is responsible for the owner's earning's, and then he will go on to valuate each part.

### Valuating like Aesop
This is what we learnt early in school as a proverb - "a bird in the hand is worth two in the bush". This is essentially 
taught to every 1990s kid growing up in a middle class family in india. The extension of this becomes the complete 
investment philosophy -
1. **How certain are you** that there are birds in the bush?
2. **When will they emerge** and how many will there be?
3. **What is the risk-free interest rate** (the cost of waiting)?

## Berkshire Hathaway valuation like a private business deal
As a private business owner looking to buy a business from your town. 
For example - If you are buying a local retail shop. You will first calculate replacement cost( commercial shop price of
similar size and price of buying the same inventory). Then you will calculate the earning power / unit economics of the
business.  For retail shop, you will calculate the value of per sq ft by analyzing the footfall, average purchase, 
average time spent, seasonal impact. The idea to understand how much earning (post expenses) is retained by the 
business. Then you estimate future earnings and come up with the value that is reasonable to buy the business.

![ BRK Owners Earning 2024 10-K ](/assets/business/valuation/brk%202024%2010-k%20Owners%20Earning.png)

Now for Berkshire Hathaway, we can calculate the same - 
1. Cash - ~377 billion,
2. Railroad - has an operating earning of ~5 billion, and at 15 times earning(past anecdote) would be worth ~$75 billion,
3. Energy - has an operating earning of ~3.7 billion, and at 15 times earning(historical average) would be worth ~55 billion,
4. Insurance underwriting - has an operating earning (without float) of ~9 billion, and at 15 times earning(historical average) would be worth ~135 billion,
5. Manufacturing, service and retailing - has an operating earning of ~13 billion, and at 15 times earning(historical average) would be worth ~195 billion, 
6. Stock portfolio - has an investment of ~283 billion, 
7. other - has an operating earning of ~2.8 billion, and at 15 times earning(historical average) would be worth ~42 billion,

Adding all would be valued at ~$1,162 billion, where I took 15 times earning for all earning groups. Thus, at current 
market value of ~$1,060 billion BRK is fairly priced with no margin of safety. Conservatively, at 10 times earning, it
would be valued at ~$995 billion.  

Overall I think its fair valued and can be good place to start allocating capitol for long term.

## Berkshire Hathaway valuation like Aesop
Let's go deep into converting the philosophy to practice. What we defined in philosophy is a simple present value 
calculator.
```markdown
 PV = FV / (1 + r)^n
PV: Present Value
FV: Future Value
r: Rate of return (interest rate or discount rate) per period
n: Number of period
```
![ BRK Owners Earning 2024 10-K ](/assets/business/valuation/brk%202026-01%20DCF.png)

**As per the evaluation with margin of safety I think BRK will be an excellent buy at ~$570 billion, which is currently 
selling at ~$1.06 trillion.** 

### Owner's earning
Owner's earning is defined as the cash flows that can be distributed to owners after the business has paid all expenses
and capital expenditures for the investment.
```markdown
Owner Earnings = (a) Reported Earnings + 
                 (b) Depreciation, Depletion, Amortization, and other non-cash charges - 
                 (c) Average Annual Maintenance Capital Expenditures
```

Now for the Berkshire Hathaway, the owner's earning per year is ~$47 billion.    
Looking at the latest report filed to SEC - [Q3, 2025 10-Q](https://www.berkshirehathaway.com/qtrly/3rdqtr25.pdf) and 
[2024 10-k](https://www.berkshirehathaway.com/qtrly/3rdqtr24.pdf), we can see that it earned $88,995 million and 
subtract an investment gain of $41,558 million which gives us $47,437 million. If you want you can average out for 
last 5 years you can see that its around ~40 billion of value creation every year.


#### Why subtract ?   

> Investment gains (losses) predominantly derive from our investments in equity securities and include significant
unrealized gains and losses from changes in market prices and foreign currency exchange rates applicable to certain of our
investments. We believe that investment gains and losses, whether realized from dispositions or unrealized from changes in
market prices, are **generally meaningless in understanding our reported periodic results** or evaluating the economic performance
of our operating businesses.


### Terminal multiple
The terminal value is an estimation of cash you get in the last year of the assumed ownership of the stock. To get the 
estimation of selling price we can multiply the owner's earning of last year with the terminal multiple.   

It is impossible to predict the terminal multiple (P/E ratio) at the end of ownership. All we can do is estimate based on
the company, its story, and likelihood of its survival.

We need to keep things simple, we can do one of the following things -
1/ Use historical range P/E of S&P500 (7 and 30 - Source: [Multpl](https://www.multpl.com/s-p-500-pe-ratio)),  
2/ Use historical range of its industry,  
3/ Check the historical range of the P/E of the business itself,    
3/ too hard pile :)    

Now for the Berkshire Hathaway we can see that its fairly stable business and reflects majorly the strength of american
industrial and financial business. I think using the historical range P/E of S&P500 of 15 (normal case),
25 (exuberance), and 12 (margin of safety) make sense. 

### Discount rate
The discount rate is essentially defining the risk you are taking while owning the stock over 10-year rate on the US Treasury.
I keep my life simple and keep it at as 10% for most of the developed country. I came around this after scanning through
all the annual meeting and letters from Berkshire Hathaway.

If you wish you can complicate your life by learning risk free rates as taught in college MBA with concepts like 
Weighted Average Cost of Capital (WACC) or cost of equity using capital Asset Pricing Model. You can watch 
[Session 4 to Session 7](https://www.youtube.com/playlist?list=PLUkh9m2BorqkgpNyRpP-NL3BS4yvFabXk). I watched it and 
decided to simplify for my use case. 

I keep my life simple based on the universe of business I am looking at - 10%.  

### Growth rate
For coming up with growth rate, you need to dive deep to understand the business. I have divided the growth rate into 
two buckets - 1/ first 5 years, which will be closer to current rate, 2/ 5-10 years, that will require more imagination.  
Of-course, its impossible to predict future growth, thus we have three scenarios - normal case, exuberance (best case) 
and margin of safety (worst case). Here ideally one should write down 1 paragraph story for each case defining what 
should future looks like to meet the defined case. The paragraph should be constructed with both risk and reward. 
Thinking about risk is critical to the story.

Now for the Berkshire Hathaway I came up with growth rate as 6% (normal case), 5% (worst case) and 8% (best case). 
Notice I moved the best case to last as I want my brain to first think about risks.

My story behind the numbers:
BRK have these main earnings engines -    
1/ Insurance underwriting/income - representing 35-45% of total earning over last 10 years,   
2/ Railroad (BNSF) - representing 10-20% of total earning over last 10 years,  
3/ Energy (BHE) - representing 8-15% of total earning over last 10 years,  
4/ Manufacturing, Service and Retailing (MSR) - representing 27-40% of total earning over last 10 years,  
5/ stock portfolio - earnings are not meaningful, but represent ~6% of overall. 

All business BRK is in are relatively stable in terms of earnings and returns. I used AI to give me average growth rate
of each of these business for last 10 years
1. Insurance business has historically grown at ~14% with weight contribution to overall business is ~6.74%, 
2. Railroad business has historically been flat with growth at ~1% with weight contribution to overall business is ~0.18%,
3. Manufacturing, Service & Retailing has historically grown at ~10% with weight contribution to overall business is ~3%,
4. Energy (BHE) has historically grown at ~5.9% with weight contribution to overall business is ~0.5%.

If we take weighted average we get ~10% growth rate.

Now my estimations of growth rate is 6% (normal case), 5% (worst case) and 10% (best case) because of the size I don't 
think Berkshire (without Warren and Charlie) can replicate the past 10 year performance. At best case I am assuming 8% 
growth if Greg (new CEO) is able to do some big acquisition that can move the needle, and perform the capitol allocation
like Warren. At normal case I see it growing 6% where Greg's does excellent job of maintaining culture and
average job of capitol allocation. Given the fortress Warren/Charlie created I think 5% of growth at worst case is reasonable.

Note: I am bringing these numbers out of the thin air based on my understanding of the business.

### Dividend payout ratio
Dividend payout is the percentage of a company's earnings paid to shareholders as dividends. While calculating present
value we need to add the dividend payout ratio as that earning is paid to the shareholder.

Now Berkshire Hathaway has never paid any dividend, so we can keep it as 0. 

## Inverted thinking - How much BRK need in revenue to justify current market pricing
BRK has a 
1. current market cap - ~$1.04 trillion,
2. net margin - ~18%,
3. Revenue - ~372 billion dollar,
4. Revenue Growth (last 5 years) - ~7.7%
5. Revenue Growth (last year) - ~0.14%
6. ROE - ~10%
7. Avg ROE (last 5 year) - ~11% 
8. Cost of equity - ~8%
 
![brk 2026-01 breakeven revenue.png](/assets/business/valuation/brk%202026-01%20breakeven%20revenue.png)

Now based on above break even revenue to justify market cap of ~$1.04 trillion is ~$350 billion, which it already is earning.
That means its fairly valued, and don't need a fairy tale to justify the earnings.

[Valuation Template.xlsx](/assets/business/valuation/Valuation%20Template.xlsx)

## Reference
- [BRK fillings and letters](https://www.berkshirehathaway.com/), 
- [NYU MBA: primer to present value](https://pages.stern.nyu.edu/~adamodar/New_Home_Page/PVPrimer/pvprimer.htm),
- Free DCF calculator downloaded from google
- [NYC prof: Aswath Damodaran](https://www.youtube.com/playlist?list=PLUkh9m2BorqkgpNyRpP-NL3BS4yvFabXk) 
- [Watching bunch of value investing channels in youtube](https://www.youtube.com/)

## Disclaimer
This is not a financial advice, and only for educational purpose.
