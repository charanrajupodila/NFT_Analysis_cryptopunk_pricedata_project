# NFT_Analysis_cryptopunk_pricedata_project
Over the past 18 months, an emerging technology has caught the attention of the world; the NFT. What is an NFT? They are digital assets stored on the blockchain. And over $22 billion was spent last year on purchasing NFTs. Why? People enjoyed the art, the speculated on what they might be worth in the future, and people didn’t want to miss out. 
 
The future of NFT’s is unclear as much of the NFT’s turned out to be scams of sorts since the field is wildly unregulated. They’re also contested heavily for their impact on the environment.
 
Regardless of these controversies, it is clear that there is money to be made in NFT’s. And one cool part about NFT’s is that all of the data is recorded on the blockchain, meaning anytime something happens to an NFT, it is logged in this database. 
 
In this project, you’ll be tasked to analyze real-world NFT data. 
That data set is a sales data set of one of the most famous NFT projects, Cryptopunks. Meaning each row of the data set represents a sale of an NFT. The data includes sales from January 1st, 2018 to December 31st, 2021. The table has several columns including the buyer address, the ETH price, the price in U.S. dollars, the seller’s address, the date, the time, the NFT ID, the transaction hash, and the NFT name.
You might not understand all the jargon around the NFT space, but you should be able to infer enough to answer the following prompts.

__________________________________________________________________


```sql
-- Use the specified database
USE DATABASE_NAME;

-- Task 1: Count the total number of sales in the given time period
SELECT count(*) AS total_sales 
FROM pricedata
WHERE event_date >= '2018-01-01'
  AND event_date <= '2021-12-31';

-- Task 2: Return the top 5 most expensive transactions
SELECT name, eth_price, usd_price, event_date 
FROM pricedata
ORDER BY usd_price DESC 
LIMIT 5;

-- Task 3: Return a table with moving average of USD price for the last 50 transactions
SELECT CAST(event_date AS date) AS date,
       USD_price,
       AVG(USD_price) OVER (ORDER BY transaction_hash ROWS BETWEEN 49 PRECEDING AND CURRENT ROW) AS moving_average
FROM pricedata
ORDER BY transaction_hash;

-- Task 4: Return NFT names and their average sale price in USD
SELECT name, AVG(usd_price) AS average_price 
FROM pricedata
GROUP BY name
ORDER BY average_price DESC;

-- Task 5: Return each day of the week with number of sales and average ETH price
SELECT DATE(event_date) AS day_of_week, 
       COUNT(*) AS no_of_sales_ontheday, 
       AVG(eth_price) AS avg_eth_price
FROM pricedata
GROUP BY DATE(event_date)
ORDER BY no_of_sales_ontheday ASC;

-- Task 6: Construct a summary column describing each sale
SELECT CONCAT(name, ' was sold for $', ROUND(usd_price, -3), ' to ', buyer_address, ' from ', seller_address, ' on ', event_date) AS summary 
FROM pricedata;

-- Task 7: Create a view for sales where specific wallet was the buyer
CREATE VIEW 1919_purchases AS
SELECT * 
FROM pricedata
WHERE buyer_address = '0x1919db36ca2fa2e15f9000fd9cdc2edcf863e685';

SELECT * FROM 1919_purchases;

-- Task 8: Create a histogram of ETH price ranges
SELECT ROUND(eth_price, -2) AS ETH_price_range, 
       COUNT(*) AS Count,
       RPAD('', COUNT(*), '*') AS BAR
FROM pricedata
GROUP BY ETH_price_range
ORDER BY ETH_price_range;

-- Task 9: Return a unioned query for highest and lowest price each NFT was bought for
SELECT name, MAX(usd_price) AS price, 'highest' AS status
FROM pricedata
GROUP BY name

UNION ALL

SELECT name, MIN(usd_price) AS price, 'lowest' AS status
FROM pricedata
GROUP BY name
ORDER BY name, status;

-- Task 10: NFT sold the most each month/year combination with name and price in USD
SELECT DATE_FORMAT(event_date, '%Y-%m') AS month_year,
       name AS NFT_name,
       MAX(usd_price) AS max_price_in_usd
FROM pricedata
GROUP BY DATE_FORMAT(event_date, '%Y-%m'), name
ORDER BY month_year, max_price_in_usd DESC;

-- Task 11: Total volume of sales on a monthly basis
SELECT DATE_FORMAT(event_date, '%Y-%m-01') AS month_year,
       ROUND(SUM(usd_price), -2) AS total_volume
FROM pricedata
GROUP BY DATE_FORMAT(event_date, '%Y-%m-01')
ORDER BY month_year;

-- Task 12: Count transactions for a specific wallet
SELECT COUNT(*) AS total_transaction_count
FROM pricedata
WHERE (buyer_address = '0x1919db36ca2fa2e15f9000fd9cdc2edcf863e685'
OR seller_address = '0x1919db36ca2fa2e15f9000fd9cdc2edcf863e685')
AND event_date >= '2018-01-01'
AND event_date <= '2021-12-31';

-- Task 13: Estimated average value calculator excluding daily outliers
WITH DailyAverage AS (
    SELECT event_date, usd_price,
           AVG(usd_price) OVER (PARTITION BY event_date) AS daily_avg_price
    FROM pricedata
),
FilteredData AS (
    SELECT event_date, usd_price, daily_avg_price
    FROM DailyAverage
    WHERE usd_price >= 0.1 * daily_avg_price
)
SELECT event_date,
       AVG(usd_price) AS estimated_value
FROM FilteredData
GROUP BY event_date
ORDER BY event_date;

-- Task 14: List ordered by wallet profitability (create your query here)

-- Example placeholder query for wallet profitability
-- You will need to define what "profitability" means in your context, possibly calculating total buy vs sell for each wallet.
SELECT buyer_address AS wallet_address, 
       SUM(usd_price) AS total_spent
FROM pricedata
GROUP BY buyer_address
ORDER BY total_spent DESC;
