-- How many users do we have?
select
    count(distinct user_id) as total_users
from DEV_DB.DBT_ANASCHAMBACHMECOM.STG_POSTGRES__USERS
-- 130 users

-- On average, how many orders do we receive per hour?
select
    round(count(*) / nullif(datediff(hour, min(created_at), max(created_at)), 0))
        as avg_orders_per_hour
from orders
-- 8 orders

-- On average, how long does an order take from being placed to being delivered?
select
 round(avg(datediff('day', created_at, delivered_at)))
    as avg_time_to_delivery
from orders
where delivered_at is not null
-- 4 days

-- How many users have only made one purchase? Two purchases? Three+ purchases?
with purchase_count as (
    select
        user_id
        , count(*) as purchase_count
    from orders
    group by user_id
)

select
    sum(case when purchase_count = 1 then 1 else 0 end)
        as one_purchase
    , sum(case when purchase_count = 2 then 1 else 0 end)
        as two_purchases
    , sum(case when purchase_count >= 3 then 1 else 0 end)
        as three_or_more_purchases
from purchase_count
-- ONE_PURCHASE	TWO_PURCHASES	THREE_OR_MORE_PURCHASES
--  25	         28	             71

-- On average, how many unique sessions do we have per hour?
with session_hours as (
    select
        date_trunc('hour', created_at) as session_hour
        , count(distinct session_id) as unique_sessions
    from events
    group by session_hour
)

, total_hours as (
    select count(distinct session_hour) as total_hours
    from session_hours
)

select
    round((sum(unique_sessions) / (select total_hours from total_hours)))
        as avg_unique_sessions_per_hour
from session_hours
-- 16 sessions