# Learning `data.table` Package for Finance (Introductory)
**Author**: Neeraj Jain

**email**: neeraj2409@outlook.com

------------

#### 1. Loading R Libraries
```{r echo=TRUE}
library(data.table)
library(magrittr) #for pipe operations (just used once)
```

#### 2. Reading data from GitHub
```{r echo=TRUE}
price <- fread("https://raw.githubusercontent.com/Neeraj2308/DataSet/main/price.csv")
```

Note: `fread` automatically detect date variable, and how values are separated. No need to add additional arguments. 

**Source**: *Prowess*

#### 3. Number of companies in the dataset
```{r echo=TRUE}
price[, unique(company_name)] %>% length
```
Pipe operations make the coding simple by using `%>%` 

#### 4.Variables to be considered
```{r echo=TRUE}
indicators <- c("company_name", "co_stkdate", "bse_closing_price", "bse_returns", "bse_market_cap")
```

Getting data for above variables only

```{r echo=TRUE}
price <- price[, ..indicators]
```

#### 5. Easy to put data into order

```{r echo=TRUE}
setorder(price, company_name, co_stkdate)
```
use `-` to put data into descending order.. 

```{r echo=TRUE}
setorder(price, -company_name, co_stkdate)
```


#### 6. Creating new variables

For example: taking return in decimal forms
```{r echo=TRUE}
price[, returns.d := bse_returns / 100]
```

#### 7. Removing NA values
```{r echo=TRUE}
price <- price[!is.na(returns.d)]
```




# Group Operations

**Easy to performs operations group wise in `data.table` package**

#### 8. Number of observations for each company
```{r echo=TRUE}
price[, .N, by = company_name]
```

#### 9. Calculating log returns own for each company

```{r echo=TRUE}
price[, logret := c(NA, diff(log(bse_closing_price))), by = company_name]

#alternative using shift
#price[, logre := log(bse_closing_price / shift(bse_closing_price, 1)), by = company_name]
```

#### 10. Removing first observation of each company, as return is NA
```{r echo=TRUE}
price <- price[, .SD[-1], by = company_name]
```



#### 11. Getting average return, sd, min, max, etc for each company
```{r echo=TRUE}
price[, .(Avg = mean(logret, na.rm = TRUE)*100, 
          SD = sd(logret, na.rm = TRUE)*100, 
          Min = min(logret, na.rm = TRUE)*100 ), 
      by = company_name]
```



#### 12. Calculating standard deviation of returns for each company for each year . 

```{r echo=TRUE}
price[, .(stdev = sd(logret, na.rm = TRUE)),
      keyby = .(company_name, year(co_stkdate))]
```
Note: keyby also put data into ascending order too. `by` can also be used.

#### 13. Doing winsorization for each company (for logret only)

Defining winsorization function
```{r echo=TRUE}
winsorize <- function(x, prob = .01) {
  q <- quantile(x, probs = c(prob, 1 - prob))
  x[x < q[1]] <- q[1]
  x[x > q[2]] <- q[2]
  return(x)
}
```

Doing winsorization: 
```{r echo=TRUE}
price[, logret.w := winsorize(logret, prob = 0.01), by = company_name]
```
New variable created logret.w

#### 14. comparing summary of winsorized data with non-winsorized data.
```{r echo=TRUE}
summary(price[, .(logret, logret.w)])
```


#### 15. getting month end price only or monthly data

```{r echo=TRUE}
price.m <- price[, .SD[.N], keyby = .(company_name, year(co_stkdate), month(co_stkdate))]
```
This can be also done by other packages like quantmod

#### 16. Getting average return for each day and setting this as a variable
```{r echo=TRUE}
price[, index.ret := mean(logret),
      by = .(year(co_stkdate),  yday(co_stkdate))]
```
Similarly we can calculate weighted return using market cap data. This can be done for sector wise also. 


# Final Note
we can use this package to do any kind of calculations required with financial data. Most important is working with this package is very fast. 
