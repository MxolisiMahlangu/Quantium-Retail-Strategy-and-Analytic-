# Quantium Virtual Internship - Retail Strategy and Analytics

## Table of contants

- [Project Overview ](#project-overview)
- [Data Sources](#data-sources)
- [Tools](#tools)
- [Data Cleaning](#data-cleaning)
- [Explanatory Data Analysis](#explanatory-data-analysis)
- [Data Analysis](#data-analysis)
- [Analysis Results](#analysis-results)
- [Recommendations](#recommendations)
- [References](#references)

### Project Overview 
This data analysis project aims to provide insights into retail customers to better understand the types of customers who purchase chips and their purchasing behavior within the region. By analyzing the various aspects of data, we seek to find the total sales of chips, describe customers by lifestage and how premium their general purchasing behavior is, how many customers are in each segment, how many chips are bought, etc.

The solution in this project is not the only solution as the case study is open-ended

### Data Sources 

The client provided two csv file datasets:

1. Transaction data : This dataset contain all the transection that were made by the company 

2. Customer data: This dataset contain all informatiion about customer like Lifestage or premium customers 

### Tools

- Excel - Used for Data Viewing
- R studio - For Data Cleaning and Data Analysis 

### Data Cleaning 

1. Load required libriries 
2. Data loading
3. Examine transaction data
4. Convert DATE column to a date format
5. Examine PROD_NAME
6. Examine the words in PROD_NAME to see if there are any incorrect entries, such as products that are not chips
7. Removing digits
8. Sorting them by this frequency in order of highest to lowest frequency
9. Remove salsa products
10. Summarise the data to check for nulls and possible outlier
11. Filter the dataset to find the outlier
12. Re‐examine transaction data
13. Count the number of transactions by date
14. Create a sequence of dates and join this the count of transactions by date
15. Setting plot themes to format graphs
16. Plot transactions over time
17. Filter to December and look at individual days
18. Creating Pack siz column by taking the digits that are in PROD_NAME
19. Creating a Brand from PROD_NAME
20. Cleaning a brand names
21. Examining customer data
22. Examining the values of lifestage and premium_customer
24. Merge transaction data to customer data   

### Explanatory Data Analysis 
EDA involves exploring merged retail data to understand the following:

**Understanding Customer Behavior**
Analyzing purchase history, preferences, and frequency.
Identifying high-value customers and their shopping habits.

**Sales and Revenue Analysis**
Finding trends in sales over time (seasonality, peak periods).
Comparing product performance across different regions or stores.

**Product and Inventory Insights**
Understanding demand patterns to optimize stock levels.
Identifying fast-moving vs. slow-moving products.

**Pricing and Discount Effectiveness**
Evaluating the impact of discounts, promotions, and dynamic pricing.
Understanding price elasticity and customer responses to price changes.

**Customer Segmentation**
Grouping customers based on demographics, spending behavior, or loyalty.
Tailoring marketing campaigns to specific segments.

**Market Basket Analysis**
Identifying frequently bought-together products.
Optimizing cross-selling and upselling strategies.

**Competitor and Trend Analysis**
Benchmarking sales and pricing against competitors.
Identifying emerging trends in customer preferences.



### Data Analysis

Let’s start with calculating total sales by LIFESTAGE and PREMIUM_CUSTOMER and plotting the split by
these segments to describe which customer segment contribute most to chip sales.

```R
p <‐ ggplot(data = sales) +
geom_mosaic(aes(weight = SALES, x = product(PREMIUM_CUSTOMER, LIFESTAGE),
fill = PREMIUM_CUSTOMER)) +↪
labs(x = "Lifestage", y = "Premium customer flag", title = "Proportion of
sales") +↪
theme(axis.text.x = element_text(angle = 90, vjust = 0.5))
#### Plot and label with proportion of sales
p + geom_text(data = ggplot_build(p)$data[[1]], aes(x = (xmin + xmax)/2 , y =
(ymin + ymax)/2, label = as.character(paste(round(.wt/sum(.wt),3)*100,
'%'))))
```
picture......
Sales are coming mainly from Budget - older families, Mainstream - young singles/couples, and Mainstream - retirees

```R
p <‐ ggplot(data = customers) +
geom_mosaic(aes(weight = CUSTOMERS, x = product(PREMIUM_CUSTOMER,
LIFESTAGE), fill = PREMIUM_CUSTOMER)) +↪
labs(x = "Lifestage", y = "Premium customer flag", title = "Proportion of
customers") +↪
theme(axis.text.x = element_text(angle = 90, vjust = 0.5))
#### Plot and label with proportion of customers
p + geom_text(data = ggplot_build(p)$data[[1]], aes(x = (xmin + xmax)/2 , y =
(ymin + ymax)/2, label = as.character(paste(round(.wt/sum(.wt),3)*100,
'%'))))
```
PICTURE
there are more Mainstream - young singles/couples and Mainstream - retirees who buy chips. This con-
tributes to there being more sales to these customer segments but this is not a major driver for the Budget - Older families segment.

Higher sales may also be driven by more units of chips being bought per customer. Let’s have a look at this
next.

```r
avg_units <‐ data[, .(AVG = sum(PROD_QTY)/uniqueN(LYLTY_CARD_NBR)),
.(LIFESTAGE, PREMIUM_CUSTOMER)][order(‐AVG)]↪
#### Create plot
ggplot(data = avg_units, aes(weight = AVG, x = LIFESTAGE, fill =
PREMIUM_CUSTOMER)) +↪
geom_bar(position = position_dodge()) +
labs(x = "Lifestage", y = "Avg units per transaction", title = "Units per
customer") +↪
theme(axis.text.x = element_text(angle = 90, vjust = 0.5))
```
picture

We also find that Older families and young families in general buy more chips per customer

Let’s also investigate the average price per unit chips bought for each customer segment as this is also a
driver of total sales.

```R
avg_price <‐ data[, .(AVG = sum(TOT_SALES)/sum(PROD_QTY)), .(LIFESTAGE,
PREMIUM_CUSTOMER)][order(‐AVG)]↪
#### Create plot
ggplot(data = avg_price, aes(weight = AVG, x = LIFESTAGE, fill =
PREMIUM_CUSTOMER)) +↪
geom_bar(position = position_dodge()) +
labs(x = "Lifestage", y = "Avg price per unit", title = "Price per unit") +
theme(axis.text.x = element_text(angle = 90, vjust = 0.5))
```
picture...........

Mainstream midage and young singles and couples are more willing to pay more per packet of chips com-
pared to their budget and premium counterparts. This may be due to premium shoppers being more likely to
buy healthy snacks and when they buy chips, this is mainly for entertainment purposes rather than their own
consumption. This is also supported by there being fewer premium midage and young singles and couples
buying chips compared to their mainstream counterparts.

As the difference in average price per unit isn’t large, we can check if this difference is statistically different. By performing an indipendent t-test between mainstream vs premium and budget midage
```R
pricePerUnit <‐ data[, price := TOT_SALES/PROD_QTY]
t.test(data[LIFESTAGE %in% c("YOUNG SINGLES/COUPLES", "MIDAGE SINGLES/COUPLES")
& PREMIUM_CUSTOMER == "Mainstream", price]↪
, data[LIFESTAGE %in% c("YOUNG SINGLES/COUPLES", "MIDAGE SINGLES/COUPLES")
& PREMIUM_CUSTOMER != "Mainstream", price]↪
, alternative = "greater")
```
picture.....

The t-test results in a p-value < 2.2e-16, i.e. the unit price for mainstream, young and mid-age singles and
couples are significantly higher than that of budget or premium, young and midage singles and couples.

**For control stores seletion**

The client has selected store numbers 77, 86 and 88 as trial stores and want control stores to be established
stores that are operational for the entire observation period.
We would want to match trial stores to control stores that are similar to the trial store prior to the trial period
of Feb 2019 in terms of :
- Monthly overall sales revenue
- Monthly number of customers
- Monthly number of transactions per customer

To find the control stores  we’ll select control stores based on how similar
monthly total sales in dollar amounts and monthly number of customers are to the trial stores. So we will
need to use our functions to get four scores, two for each of total sales and total customers. In the full document there are functions used to calculate this control stores based on the highest score. 

The store with the highest score is then selected as the control store since it is most similar to the trial store

```R
control_store <‐ score_Control[Store1 == trial_store,
][order(‐finalControlScore)][2, Store2]↪
control_store
```
233 is the control store for trail store 77  

### Analysis Results 

The analysis results are summaried as follow: 

Sales have mainly been due to Budget - older families, Mainstream - young singles/couples, and Mainstream- retirees shoppers. We found that the high spend in chips for mainstream young singles/couples and retirees is due to there being more of them than other buyers. Mainstream, midage and young singles andcouples are also more likely to pay more per packet of chips. This is indicative of impulse buying behaviour. We’ve also found that Mainstream young singles and couples are 23% more likely to purchase Tyrrells chips
compared to the rest of the population. The Category Manager may want to increase the category’s performance by off-locating some Tyrrells and smaller packs of chips in discretionary space near segments where young singles and couples frequent more often to increase visibilty and impulse behaviour.

For the control store selection we’ve found control stores 233, 155, 237 for trial stores 77, 86 and 88 respectively. The results for trial stores 77 and 88 during the trial period show a significant difference in at least two of the three trial months but this is not the case for trial store 86. We can check with the client if the implementation of the trial was different in trial store 86 but overall, the trial shows a significant increase in sales

### Recommendations 

**Improve Customer Segmentation**

Use LIFESTAGE and PREMIUM_CUSTOMER attributes to better target customer groups.
Identify key customer groups contributing to higher sales and tailor marketing strategies accordingly.

**Optimize Product Pricing and Promotions**

Evaluate the average chip price per customer segment and adjust pricing to maximize revenue.
Consider running promotions for high-value customer segments such as mainstream retirees and young singles/couples.

**Boost Customer Engagement**

Leverage insights into who buys the most chips to personalize marketing campaigns.
Implement loyalty programs targeted at frequent buyers to encourage repeat purchases.

**Monitor Trial Performance with Statistical Testing**

Compare total sales and customer numbers between trial and control stores.
Use t-tests to determine if the increase in sales is statistically significant.

**Evaluate Promotional Impact**

If a trial increases customer numbers but not sales, investigate potential price reductions or discounts affecting revenue.
Adjust pricing strategies to balance attracting new customers and maintaining profitability.

**Scale Successful Trials**

If a trial shows a statistically significant increase in sales, consider rolling out the strategy across more stores.
If results vary across different trial stores, refine the approach based on store-specific factors.

### References

1. OpenAI. (2025). ChatGPT (March 17 version) [Large language model]. https://openai.com


   
