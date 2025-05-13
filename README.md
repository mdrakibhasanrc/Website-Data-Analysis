## Rakiblytics Website Data Analysis

This project showcases a full-cycle **Website Data Analysis** using **GA4**, **BigQuery**, and **Looker Studio**. It focuses on uncovering actionable insights for improving **conversion rate**, **user experience**, and overall **website performance**.

---

## 📊 Project Objective

To analyze key performance indicators (KPIs) and user behavior to identify growth opportunities, segment audiences, and provide data-driven recommendations for website optimization.

---

## 🚀 Tech Stack

- **Google Analytics 4 (GA4)** – for website tracking data  
- **BigQuery** – for querying and modeling raw event data  
- **Looker Studio** – for interactive dashboard and visual reporting  
- **SQL** – for data transformation and segmentation  
- **Google Slides** – for presentation

---

## 📈 Key Analysis Modules:
1.  **KPI :**  
- Total user, View Item Rate, Add to cart Rate, Checkout Rate, Purchase Rate, Cart Abandonment Rate,
- Checkout Abandonment Rate, CR, Revenue, ARPU, Bounce Rate, Avg Session Duration
- 
2 . **Breakdown Analysis by:**  
   - Device Type  - Traffic Source & Medium  - Category  - Age & Country   - Campaign   - Landing Page & Page Group  
   - Returning vs New Visitors   - Funnel Stages   - Product (Item)  - Customer Segments

4. **Customer Cohort Analysis**  
5. **Customer Lifetime Value (LTV) Analysis**

---

## ✅Website Metrics Performance Overview

This section provides key insights into website engagement, conversion rates, and user behavior across the e-commerce funnel. It highlights areas of strength and opportunities for optimizing the user journey from product view to purchase.

 ```
-- Declare the start and end dates for the analysis period
declare start_date date default '2020-11-01';  
declare end_date date default '2021-01-31';

with base as (
  SELECT 
    user_pseudo_id,
    event_name,
    concat(user_pseudo_id,(select value.int_value from unnest(event_params) where key ='ga_session_id')) as session_id,
    (select value.int_value from unnest (event_params) where key='engagement_time_msec') as engagement_time_msec,
    event_date,
    (select value.string_value from unnest (event_params) where key ='session_engaged') as session_engaged,
    ecommerce.purchase_revenue AS revenue

FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
where parse_date('%Y%m%d', event_date) between start_date and end_date
),

metric as (
  select
      count(distinct user_pseudo_id) as total_users,
      count(distinct case when event_name='page_view' then user_pseudo_id end) as page_view_users,
      count(distinct case when event_name='view_item' then user_pseudo_id end) as view_item_users,
      count(distinct case when event_name='add_to_cart' then user_pseudo_id end) as add_to_cart_users,
      count(distinct case when event_name='begin_checkout' then user_pseudo_id end) as begin_checkout_users,
      count(distinct case when event_name='purchase' then user_pseudo_id end) as purchase_users,
      sum(revenue) as total_revenue
  from base
),
 metric_calc as (
      select
       total_users as users,

       round(safe_divide(view_item_users,nullif(page_view_users,0))*100,2) as view_rate,
       round((1 - safe_divide(view_item_users, nullif(page_view_users, 0))) * 100, 2) as afetr_page_abandonment_rate,

       round(safe_divide(add_to_cart_users,nullif(view_item_users,0))*100,2) as add_to_cart_rate,
       round((1 - safe_divide(add_to_cart_users, nullif(view_item_users, 0))) * 100, 2) as after_view_abandonment_rate,

       round(safe_divide(begin_checkout_users,nullif(add_to_cart_users,0))*100,2) as checkout_rate,
       round((1 - safe_divide(begin_checkout_users, nullif(add_to_cart_users, 0))) * 100, 2) as cart_abandonment_rate,
       

       round(safe_divide(purchase_users,nullif(begin_checkout_users,0))*100,2) as purchase_rate,
       round((1 - safe_divide(purchase_users, nullif(begin_checkout_users, 0))) * 100, 2) as checkout_abandonment_rate,
      
       total_revenue as revenue,
       round(safe_divide(total_revenue,nullif(purchase_users,0)),2) as AOV,
       round(safe_divide(total_revenue,total_users),2) as ARPU,

       round(safe_divide(purchase_users,nullif(total_users,0))*100,2) as conversion_rate,

from metric
 ),

 bounce_rate_calc as (
      select
         round(safe_divide(count(distinct session_id)-count(distinct case when session_engaged='1' then session_id end),
         count(distinct session_id))*100,2) as bounce_rate
    from base
 ),

 session_duration as (
  select
    safe_divide(sum(engagement_time_msec), 1000 * count(distinct session_id)) as avg_session_duration_sec
  from base
)

select
    m.users,
    m.view_rate,
    m.afetr_page_abandonment_rate,
    m.add_to_cart_rate,
    m.checkout_rate,
    m.cart_abandonment_rate,
    m.purchase_rate,
    m.checkout_abandonment_rate,
    m.revenue,
    m.AOV,
    m.ARPU,
    m.conversion_rate,
    b.bounce_rate,
    s.avg_session_duration_sec
  
from metric_calc m, bounce_rate_calc b, session_duration s;

 ```

### 🔑 Key Insights:

1. Strong Product View Engagement
      22.7% of total users viewed product pages.
      However, 77.3% of users dropped off after landing (after_page_abandonment_rate).
      🔍 Insight: High homepage or landing page drop-off — consider improving above-the-fold content, page load speed, or value proposition clarity.

2. Add-to-Cart Rate Looks Healthy
      20.48% of users who stayed added items to the cart.
      This suggests interest in products is strong once users engage.

3. Cart and Checkout Abandonment Needs Attention
      Cart Abandonment Rate: 77.44%
      Checkout Abandonment Rate: 45.49%
      🔍 Insight: These are higher than industry benchmarks.

4. Purchase Conversion Rate
      22.56% (this seems high — please confirm if this is of users who entered checkout or all users).
      If it's based on all users, this is excellent and shows strong bottom-funnel performance.

5. Revenue Metrics
      Total Revenue: $362,165
      AOV (Average Order Value): $81.96
      ARPU (Average Revenue Per User): $1.34
      🔍 Insight:
      High AOV suggests successful upselling or higher-ticket products.
      ARPU is low relative to AOV → Opportunity to improve conversion rate or retention to lift ARPU.

6. Site Engagement
      Bounce Rate: 32.82% → Fairly good (under 40% is healthy).
      Average Session Duration: ~70 seconds → Suggests users are exploring but might not be deeply engaging.
      🔍 Insight: Could experiment with personalized recommendations, better CTAs, or product discovery paths.

## 🎯 Summary Key Findings::


      Strong product engagement: 22.7% view rate, but 77.3% drop after landing → needs homepage optimization.
      
      Healthy add-to-cart rate (20.48%), but high cart (77.44%) and checkout (45.49%) abandonment → opportunity to streamline purchase experience.
      
      AOV is solid at $81.96, but ARPU is low ($1.34) → boost conversions to lift revenue per user.
      
      Bounce rate (32.8%) and session duration (70s) suggest room to improve site stickiness and engagement.
