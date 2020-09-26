## 1. The World Bank's international debt data
<p>It's not that we humans only take debts to manage our necessities. A country may also take debt to manage its economy. For example, infrastructure spending is one costly ingredient required for a country's citizens to lead comfortable lives. <a href="https://www.worldbank.org">The World Bank</a> is the organization that provides debt to countries.</p>
<p>In this notebook, we are going to analyze international debt data collected by The World Bank. The dataset contains information about the amount of debt (in USD) owed by developing countries across several categories. We are going to find the answers to questions like: </p>
<ul>
<li>What is the total amount of debt that is owed by the countries listed in the dataset?</li>
<li>Which country owns the maximum amount of debt and what does that amount look like?</li>
<li>What is the average amount of debt owed by countries across different debt indicators?</li>
</ul>
<p><img src="https://assets.datacamp.com/production/project_754/img/image.jpg" alt=""></p>
<p>The first line of code connects us to the <code>international_debt</code> database where the table <code>international_debt</code> is residing. Let's first <code>SELECT</code> <em>all</em> of the columns from the <code>international_debt</code> table. Also, we'll limit the output to the first ten rows to keep the output clean.</p>

%%sql
postgresql:///international_debt
SELECT *
FROM international_debt
LIMIT 10
    
    country_name	country_code	indicator_name	indicator_code	debt
Afghanistan	AFG	Disbursements on external debt, long-term (DIS, current US$)	DT.DIS.DLXF.CD	72894453.700000003
Afghanistan	AFG	Interest payments on external debt, long-term (INT, current US$)	DT.INT.DLXF.CD	53239440.100000001
Afghanistan	AFG	PPG, bilateral (AMT, current US$)	DT.AMT.BLAT.CD	61739336.899999999
Afghanistan	AFG	PPG, bilateral (DIS, current US$)	DT.DIS.BLAT.CD	49114729.399999999
Afghanistan	AFG	PPG, bilateral (INT, current US$)	DT.INT.BLAT.CD	39903620.100000001
Afghanistan	AFG	PPG, multilateral (AMT, current US$)	DT.AMT.MLAT.CD	39107845
Afghanistan	AFG	PPG, multilateral (DIS, current US$)	DT.DIS.MLAT.CD	23779724.300000001
Afghanistan	AFG	PPG, multilateral (INT, current US$)	DT.INT.MLAT.CD	13335820
Afghanistan	AFG	PPG, official creditors (AMT, current US$)	DT.AMT.OFFT.CD	100847181.900000006
Afghanistan	AFG	PPG, official creditors (DIS, current US$)	DT.DIS.OFFT.CD	72894453.700000003


## 2. Finding the number of distinct countries
<p>From the first ten rows, we can see the amount of debt owed by <em>Afghanistan</em> in the different debt indicators. But we do not know the number of different countries we have on the table. There are repetitions in the country names because a country is most likely to have debt in more than one debt indicator. </p>
<p>Without a count of unique countries, we will not be able to perform our statistical analyses holistically. In this section, we are going to extract the number of unique countries present in the table. </p>

%%sql
SELECT 
    COUNT(DISTINCT country_name) AS total_distinct_countries
FROM international_debt;

## 3. Finding out the distinct debt indicators
<p>We can see there are a total of 124 countries present on the table. As we saw in the first section, there is a column called <code>indicator_name</code> that briefly specifies the purpose of taking the debt. Just beside that column, there is another column called <code>indicator_code</code> which symbolizes the category of these debts. Knowing about these various debt indicators will help us to understand the areas in which a country can possibly be indebted to. </p>

%%sql
SELECT DISTINCT indicator_code AS distinct_debt_indicators
FROM international_debt
ORDER BY distinct_debt_indicators

## 4. Totaling the amount of debt owed by the countries
<p>As mentioned earlier, the financial debt of a particular country represents its economic state. But if we were to project this on an overall global scale, how will we approach it?</p>
<p>Let's switch gears from the debt indicators now and find out the total amount of debt (in USD) that is owed by the different countries. This will give us a sense of how the overall economy of the entire world is holding up.</p>

%%sql
SELECT 
    ROUND(SUM(debt)/1000000, 2) AS total_debt
FROM international_debt; 

## 5. Country with the highest debt
<p>"Human beings cannot comprehend very large or very small numbers. It would be useful for us to acknowledge that fact." - <a href="https://en.wikipedia.org/wiki/Daniel_Kahneman">Daniel Kahneman</a>. That is more than <em>3 million <strong>million</strong></em> USD, an amount which is really hard for us to fathom. </p>
<p>Now that we have the exact total of the amounts of debt owed by several countries, let's now find out the country that owns the highest amount of debt along with the amount. <strong>Note</strong> that this debt is the sum of different debts owed by a country across several categories. This will help to understand more about the country in terms of its socio-economic scenarios. We can also find out the category in which the country owns its highest debt. But we will leave that for now. </p>

%%sql
SELECT 
    country_name,
    SUM(debt) AS total_debt
FROM international_debt
GROUP BY country_name
ORDER BY total_debt DESC
LIMIT 1;

## 6. Average amount of debt across indicators
<p>So, it was <em>China</em>. A more in-depth breakdown of China's debts can be found <a href="https://datatopics.worldbank.org/debt/ids/country/CHN">here</a>. </p>
<p>We now have a brief overview of the dataset and a few of its summary statistics. We already have an idea of the different debt indicators in which the countries owe their debts. We can dig even further to find out on an average how much debt a country owes? This will give us a better sense of the distribution of the amount of debt across different indicators.</p>

%%sql
SELECT 
    indicator_code AS debt_indicator,
    indicator_name,
    AVG(debt) AS average_debt
FROM international_debt
GROUP BY debt_indicator, indicator_name
ORDER BY average_debt DESC
LIMIT 10;

## 7. The highest amount of principal repayments
<p>We can see that the indicator <code>DT.AMT.DLXF.CD</code> tops the chart of average debt. This category includes repayment of long term debts. Countries take on long-term debt to acquire immediate capital. More information about this category can be found <a href="https://datacatalog.worldbank.org/principal-repayments-external-debt-long-term-amt-current-us-0">here</a>. </p>
<p>An interesting observation in the above finding is that there is a huge difference in the amounts of the indicators after the second one. This indicates that the first two indicators might be the most severe categories in which the countries owe their debts.</p>
<p>We can investigate this a bit more so as to find out which country owes the highest amount of debt in the category of long term debts (<code>DT.AMT.DLXF.CD</code>). Since not all the countries suffer from the same kind of economic disturbances, this finding will allow us to understand that particular country's economic condition a bit more specifically. </p>

%%sql
SELECT 
    country_name, 
    indicator_name
FROM international_debt
WHERE debt = (SELECT 
                 MAX(debt)
             FROM international_debt
             WHERE indicator_code = 'DT.AMT.DLXF.CD');

## 8. The most common debt indicator
<p>China has the highest amount of debt in the long-term debt (<code>DT.AMT.DLXF.CD</code>) category. This is verified by <a href="https://data.worldbank.org/indicator/DT.AMT.DLXF.CD?end=2018&most_recent_value_desc=true">The World Bank</a>. It is often a good idea to verify our analyses like this since it validates that our investigations are correct. </p>
<p>We saw that long-term debt is the topmost category when it comes to the average amount of debt. But is it the most common indicator in which the countries owe their debt? Let's find that out. </p>

%%sql
SELECT indicator_code,
COUNT(indicator_code) AS indicator_count
FROM international_debt
GROUP BY indicator_code
ORDER BY indicator_count DESC, indicator_code DESC
LIMIT 20

## 9. Other viable debt issues and conclusion
<p>There are a total of six debt indicators in which all the countries listed in our dataset have taken debt. The indicator <code>DT.AMT.DLXF.CD</code> is also there in the list. So, this gives us a clue that all these countries are suffering from a common economic issue. But that is not the end of the story, a part of the story rather. </p>
<p>Let's change tracks from <code>debt_indicator</code>s now and focus on the amount of debt again. Let's find out the maximum amount of debt across the indicators along with the respective country names. With this, we will be in a position to identify the other plausible economic issues a country might be going through. By the end of this section, we will have found out the debt indicators in which a country owes its highest debt. </p>
<p>In this notebook, we took a look at debt owed by countries across the globe. We extracted a few summary statistics from the data and unraveled some interesting facts and figures. We also validated our findings to make sure the investigations are correct.</p>

%%sql
SELECT country_name, indicator_code, MAX(debt) AS maximum_debt
FROM international_debt
GROUP BY country_name, indicator_code
ORDER BY maximum_debt DESC
LIMIT 10
