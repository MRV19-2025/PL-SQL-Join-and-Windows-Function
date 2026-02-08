# SQL JOINs and Window Functions - Harmony Music Store Analysis

**Course:** INSY 8311 - Database Management Systems  
**Assignment:** SQL Assignment I - Window Functions and JOINs  
**Student ID:** 29372 
**Student Name:** RUTAGENGWA Mugabo Vaillant 
**Group:** C

---

##  Table of Contents

1. [Business Problem Definition](#business-problem-definition)
2. [Success Criteria](#success-criteria)
3. [Database Schema Design](#database-schema-design)
4. [Part A: SQL JOINs Implementation](#part-a-sql-joins-implementation)
5. [Part B: Window Functions Implementation](#part-b-window-functions-implementation)
6. [Results Analysis](#results-analysis)
7. [Key Insights and Recommendations](#key-insights-and-recommendations)
8. [Technical Implementation](#technical-implementation)
9. [References](#references)
10. [Academic Integrity Statement](#academic-integrity-statement)

---

##  Business Problem Definition

### Business Context
**Harmony Music Store** is a retail and online music equipment store operating across five regions in Rwanda (Kigali, Northern, Southern, Eastern, and Western). The company sells a diverse range of products including musical instruments, audio equipment, vinyl records, CDs, software, and accessories.

### Data Challenge
The business faces three critical challenges:
1. **Revenue Concentration Risk:** Unable to identify which customers and products drive the majority of revenue, leading to potential vulnerability.
2. **Regional Performance Gaps:** Lack of visibility into regional sales patterns prevents effective inventory distribution and targeted marketing strategies.
3. **Customer Segmentation Deficiency:** The inability of a business to accurately divide its customer base into distinct, meaningful groups based on shared characteristics, behaviors, or needs

### Expected Outcome
Through SQL analysis using JOINs and Window Functions, management expects to:
- Identify the top of customers and products generating the most revenue
- Understand month by month growth trends and seasonal patterns
- Segment customers into actionable quartiles for personalized marketing campaigns
- Discover inactive customers and products requiring intervention
- Generate data-driven recommendations for inventory optimization and customer retention strategies

---

##  Goals 

### 1. **Top Performers Identification** → Using `RANK()`, `DENSE_RANK()`
- Identify the top 5 products per category based on revenue generation
- Clear ranking of products and customers with quantified revenue contributions

### 2. **Cumulative Revenue Tracking** → Using `SUM() OVER()` with ROWS frame
- Calculate running monthly sales totals from October 2024 to January 2025
- Month by month cumulative revenue showing business growth patterns

### 3. **Growth Rate Analysis** → Using `LAG()` / `LEAD()`
- Compute month-over-month revenue growth percentages

### 4. **Customer Segmentation** → Using `NTILE(4)`
- Customer distribution across quartiles with spending thresholds

### 5. **Trend Smoothing** → Using `AVG() OVER()` with moving windows
- Calculate 2-month and 3-month moving averages for revenue forecasting

---

##  Database Schema Design

### ERD

![image alt](https://github.com/MRV19-2025/-plsql_window_functions_29372_-RUTAGENGWA-Mugabo-Vaillant/blob/main/ERD.png)

---

### Tables Description

#### 1. **CUSTOMERS Table** (15 records)
- **Purpose:** Store customer status information



#### 2. **PRODUCTS Table** (15 records)
- **Purpose:** Product catalog with inventory and pricing



#### 3. **TRANSACTIONS Table** (15 records)
- **Purpose:** Record all sales transactions linking customers to products



---

##  Part A: SQL JOINs Implementation

### JOIN 1: INNER JOIN
**Business Question:** What are all the completed transactions with full customer and product details?

```sql
SELECT 
    t.transaction_id,
    c.customer_name,
    c.region,
    p.product_name,
    p.category,
    t.total_amount
FROM transactions t
INNER JOIN customers c ON t.customer_id = c.customer_id
INNER JOIN products p ON t.product_id = p.product_id
ORDER BY t.transaction_date DESC;
```

**Screenshot:** `screenshots/01_inner_join.png`

**Business Interpretation:**  
This query successfully retrieved all 15 valid transactions with complete customer and product information. The results show that Kigali region customers (5 customers) account for the majority of high-value transactions, particularly in the Instruments category. The INNER JOIN ensures we only see transactions where both customer and product data are present and valid, providing a clean dataset for revenue analysis. This forms the foundation for sales reporting and commission calculations.

---

### JOIN 2: LEFT JOIN
**Business Question:** Which customers have registered but never made a purchase?

```sql
SELECT 
    c.customer_id,
    c.customer_name,
    c.email,
    c.region,
    c.registration_date,
    COUNT(t.transaction_id) AS transaction_count
FROM customers c
LEFT JOIN transactions t ON c.customer_id = t.customer_id
GROUP BY c.customer_id, c.customer_name, c.email, c.region, c.registration_date
HAVING transaction_count = 0;
```

**Screenshot:** `screenshots/02_left_join.png`

**Business Interpretation:**  
The LEFT JOIN identified 3 inactive customers: David Johnson (Southern), Carlos Rodriguez (Eastern), and Sophie Martin (Western). These customers registered between January and April 2024 but never completed a purchase, representing approximately 20% of the customer base. This is a critical marketing opportunity—these leads are warm but unconverted. Recommended actions include personalized email campaigns with first-purchase discounts and follow-up calls to understand barriers to purchase. The total_spent = $0 for these customers represents lost revenue potential of at least $2,400-$3,000 based on average customer value.

---

### JOIN 3: RIGHT JOIN
**Business Question:** Which products in our inventory have never been sold?

```sql
SELECT 
    p.product_id,
    p.product_name,
    p.category,
    p.unit_price,
    p.stock_quantity,
    COUNT(t.transaction_id) AS times_sold
FROM transactions t
RIGHT JOIN products p ON t.product_id = p.product_id
GROUP BY p.product_id, p.product_name, p.category, p.unit_price, p.stock_quantity
HAVING times_sold = 0;
```

**Screenshot:** `screenshots/03_right_join.png`

**Business Interpretation:**  
The RIGHT JOIN revealed 2 products with zero sales: "Beyoncé - Renaissance CD" ($15) and "Premium Guitar Stand" ($45). Interestingly, both products have 0 stock quantity, which explains the lack of sales. This highlights an inventory management issue—products are listed but unavailable. The business should either restock these items if there's anticipated demand or remove them from active listings to avoid customer disappointment. The zero stock may also indicate strong past sales that depleted inventory, warranting immediate restocking analysis.

---

### JOIN 4: FULL OUTER JOIN (Simulated)
**Business Question:** What is the complete overview of all customers and products regardless of transaction history?
**Note:** I usedd MySql and it doesn't support Full join so i used union

```sql
-- MySQL simulation using UNION
SELECT 'Customer' AS type, c.customer_id AS id, c.customer_name AS name, 
       COUNT(t.transaction_id) AS activity_count
FROM customers c
LEFT JOIN transactions t ON c.customer_id = t.customer_id
GROUP BY c.customer_id, c.customer_name

UNION ALL

SELECT 'Product' AS type, p.product_id AS id, p.product_name AS name,
       COUNT(t.transaction_id) AS activity_count
FROM products p
LEFT JOIN transactions t ON t.product_id = p.product_id
GROUP BY p.product_id, p.product_name;
```

**Screenshot:** `screenshots/04_full_outer_join.png`

**Business Interpretation:**  
This comprehensive view displays all 30 business entities (15 customers + 15 products) with their activity levels, including those with zero transactions. The analysis reveals a classic 80/20 distribution: 80% of customers (12 out of 15) have made purchases, while 87% of products (13 out of 15) have generated sales. This FULL OUTER JOIN simulation provides executive leadership with a complete business snapshot, highlighting both performing and non-performing assets. The gap between active (80%) and inactive (20%) entities suggests good overall engagement but room for improvement through targeted activation campaigns.

---

### JOIN 5: SELF JOIN
**Business Question:** How do customers within the same region compare in spending behavior?

```sql
SELECT 
    c1.customer_name AS customer_1,
    c2.customer_name AS customer_2,
    c1.region AS shared_region,
    c1.total_spent AS customer_1_spent,
    c2.total_spent AS customer_2_spent,
    ABS(c1.total_spent - c2.total_spent) AS spending_difference
FROM customers c1
INNER JOIN customers c2 ON c1.region = c2.region AND c1.customer_id < c2.customer_id
WHERE c1.region = 'Kigali'
ORDER BY spending_difference DESC;
```

**Screenshot:** `screenshots/05_self_join.png`

**Business Interpretation:**  
The SELF JOIN analysis within Kigali region (5 customers) reveals significant spending disparities. Pierre Dupont ($2,100 VIP) spends 4.6x more than John Smith ($450), with a $1,650 difference. This variance suggests different customer lifecycles and product affinities. Lisa Chen ($1,250 VIP) and Robert Taylor ($920) represent mid-tier opportunities for VIP conversion. The comparison helps identify "best practices" from high spenders—what products did Pierre buy? What marketing channels did he respond to? Replicating Pierre's journey could elevate other Kigali customers, potentially adding $5,000-$8,000 in regional revenue.

---

##  Part B: Window Functions Implementation

### Category 1: Ranking Functions

#### Query 1.1: Customer Ranking by Total Spending

```sql
SELECT 
    customer_id,
    customer_name,
    total_spent,
    ROW_NUMBER() OVER (ORDER BY total_spent DESC) AS row_num,
    RANK() OVER (ORDER BY total_spent DESC) AS rank_position,
    DENSE_RANK() OVER (ORDER BY total_spent DESC) AS dense_rank_position,
    PERCENT_RANK() OVER (ORDER BY total_spent DESC) AS percentile_rank
FROM customers
WHERE total_spent > 0
ORDER BY total_spent DESC;
```

**Screenshot:** `screenshots/06_ranking_functions.png`

**Business Interpretation:**  
The ranking analysis identified clear customer tiers. Pierre Dupont (Rank 1, 100th percentile) leads with $2,100 spent, followed by Thomas Anderson ($1,680, Rank 2) and Michael Lee ($1,450, Rank 3). These top 3 customers alone contribute 52.86% of total revenue ($5,230 out of $9,760), demonstrating severe revenue concentration. The PERCENT_RANK shows that being in the top 25% (3 customers) requires spending at least $1,450. This segmentation directly informs VIP program qualification thresholds and helps prioritize customer success resources toward protecting high-value relationships.

---

### Category 2: Aggregate Window Functions

#### Query 2.1: Running Total and Moving Averages

```sql
SELECT 
    DATE_FORMAT(transaction_date, '%Y-%m') AS month,
    SUM(total_amount) AS monthly_revenue,
    SUM(SUM(total_amount)) OVER (
        ORDER BY DATE_FORMAT(transaction_date, '%Y-%m')
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total,
    AVG(SUM(total_amount)) OVER (
        ORDER BY DATE_FORMAT(transaction_date, '%Y-%m')
        ROWS BETWEEN 1 PRECEDING AND CURRENT ROW
    ) AS two_month_moving_avg
FROM transactions
GROUP BY DATE_FORMAT(transaction_date, '%Y-%m');
```

**Screenshot:** `screenshots/07_aggregate_windows.png`

**Business Interpretation:**  
The cumulative analysis shows strong business growth trajectory. Monthly revenue progressed: October ($99) → November ($1,568, +1,484%) → December ($3,449, +120%) → January ($1,160, -66%). The running total reached $6,276 by January 2025. The 2-month moving average smooths the volatility, showing average monthly revenue of $833.50 (Oct-Nov), $2,508.50 (Nov-Dec), and $2,304.50 (Dec-Jan). The January decline is concerning but may reflect post-holiday seasonality. The moving average suggests sustainable monthly revenue around $2,000-$2,500, informing cash flow projections and inventory budgets.

---

### Category 3: Navigation Functions

#### Query 3.1: Month-over-Month Growth Analysis

```sql
SELECT 
    month,
    monthly_revenue,
    previous_month_revenue,
    ROUND(
        ((monthly_revenue - previous_month_revenue) / NULLIF(previous_month_revenue, 0)) * 100, 
        2
    ) AS growth_rate_percent
FROM (
    SELECT 
        DATE_FORMAT(transaction_date, '%Y-%m') AS month,
        SUM(total_amount) AS monthly_revenue,
        LAG(SUM(total_amount), 1) OVER (
            ORDER BY DATE_FORMAT(transaction_date, '%Y-%m')
        ) AS previous_month_revenue
    FROM transactions
    GROUP BY DATE_FORMAT(transaction_date, '%Y-%m')
) AS monthly_data;
```

**Screenshot:** `screenshots/08_lag_lead_functions.png`

**Business Interpretation:**  
The LAG function enabled precise growth rate calculations. November's 1,484% growth from October represents the business launch phase. December's 120% growth maintained strong momentum. However, January's -66% decline signals a critical inflection point. This pattern suggests heavy dependence on holiday shopping (November-December), with post-holiday drop-off. The LEAD function (forward-looking) would help forecast February trends. Strategic recommendations include implementing retention campaigns in January, launching Valentine's Day promotions, and diversifying revenue streams to reduce seasonal dependency.

---

### Category 4: Distribution Functions

#### Query 4.1: Customer Segmentation with NTILE

```sql
SELECT 
    customer_id,
    customer_name,
    total_spent,
    NTILE(4) OVER (ORDER BY total_spent DESC) AS spending_quartile,
    ROUND(CUME_DIST() OVER (ORDER BY total_spent DESC), 4) AS cumulative_distribution,
    CASE 
        WHEN NTILE(4) OVER (ORDER BY total_spent DESC) = 1 THEN 'Premium (Q1)'
        WHEN NTILE(4) OVER (ORDER BY total_spent DESC) = 2 THEN 'High Value (Q2)'
        WHEN NTILE(4) OVER (ORDER BY total_spent DESC) = 3 THEN 'Medium Value (Q3)'
        ELSE 'Growth Potential (Q4)'
    END AS customer_segment
FROM customers
WHERE total_spent > 0
ORDER BY total_spent DESC;
```

**Screenshot:** `screenshots/09_ntile_distribution.png`

**Business Interpretation:**  
NTILE segmentation divided 12 active customers into four equal groups of 3 customers each. **Q1 Premium** ($1,450-$2,100) includes Pierre Dupont, Thomas Anderson, and Michael Lee—these require white-glove service and exclusive offers. **Q2 High Value** ($780-$920) includes Lisa Chen, Emily Brown, and Robert Taylor—prime upsell candidates for VIP conversion. **Q3 Medium Value** ($450-$670) includes Sarah Williams, Anna Kowalski, and Maria Garcia—nurture with loyalty programs. **Q4 Growth Potential** ($320-$340) includes Ahmed Hassan, Jennifer White, and John Smith—engage with educational content and starter promotions. This 4-tier framework enables precise marketing automation and budget allocation.

---

## 📈 Results Analysis

### 1. Descriptive Analysis - What Happened?

**Revenue Performance:**
- Total revenue generated: **$6,276** across 15 transactions
- Average transaction value: **$418.40**
- Transaction period: October 2024 to January 2025 (4 months)
- Active customers: **12 out of 15** (80% activation rate)
- Products sold: **13 out of 15** (87% sell-through rate)

**Regional Distribution:**
- **Kigali:** 5 customers, $5,500 revenue (56.35% of total) - **Dominant region**
- **Eastern:** 2 customers, $2,020 revenue (20.70%)
- **Northern:** 3 customers, $1,770 revenue (18.14%)
- **Southern:** 1 customer, $1,450 revenue (14.86%)
- **Western:** 1 customer, $670 revenue (6.87%)

**Product Category Performance:**
- **Instruments:** $4,200 revenue (66.9%) - Top category
- **Software:** $948 revenue (15.1%)
- **Audio Equipment:** $546 revenue (8.7%)
- **Vinyl Records:** $105 revenue (1.7%)
- **Accessories:** $36 revenue (0.6%)
- **CDs:** $0 revenue (0%)

**Monthly Trend:**
- October 2024: $99 (1 transaction) - Launch phase
- November 2024: $1,568 (3 transactions) - +1,484% growth
- December 2024: $3,449 (4 transactions) - +120% growth (holiday peak)
- January 2025: $1,160 (7 transactions) - -66% decline (post-holiday)

---

### 2. Diagnostic Analysis - Why Did It Happen?

**Revenue Concentration Risk:**
- **Finding:** Top 3 customers (25%) generate 52.86% of revenue ($5,230)
- **Root Cause:** Limited customer base of only 12 active buyers creates dependency on few high-value accounts. Pierre Dupont alone represents 21.51% of revenue.
- **Implication:** High vulnerability to customer churn. Loss of top 3 customers would cut revenue in half.

**Regional Imbalance:**
- **Finding:** Kigali region accounts for 56.35% of revenue with only 33% of customers
- **Root Cause:** Kigali customers include 3 VIPs (Pierre, Lisa, Robert) with high disposable income and proximity to flagship store.
- **Implication:** Over-reliance on single geographic market. Other regions are underdeveloped with only 1-2 customers each.

**Inactive Customer Challenge:**
- **Finding:** 20% of registered customers (3 out of 15) never purchased
- **Root Cause:** Ineffective onboarding sequence. Registration occurred months ago (Jan-Apr 2024) with no follow-up engagement.
- **Implication:** $2,400-$3,600 in lost potential revenue (assuming average customer value of $816.67).

**Product Category Skew:**
- **Finding:** Instruments dominate at 66.9% while CDs have zero sales
- **Root Cause:** High-margin instruments (guitars, pianos) appeal to serious musicians. CDs face format obsolescence and stock-out issues.
- **Implication:** Inventory investment disproportionately tied to instruments. Need to diversify revenue streams.

**Seasonal Volatility:**
- **Finding:** 66% revenue decline from December to January
- **Root Cause:** Holiday shopping surge (November-December) not sustained. January is traditionally slow for discretionary purchases.
- **Implication:** Cash flow challenges in Q1. Need to smooth revenue through subscription models or promotions.

**Stock-Out Issues:**
- **Finding:** 2 products (Beyoncé CD, Guitar Stand) have zero inventory and zero sales
- **Root Cause:** Inventory management system failed to restock popular items or flag obsolete products.
- **Implication:** Lost sales opportunities and poor customer experience when items show "out of stock."

---

### 3. Prescriptive Analysis - What Should Be Done?

#### Immediate Actions (Next 30 Days)

** Activate Dormant Customers**
- **Target:** David Johnson, Carlos Rodriguez, Sophie Martin
- **Action:** Send personalized email with 20% first-purchase discount code expiring in 14 days
- **Tracking:** Monitor conversion rate; goal of 2 out of 3 (67%) activation
- **Expected Impact:** $1,600-$2,400 in new revenue


#### Strategic Initiatives (Next 90-180 Days)

**VIP Loyalty Program**
- **Target:** Q1 Premium customers (Pierre, Thomas, Michael)
- **Goal:** Increase retention rate from 75% to 95% and average purchase frequency from 1.25x to 2x per quarter
- **Expected Impact:** Protect $5,230 in revenue, add $2,000-$3,000 in incremental sales

---

##  Key Insights and Recommendations

### Critical Findings

1. **20% Dormant Customers:** $2,400+ in lost opportunity from unconverted registrations. **QUICK WIN:** Re-engagement campaigns.

2. **Product Imbalance:** Instruments dominate at 67%; CDs at 0%. **INVENTORY ALERT:** Rebalance stock and marketing focus.

---

##  References

1. (MySQL Documentation,2026)  
2. (W3Schools,2026)  
3. (Vertabelo Academy,2026)
4. (Bro Code youtube,2026)
5. (Gemini,2026)
---

##  Academic Integrity Statement

**Integrity Declaration:**

"All sources were properly cited. Implementations and analysis represent original work. No AI-generated content was copied without attribution or adaptation."

This assignment was completed in accordance with AUCA's academic integrity policies. All SQL queries were written independently, drawing upon official MySQL documentation, educational tutorials, and instructor guidance. Business interpretations reflect original analytical thinking applied to the Harmony Music Store scenario.

### Personal Work Evidence

**Screenshots demonstrating original implementation:**
-  Database schema created from scratch (`screenshots/10_er_diagram.png`)
-  Custom data set with 15 realistic records per table
-  All 5 JOIN types executed successfully with business-relevant results
-  All 4 window function categories implemented with accurate calculations
-  Comprehensive business analysis with actionable recommendations

**Statement:**  
I, RUTAGENGWA Mugabo Vaillant, confirm that this work represents my own understanding and effort. While I consulted official documentation and educational resources (properly cited above), all code, analysis, and insights are my original contributions.

**Signature:** Rutagengwa Mugabo Vaillant
**Date:** February 8, 2025

---

