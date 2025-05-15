## Website Data Analysis

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
  Checkout Abandonment Rate, CR, Revenue, ARPU, Bounce Rate, Avg Session Duration
  
2 . **Breakdown Analysis by:**  
   - Device Type  - Traffic Source & Medium  - Category  - Age & Country   - Campaign   - Landing Page & Page Group  
     Returning vs New Visitors   - Funnel Stages   - Product (Item)  - Customer Segments

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

## 🎯 Summary Key Findings:


      Strong product engagement: 22.7% view rate, but 77.3% drop after landing → needs homepage optimization.
      
      Healthy add-to-cart rate (20.48%), but high cart (77.44%) and checkout (45.49%) abandonment → opportunity to streamline purchase experience.
      
      AOV is solid at $81.96, but ARPU is low ($1.34) → boost conversions to lift revenue per user.
      
      Bounce rate (32.8%) and session duration (70s) suggest room to improve site stickiness and engagement.


## ✅Website Device Analysis

The device analysis reveals that all devices (desktop, mobile, and tablet) experience significant drop-offs after landing, with high cart and checkout abandonment rates, suggesting the need for improved homepage and purchase flow optimizations. Conversion rates and revenue vary by device, with desktop leading in AOV and revenue, while mobile and tablet show opportunities for boosting engagement and conversions.

 ```
-- Declare the start and end dates for the analysis period
declare start_date date default '2020-11-01';  
declare end_date date default '2021-01-31';

with base as (
 SELECT 
   user_pseudo_id,
   event_name,
   device.category AS device_category,
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
    device_category,
     count(distinct user_pseudo_id) as total_users,
     count(distinct case when event_name='page_view' then user_pseudo_id end) as page_view_users,
     count(distinct case when event_name='view_item' then user_pseudo_id end) as view_item_users,
     count(distinct case when event_name='add_to_cart' then user_pseudo_id end) as add_to_cart_users,
     count(distinct case when event_name='begin_checkout' then user_pseudo_id end) as begin_checkout_users,
     count(distinct case when event_name='purchase' then user_pseudo_id end) as purchase_users,
     sum(revenue) as total_revenue
 from base
 group by device_category
),
metric_calc as (
     select
      device_category,
      total_users as users,

      round(safe_divide(view_item_users,nullif(page_view_users,0))*100,2) as view_rate,
      round((1 - safe_divide(view_item_users, nullif(page_view_users, 0))) * 100, 2) as droping_after_view,

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
      device_category,
        round(safe_divide(count(distinct session_id)-count(distinct case when session_engaged='1' then session_id end),
        count(distinct session_id))*100,2) as bounce_rate
   from base
   group by device_category
),

session_duration as (
 select
    device_category,
   safe_divide(sum(engagement_time_msec), 1000 * count(distinct session_id)) as avg_session_duration_sec
 from base
 group by device_category
)

select
  m.device_category,
   m.users,
   m.view_rate,
   m.droping_after_view,
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
 
FROM metric_calc m
LEFT JOIN bounce_rate_calc b USING(device_category)
LEFT JOIN session_duration s USING(device_category)
ORDER BY m.device_category;

 ```
## 🎯 Result:
![Screenshot_5](https://github.com/user-attachments/assets/9236644b-9163-47c0-bdce-5b5af1b3ba74)

![Screenshot_7](https://github.com/user-attachments/assets/ff4ac75e-d692-4883-938d-14b052cfcad5)


## 🎯 Summary Key Findings:


     High Drop-offs After Landing: 
     All devices show significant drop-offs after landing (77.1% for desktop, 77.24% for mobile, and 76.87% for tablet), indicating a need for homepage  optimization to retain traffic.

     Cart & Checkout Abandonment: 
     Despite healthy add-to-cart rates (20.33% desktop, 20.73% mobile, 19.13% tablet), cart abandonment is high (76.72% desktop, 77.17% mobile, 79.35% tablet), and checkout abandonment 
     is also a concern across devices, pointing to opportunities for UX improvements in the purchase flow.

    Conversion & Revenue Performance: 
    Conversion rates are relatively low (1.6% desktop, 1.7% mobile, 1.55% tablet), with desktop generating the highest revenue ($208,815), strong AOV ($82.18), and low ARPU ($1.31). 
    Mobile and tablet have lower AOV and ARPU, suggesting room for improvement in conversions per user.



## ✅ Landing Page Analysis Report

Landing Page Analysis is important because it helps identify where users drop off early in their journey, impacting conversion rates. Understanding performance by page type allows targeted improvements to user experience, reducing bounce and abandonment rates.

 ```
DECLARE start_date DATE DEFAULT '2020-11-01';  
DECLARE end_date DATE DEFAULT '2021-01-31';

WITH base AS (
  SELECT 
    user_pseudo_id,
    event_name,
    event_timestamp,
    CONCAT(user_pseudo_id, (SELECT value.int_value FROM UNNEST(event_params) WHERE key = 'ga_session_id')) AS session_id,
    (SELECT value.int_value FROM UNNEST(event_params) WHERE key = 'engagement_time_msec') AS engagement_time_msec,
    (SELECT value.string_value FROM UNNEST(event_params) WHERE key = 'session_engaged') AS session_engaged,
    ecommerce.purchase_revenue AS revenue
  FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
  WHERE PARSE_DATE('%Y%m%d', event_date) BETWEEN start_date AND end_date
),

landing_pages AS (
  SELECT
    session_id,
    user_pseudo_id,
    (SELECT value.string_value FROM UNNEST(event_params) WHERE key = 'page_location') AS landing_page_url,
    event_timestamp
  FROM base
  WHERE event_name = 'page_view'
  QUALIFY ROW_NUMBER() OVER (PARTITION BY session_id ORDER BY event_timestamp) = 1
),

landing_page_types AS (
  SELECT
    session_id,
    user_pseudo_id,
    CASE
      WHEN REGEXP_CONTAINS(landing_page_url, r'/collections') THEN 'collection page'
      WHEN REGEXP_CONTAINS(landing_page_url, r'/products') THEN 'product page'
      WHEN REGEXP_CONTAINS(landing_page_url, r'/checkout') THEN 'checkout page'
      WHEN REGEXP_CONTAINS(landing_page_url, r'/home') THEN 'homepage'
      ELSE 'other'
    END AS landing_page_type
  FROM landing_pages
),

base_with_landing AS (
  SELECT b.*, l.landing_page_type
  FROM base b
  LEFT JOIN landing_page_types l
  USING (session_id, user_pseudo_id)
),

metrics_by_landing AS (
  SELECT
    landing_page_type,
    COUNT(DISTINCT user_pseudo_id) AS users,
    COUNT(DISTINCT CASE WHEN event_name='page_view' THEN user_pseudo_id END) AS page_view_users,
    COUNT(DISTINCT CASE WHEN event_name='view_item' THEN user_pseudo_id END) AS view_item_users,
    COUNT(DISTINCT CASE WHEN event_name='add_to_cart' THEN user_pseudo_id END) AS add_to_cart_users,
    COUNT(DISTINCT CASE WHEN event_name='begin_checkout' THEN user_pseudo_id END) AS begin_checkout_users,
    COUNT(DISTINCT CASE WHEN event_name='purchase' THEN user_pseudo_id END) AS purchase_users,
    COUNT(DISTINCT session_id) AS total_sessions,
    COUNT(DISTINCT CASE WHEN session_engaged='1' THEN session_id END) AS engaged_sessions,
    SUM(revenue) AS revenue,
    SUM(engagement_time_msec) AS total_engagement_time
  FROM base_with_landing
  GROUP BY landing_page_type
)

SELECT
  landing_page_type,
  users,
  ROUND(SAFE_DIVIDE(view_item_users, NULLIF(page_view_users, 0)) * 100, 2) AS view_rate,
  ROUND((1 - SAFE_DIVIDE(view_item_users, NULLIF(page_view_users, 0))) * 100, 2) AS after_page_abandonment_rate,
  
  ROUND(SAFE_DIVIDE(add_to_cart_users, NULLIF(view_item_users, 0)) * 100, 2) AS add_to_cart_rate,
  ROUND((1 - SAFE_DIVIDE(add_to_cart_users, NULLIF(view_item_users, 0))) * 100, 2) AS after_view_abandonment_rate,

  ROUND(SAFE_DIVIDE(begin_checkout_users, NULLIF(add_to_cart_users, 0)) * 100, 2) AS checkout_rate,
  ROUND((1 - SAFE_DIVIDE(begin_checkout_users, NULLIF(add_to_cart_users, 0))) * 100, 2) AS cart_abandonment_rate,

  ROUND(SAFE_DIVIDE(purchase_users, NULLIF(begin_checkout_users, 0)) * 100, 2) AS purchase_rate,
  ROUND((1 - SAFE_DIVIDE(purchase_users, NULLIF(begin_checkout_users, 0))) * 100, 2) AS checkout_abandonment_rate,

  ROUND(SAFE_DIVIDE(revenue, NULLIF(purchase_users, 0)), 2) AS AOV,
  ROUND(SAFE_DIVIDE(revenue, NULLIF(users, 0)), 2) AS ARPU,
  ROUND(SAFE_DIVIDE(purchase_users, NULLIF(users, 0)) * 100, 2) AS conversion_rate,
  ROUND(SAFE_DIVIDE(total_sessions - engaged_sessions, NULLIF(total_sessions, 0)) * 100, 2) AS bounce_rate,
  ROUND(SAFE_DIVIDE(total_engagement_time, 1000 * NULLIF(total_sessions, 0)), 2) AS avg_session_duration_sec

FROM metrics_by_landing
ORDER BY users DESC;


 ```
## 🎯 Result:
![Screenshot_1](https://github.com/user-attachments/assets/d607451e-9e5f-47ee-99c0-52b8d6eda13f)

![Screenshot_2](https://github.com/user-attachments/assets/1fdf30f4-278e-4322-85a5-316a91974a2c)



## 🎯 Summary Key Findings:

      High Drop-offs After Landing:
      
      All landing pages experience significant drop-offs after landing, ranging from 75% (Homepage) to 85% (Checkout Page), showing a need for targeted improvements on key pages to 
      reduce visitor loss.

      Cart & Checkout Abandonment:
      
      Add-to-cart rates vary by page (10% Checkout Page to 25% Product Page), but cart abandonment is consistently high (60% to 80%), and checkout abandonment remains a concern (35% to 
      50%), indicating friction points in the purchase process.

      Conversion & Revenue Performance:
      
      Conversion rates improve progressively from Homepage (1.5%) to Checkout Page (2.5%), with Product Page showing strong performance (2.0%). Revenue and AOV peak at the Product 
       Checkout pages ($110K–$50K revenue; $85–$90 AOV). Other pages underperform with low conversion (1.2%) and revenue ($12,165).

      Engagement Opportunities:
      
      Bounce rates are moderately high across pages (30% to 40%), with average session durations between 55s and 75s, highlighting room to improve user engagement and reduce bounce, 
      especially on Checkout and Collection pages.
        
