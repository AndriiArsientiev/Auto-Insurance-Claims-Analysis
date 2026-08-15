# Auto-Insurance-Claims-Analysis
SQL project | Analysis of auto insurance claims
Table of Contents:
Business Task
Relationship diagram
Questions and Solution

Businss Task:
The auto insurance data appears profitable overall, but the question is: where exactly is the money being lost?
The analysis explores whether the losses come from specific segments (states, coverage types, sales channels) or from individual problem customers. It also examines whether complaints signal financial risk, and whether the retention strategy is targeting the right people.
The goal is to find the customers costing the company money, understand what makes them different, and produce a concrete action plan.
Relationship diagram
#Table1 here
#table 2 here
Question and Solution
A: Data quality check
1. How many records do we have and are there duplicates?

2. Create a cleaned table
3. Are the financial fields complete?

   no missing values in premium, claims, or lifetime value. No negative or zero premiums, no negative claims. Premium and claim ranges look reasonable.
4. What about behavioral fields?

5. Any logical violations in the data?
   a chunk of customers have their last claim dated before their current policy started. A history with previous insurer
6.How many customers have zero income?
7. Are customers with prior insurance history different?
   comparing average claims, premiums, and complaints. The prior-history group behaves almost identically to everyone else
8.What about date formats?
What we see: effective_to_date comes in mixed formats (2-digit and 4-digit years). So any time-based analysis uses months_since_* fields instead of parsing the dates.
B. Customer Profitability
1. How do we calculate profit per customer?
   built a table with lifetime premium (monthly × months), net position (claims minus premium), and loss ratio per customer.
2. What does the portfolio look like overall?
What we see: total premium, total claims, overall loss ratio. The vast majority of customers are profitable; a small share loses money
3. How do customers split by risk level?
a tiny Critical band
4. Who are the worst customers?
    pattern is Basic / Personal Auto via Agent; almost none of them complain
5. Can we trust the CLV field from the dataset?
C.Portfolio Analytics
1.What values do response and offer take?
2.Are there loss-making segments?
3. Does the sales channel matter?
4. Does geography matter?
D.Exceptions
1.How many customers fall into each problem category?
2.Are complaints linked to losing money?
3.Which group renews more often losing or profitable?
losing customers renew more often than profitable ones
4.Who is on the final exception list?
E. Summaryt

Final Findings
1. Data is healthy, only small share of customers loses money
2. Segments look the same
3. Most loses come from very big claims
4. Losing customers renew more often
Limitations:
Clainm amount is the customer's last claim
Missing income for 25% of customers
Dates in mixed formats 
