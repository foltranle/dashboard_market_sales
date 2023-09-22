# dashboard_market_sales
Uploading and transforming the data
To upload the file into the Power BI, the json file needs to be converted into a ndjson file. The convertion was made using Python language in Google colab enviroment. The following query rewrite the json objects as new lines.

## creating the enviroment with python in google colab
from google.colab import files
import pandas as pd
import json

#uploading the json file and converting into a ndjson file to upload in BigQuery
json_file_name = 'WS_assignment_transactions_supermarket.json'
ndjson_file_name = 'WS_assignment_transactions_supermarket.ndjson'

with open(json_file_name, 'r') as json_file:
    data = json.load(json_file)

## Open the NDJSON file as "writer" and convert every json object as a new line.
with open(ndjson_file_name, 'w') as ndjson_file:
    for record in data:
        ndjson_file.write(json.dumps(record) + '\n')
After uploading the file, the initial treatments were made:

In VIEW tab, selected the box of Column quality and Column distribuition to help with the understandment of the data.
Selected the columns with not used values to be hiden during the data treatment


To improve the dashboard, some demographics values was inserted in the database.

A new table, obtained from Kaggle datasets: Population of european countries.
Population of European Countries 2013-2022
Annual population figures for European countries (2013-2022)
https://www.kaggle.com/datasets/annafabris/population-of-european-countries-2013-2022?resourc

After adding this dataset, an index column was added to it, relating a unic number for each country. A custom column was inserted using the following PowerQuery formula:
Random_column = NumberRrandomBetween(x,y)
Using the functionality Merge Queries, the correlation was made.
Two more columns were added with the Random formula, to generate age and gender of customers. For the first one, the chosen range was only 0 or 1. And for the second, the minimum age it’s the most common legal age of majority, 18, untill 95 years old (more than 10 years above the best life expectancy from Europe in 2022)
For the gender column, a Conditional column was created, using the following logic:
If 'created_column' equals 0, then "Male"
else, "Female"
The random numbers from the formula for the age column was not integer, so it was needed to round the numbers to get more life related values.
To use the data information, a different treatment was made to the [Transaction.RecordDate] column: a conditional column was created using the first ten on the left numbers, and then converting the result to be presented as date.
Lastly in data treatment in Power Query, the [Transaction.MerchantName] column was used as base for conditioning a new table, grouping the distinct names of the same supermarket brand.


Already working inside of Power BI, 3 metrics and 1 new column were created:

Age_range column: for a better undertanding of the age of customers (randomly assigned) the following DAX formula was used:
Age range = 
SWITCH (
    TRUE (),
    'WS_assignment_transactions_supermarket'[Age] <= 18, "less than 18",
    'WS_assignment_transactions_supermarket'[Age] <= 30, "18-30",
    'WS_assignment_transactions_supermarket'[Age] <= 45, "31-45",
    'WS_assignment_transactions_supermarket'[Age] <= 60, "46-60",
    'WS_assignment_transactions_supermarket'[Age] > 60, "60+",
    BLANK ()
)
Average ticket: calculated based on total amount of sales
Average ticket = 
AVERAGE(
   'WS_assignment_transactions_supermarket'[Transaction.Amount]
)
Frequency: calculated from number of transaction per user
Frequency = 
DIVIDE(
    COUNT('WS_assignment_transactions_supermarket'[Transaction.Id]),
    DISTINCTCOUNT('WS_assignment_transactions_supermarket'[Users])
)
Spending per user: obtainde dividing the total amount by distinct users:
Spending per user = 
DIVIDE(
	SUM('WS_assignment_transactions_supermarket'[Transaction.Amount]),
	DISTINCTCOUNT('WS_assignment_transactions_supermarket'[Users])
)
Data visualization
In this dashboard, 3 pages were added in order to present the data in a way that could help in a potential need to undertand the best place and user profile to be invested on.

The first page of the dashboard present the Big numbers about sales related data as quantity of transactions, total revenue and average ticket. The big number cards moreover presents a tendency (sparkline). Also, and evolutive graphic and a detailed table was also included on it.

The second page is related to demographic profile: how customers are distributed by gender and age. Furthermore, customer behavior as average ticket, frequency of shopping and individual spending are illustrated in three horizontal cluested bar charts by gender. On top of that, the table by the botom right corner present the same metrics but using Country as the relational data.

Finaly, in the last page, the sales performance is presented in 4 graphics divided in 2 sections: Sales by retailer and Customer behaviour by retailer. The first section is mostly based in revenue and amount of transactions. And in the second one it’s possible to see the market penetration in one of the graphics and in the other a relation between frequency of shopping and spending per user.



Last comments
The dataset provided for the assignment truly tests distinct data analysis and processing capabilities. From file conversion, column grouping, to metric inclusion. For possible future jobs related to this dataset, maybe a wider range of dates could help to increase the acurracy of the insights that can be made with these results.  

