## Website Data Analysis Project

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
 
 with flat_data as (
  SELECT  
    user_pseudo_id,
    event_name,
    event_date,
    concat(user_pseudo_id,(select value.int_value from unnest(event_params) where key='ga_session_id')) as session_id,
    (select value.string_value from unnest(event_params) where key='session_engaged') as session_engaged,
    (select value.int_value from unnest (event_params) where key ='engagement_time_msec') as engagment_time_mesc,
    ecommerce.purchase_revenue as revenue

FROM `bigquery-public-data.ga4_obfuscated_sample_ecommerce.events_*`
where parse_date('%Y%m%d', event_date) between start_date and end_date
 ),

metric as (
 select
     count(distinct user_pseudo_id) as total_users,
     count(distinct case when event_name='page_view' then user_pseudo_id end) as page_view,
     count(distinct case when event_name='view_item' then user_pseudo_id end) as view_item,
     count(distinct case when event_name='add_to_cart' then user_pseudo_id end) as add_to_cart,
     count(distinct case when event_name='begin_checkout' then user_pseudo_id end) as begin_checkout,
     count(distinct case when event_name='purchase' then user_pseudo_id end) as purchase,
     sum(revenue) as total_revenue
 from flat_data
),

metric_cal as (
        select
            total_users,
            round(safe_divide(view_item,nullif(page_view,0))*100,2) as view_rate,
            100-round(safe_divide(view_item,nullif(page_view,0))*100,2) as drop_lp_product_page,

            round(safe_divide(add_to_cart,nullif(view_item,0))*100,2) as add_to_cart_rate,
            100- round(safe_divide(add_to_cart,nullif(view_item,0))*100,2) as drop_product_cart,

            round(safe_divide(begin_checkout,nullif(add_to_cart,0))*100,2) as checkout_rate,
            100-round(safe_divide(begin_checkout,nullif(add_to_cart,0))*100,2) as drop_cart_checkout,

            round(safe_divide(purchase,nullif(begin_checkout,0))*100,2) as purchase_rate,
            100 - round(safe_divide(purchase,nullif(begin_checkout,0))*100,2) as drop_checkout_purchase,

            round(safe_divide(purchase,nullif(total_users,0))*100,2) as conversion_rate,

            total_revenue,

            round(safe_divide(total_revenue,nullif(purchase,0)),2) as AOV,

            safe_divide(total_revenue,nullif(total_users,0)) as ARPU

      from metric

),

bounce_session_duration as (
    select
       round(safe_divide(count(distinct session_id)-count(distinct case when session_engaged='1' then session_id end), 
        count(distinct session_id))*100,2) as bounce_rate,

       safe_divide(sum(engagment_time_mesc), 1000 * count(distinct session_id)) as avg_session_duration_sec
from flat_data
)

select
   m.total_users,
   m.view_rate,
   m.drop_lp_product_page,
   m.add_to_cart_rate,
   m.drop_product_cart,
   m.checkout_rate,
   m.drop_cart_checkout,
   m.purchase_rate,
   m.drop_checkout_purchase,
   m.conversion_rate,
   m.AOV,
   m.ARPU,
   b.bounce_rate,
   b.avg_session_duration_sec

from metric_cal m, bounce_session_duration b;

 ```

## 🎯 Query Result:

![Screenshot_5](https://github.com/user-attachments/assets/ad6d3573-adb9-4ef2-918b-bc991a9352e4)

![Screenshot_6](https://github.com/user-attachments/assets/05f959e7-c973-489a-828d-6c48858298ed)


## 🎯 Summary Key Findings:


      Strong product engagement: 22.7% view rate, but 77.3% drop after landing → needs homepage optimization.
      
      Healthy add-to-cart rate (20.48%), but high cart (77.44%) and checkout (45.49%) abandonment → opportunity to streamline purchase experience.
      
      AOV is solid at $81.96, but ARPU is low ($1.34) → boost conversions to lift revenue per user.
      
      Bounce rate (32.8%) and session duration (70s) suggest room to improve site stickiness and engagement.


