# Equity Portfolio Summarizer
* Given the equity portfolio transactions as a CSV file, you can get a summary of the portfolio with information about realized and unrealized gain/loss for each stock as well as for the whole portfolio.
* Only transactions on Indian Stock exchanges NSE and BSE are supported as of now.
* It also classifies the gain/loss as Short Term or Long Term capital gains according to Indian income tax laws.

## Requirements to run this code ##
* Transactions in a CSV file in the format explained below
* Python 2.7 or any version higher than that
* _googlefinance_ library for python. If you have _pip_, it is as simple as running the command `pip install googlefinance`
* Other libraries like `csv, json, requests, decimal, collections, texttable` are also being used.

## Transactions File ##
The first line needs to contain the header given below. It contains various fields that define a transaction.
___Symbol,Name,Type,Date,Shares,Price,Amount,Brokerage,STT,Charges,Receivable,Mode___
Each line after the header represents a transaction with the transaction values specified in the same order as that of the header fields.

* _Symbol_ - stock ticker value including the stock exchange informatio. For example, _NSE:VBL_ stands for _Varun Beverages_ in _NSE_, _BOM:500285_ stands for _Spicejet_ in _BSE_.
* _Name_ - name of the entity as given by the user
* _Type_ - value can be _Buy_ or _Sell_
* _Date_ - date of the transaction in _"MMM dd, YYYY"_ format.
* _Shares_ - number of shares involved in the transaction
* _Price_ - actual price of each share at which the transaction happened
* _Amount_ - this is the total amount involved in the transaction(_shares_ * _price_). For _Buy_ transactions, it is negative(money goes out) and for _Sell_ transactions, it is positive(money comes in).
* _Brokerage_ - actual brokerage incurred for the transaction
* _STT_ - Securities Transaction Tax imposed by the Indian Government for this transaction
* _Charges_ - all other charges for this transaction combined into this one component. This can include Stamp Duty, Transaction Charges, Service Charges and SEBI Turnover Tax.
* Receivable - This is the amount you will get after adjusting the _Amount_ for _Brokerage_, _STT_ and _Charges_. This is basically _Amount_ - (_Brokerage_ + _STT_ + _Charges_). You shell out more than what is required to buy the stock and you get lesser than the sale amount when you sell the stock
* Mode - This is the type of trade. Its value can be _del_ or _sqr_. _del_ stands for delivery mode/cash and carry, _sqr_ stands for intra-day square off trade
* ___Any line starting with a pound sign is ignored. You can use this to have comments or some notes___
* ___The transactions should be listed in chronological order in this file___

### Sample Transactions File (say `portfolio.csv`) ###
```
Symbol,Name,Type,Date,Shares,Price,Amount,Brokerage,STT,Charges,Receivable,Mode
NSE:VBL,Varun Beverages,Buy,"Apr 08, 2016",99,445,-44055,1.51,44,6.55,-44107.06,del
NSE:VBL,Varun Beverages,Sell,"Apr 17, 2017",99,403.45,39941.55,199.71,40,35.47,39666.37,del
BOM:540678,Cochin Shipyard,Buy,"Aug 10, 2017",30,411,-12330,0.01,12,3.31,-12345.32,del
BOM:540678,Cochin Shipyard,Sell,"Sep 10, 2017",30,311,9330,0.01,9,2.31,9318.68,del
BOM:540716,ICICI Lombard,Buy,"Sep 27, 2017",39,661,-25779,0.01,25,5.01,-25809.02,del
```
### Running the code ###
`
python equity_stats.py portfolio.csv > output.txt
`

### Sample Output ###
* b_ - stands for buy; For example, b_date - buy date, b_charges - charges incurred during buy, b_value - buy value
* s_ - stands for sell; For example, s_date - sell date, b_charges - charges incurred during sell, s_value - sell value
* n_ - stands for net; n_charges - net charges (buy+sell), n_realized - net realized
* strg - short term realized gains, ltrg - long term realized gains
* h_value - holding value, h_charges - holding charges
* m_value - market value, m_price - market price
* a_price - average price, a_cost - average cost (cost = price + charges)
* u_gain - unit gain
* stug - short term unrealized gains, ltug - long term unrealized gains

```
PortFolio Realized Summmary
============================

+---------+---------+-----------+-----------+-----------+-----------+-----------+------------+----------+-----------+-----------+
|  name   | b_value |  s_value  | realized  | b_charges | s_charges | n_charges | n_realized | percent  |   strg    |   ltrg    |
+=========+=========+===========+===========+===========+===========+===========+============+==========+===========+===========+
| Cochin  | 12330   |      9330 |     -3000 |    15.320 |    11.320 |    26.640 |  -3026.640 | -24.547% | -3026.640 |         0 |
+---------+---------+-----------+-----------+-----------+-----------+-----------+------------+----------+-----------+-----------+
| Varun   | 44055   | 39941.550 | -4113.450 |    52.060 |   275.180 |   327.240 |  -4440.690 | -10.080% |         0 | -4440.690 |
+---------+---------+-----------+-----------+-----------+-----------+-----------+------------+----------+-----------+-----------+
| SUMMARY | 56385   | 49271.550 | -7113.450 |    67.380 |   286.500 |   353.880 |  -7467.330 | -13.243% | -3026.640 | -4440.690 |
+---------+---------+-----------+-----------+-----------+-----------+-----------+------------+----------+-----------+-----------+

PortFolio Holding Summmary
===========================

+---------+---------+-----------+--------+---------+---------+-----------+---------+--------+--------------+---------+---------+------+
|  name   | h_value |  m_value  | shares | a_price | m_price | h_charges | a_cost  | u_gain | n_unrealized | percent |  stug   | ltug |
+=========+=========+===========+========+=========+=========+===========+=========+========+==============+=========+=========+======+
| ICICI   | 25779   | 26744.250 |     39 |     661 | 685.750 |    30.020 | 661.770 | 23.980 |      935.230 |  3.628% | 935.230 |    0 |
+---------+---------+-----------+--------+---------+---------+-----------+---------+--------+--------------+---------+---------+------+
| SUMMARY | 25779   | 26744.250 |   *--* |    *--* |    *--* |    30.020 |    *--* |   *--* |      935.230 |  3.628% | 935.230 |    0 |
+---------+---------+-----------+--------+---------+---------+-----------+---------+--------+--------------+---------+---------+------+

ICICI Lombard (One Line Holding Summary)
=========================================

+---------+-----------+--------+---------+---------+-----------+---------+--------+--------------+---------+---------+------+
| h_value |  m_value  | shares | a_price | m_price | h_charges | a_cost  | u_gain | n_unrealized | percent |  stug   | ltug |
+=========+===========+========+=========+=========+===========+=========+========+==============+=========+=========+======+
| 25779   | 26744.250 |     39 |     661 | 685.750 |    30.020 | 661.770 | 23.980 |      935.230 |  3.628% | 935.230 |    0 |
+---------+-----------+--------+---------+---------+-----------+---------+--------+--------------+---------+---------+------+

ICICI Lombard (Holding Summary)
================================

+------------+--------+---------+---------+--------+---------+-----------+------------+-----------+---------+------+---------+
|   b_date   | shares | b_price | m_price | u_gain | h_value |  m_value  | unrealized | b_charges |  stug   | ltug | percent |
+============+========+=========+=========+========+=========+===========+============+===========+=========+======+=========+
| 2017-09-27 | 39     |     661 | 685.750 | 24.750 |   25779 | 26744.250 |    965.250 |    30.020 | 935.230 |    0 |  3.628% |
+------------+--------+---------+---------+--------+---------+-----------+------------+-----------+---------+------+---------+
| SUMMARY    | 39     |     661 | 685.750 | 24.750 |   25779 | 26744.250 |    965.250 |    30.020 | 935.230 |    0 |  3.628% |
+------------+--------+---------+---------+--------+---------+-----------+------------+-----------+---------+------+---------+

Cochin Shipyard (One Line Realized Summary)
============================================

+---------+---------+----------+-----------+-----------+-----------+------------+----------+-----------+------+
| b_value | s_value | realized | b_charges | s_charges | n_charges | n_realized | percent  |   strg    | ltrg |
+=========+=========+==========+===========+===========+===========+============+==========+===========+======+
| 12330   | 9330    |    -3000 |    15.320 |    11.320 |    26.640 |  -3026.640 | -24.547% | -3026.640 |    0 |
+---------+---------+----------+-----------+-----------+-----------+------------+----------+-----------+------+

Cochin Shipyard (Realized Summary)
===================================

+------------+------------+---------+---------+--------+--------+---------+---------+----------+-----------+-----------+-----------+-----------+------+----------+
|   b_date   |   s_date   | b_price | s_price | u_gain | shares | b_value | s_value | realized | b_charges | s_charges | n_charges |   strg    | ltrg | percent  |
+============+============+=========+=========+========+========+=========+=========+==========+===========+===========+===========+===========+======+==========+
| 2017-08-10 | 2017-09-10 |     411 |     311 |   -100 |     30 |   12330 |    9330 |    -3000 |    15.320 |    11.320 |    26.640 | -3026.640 |    0 | -24.547% |
+------------+------------+---------+---------+--------+--------+---------+---------+----------+-----------+-----------+-----------+-----------+------+----------+
| SUMMARY    | *--*       |     411 |     311 |   -100 |     30 |   12330 |    9330 |    -3000 |    15.320 |    11.320 |    26.640 | -3026.640 |    0 | -24.547% |
+------------+------------+---------+---------+--------+--------+---------+---------+----------+-----------+-----------+-----------+-----------+------+----------+

Varun Beverages (One Line Realized Summary)
============================================

+---------+-----------+-----------+-----------+-----------+-----------+------------+----------+------+-----------+
| b_value |  s_value  | realized  | b_charges | s_charges | n_charges | n_realized | percent  | strg |   ltrg    |
+=========+===========+===========+===========+===========+===========+============+==========+======+===========+
| 44055   | 39941.550 | -4113.450 |    52.060 |   275.180 |   327.240 |  -4440.690 | -10.080% |    0 | -4440.690 |
+---------+-----------+-----------+-----------+-----------+-----------+------------+----------+------+-----------+

Varun Beverages (Realized Summary)
===================================

+------------+------------+---------+---------+---------+--------+---------+-----------+-----------+-----------+-----------+-----------+------+-----------+----------+
|   b_date   |   s_date   | b_price | s_price | u_gain  | shares | b_value |  s_value  | realized  | b_charges | s_charges | n_charges | strg |   ltrg    | percent  |
+============+============+=========+=========+=========+========+=========+===========+===========+===========+===========+===========+======+===========+==========+
| 2016-04-08 | 2017-04-17 |     445 | 403.450 | -41.550 |     99 |   44055 | 39941.550 | -4113.450 |    52.060 |   275.180 |   327.240 |    0 | -4440.690 | -10.080% |
+------------+------------+---------+---------+---------+--------+---------+-----------+-----------+-----------+-----------+-----------+------+-----------+----------+
| SUMMARY    | *--*       |     445 | 403.450 | -41.550 |     99 |   44055 | 39941.550 | -4113.450 |    52.060 |   275.180 |   327.240 |    0 | -4440.690 | -10.080% |
+------------+------------+---------+---------+---------+--------+---------+-----------+-----------+-----------+-----------+-----------+------+-----------+----------+

```
