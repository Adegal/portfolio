===============================
--  SQL QUERY PORTFOLIO
 AIRLINE FLIGHT TICKET SALES
-- ============================================================

-- Schema
CREATE TABLE airline_customers (
    customer_id      INT PRIMARY KEY,
    full_name        VARCHAR(100),
    gender           VARCHAR(10),
    age              INT,
    country          VARCHAR(60),
    loyalty_tier     VARCHAR(20),  -- Bronze, Silver, Gold, Platinum
    email            VARCHAR(100)
);

CREATE TABLE flights (
    flight_id        INT PRIMARY KEY,
    flight_number    VARCHAR(10),
    origin           VARCHAR(60),
    destination      VARCHAR(60),
    departure_date   DATE,
    departure_time   TIME,
    arrival_time     TIME,
    aircraft_type    VARCHAR(40),
    total_seats      INT,
    available_seats  INT
);

CREATE TABLE airline_tickets (
    ticket_id        INT PRIMARY KEY,
    customer_id      INT REFERENCES airline_customers(customer_id),
    flight_id        INT REFERENCES flights(flight_id),
    booking_date     DATE,
    travel_class     VARCHAR(20),  -- Economy, Business, First
    ticket_price     DECIMAL(10,2),
    base_fare        DECIMAL(10,2),
    taxes_fees       DECIMAL(10,2),
    status           VARCHAR(20),  -- Completed, Cancelled, Pending
    payment_method   VARCHAR(30),
    seat_number      VARCHAR(5)
);

-- ── AIRLINE ANALYTICS QUERIES ─────────────────────────────────

-- 1.1  Sales by Day / Week / Month
SELECT
    DATE_TRUNC('day',   booking_date)   AS sale_day,
    DATE_TRUNC('week',  booking_date)   AS sale_week,
    DATE_TRUNC('month', booking_date)   AS sale_month,
    COUNT(ticket_id)                    AS tickets_sold,
    SUM(ticket_price)                   AS total_revenue,
    SUM(ticket_price - base_fare)       AS total_profit,
    ROUND(AVG(ticket_price), 2)         AS avg_ticket_price
FROM airline_tickets
WHERE status = 'Completed'
GROUP BY 1, 2, 3
ORDER BY sale_day;

-- 1.2  Customer Demographics & Gender Analysis
SELECT
    ac.gender,
    ac.country,
    CASE
        WHEN ac.age BETWEEN 18 AND 24 THEN '18-24'
        WHEN ac.age BETWEEN 25 AND 34 THEN '25-34'
        WHEN ac.age BETWEEN 35 AND 44 THEN '35-44'
        WHEN ac.age BETWEEN 45 AND 54 THEN '45-54'
        ELSE '55+'
    END                                 AS age_group,
    COUNT(at.ticket_id)                 AS total_bookings,
    SUM(at.ticket_price)                AS total_spend,
    ROUND(AVG(at.ticket_price), 2)      AS avg_spend
FROM airline_tickets at
JOIN airline_customers ac ON at.customer_id = ac.customer_id
WHERE at.status = 'Completed'
GROUP BY ac.gender, ac.country, age_group
ORDER BY total_spend DESC;

-- 1.3  Completed vs Incomplete Orders
SELECT
    status,
    COUNT(ticket_id)                                    AS ticket_count,
    ROUND(COUNT(ticket_id) * 100.0 / SUM(COUNT(ticket_id)) OVER (), 2) AS pct_of_total,
    SUM(ticket_price)                                   AS revenue_impact
FROM airline_tickets
GROUP BY status;

-- 1.4  Profit Margin by Travel Class
SELECT
    travel_class,
    COUNT(ticket_id)                                    AS tickets_sold,
    SUM(ticket_price)                                   AS total_revenue,
    SUM(base_fare)                                      AS total_cost,
    SUM(ticket_price - base_fare)                       AS gross_profit,
    ROUND((SUM(ticket_price - base_fare) / NULLIF(SUM(ticket_price), 0)) * 100, 2) AS profit_margin_pct
FROM airline_tickets
WHERE status = 'Completed'
GROUP BY travel_class
ORDER BY profit_margin_pct DESC;

-- 1.5  Seat Occupancy / Inventory Turnover Ratio
SELECT
    f.flight_number,
    f.origin,
    f.destination,
    f.departure_date,
    f.total_seats,
    COUNT(at.ticket_id)                                 AS seats_sold,
    f.total_seats - COUNT(at.ticket_id)                 AS seats_remaining,
    ROUND(COUNT(at.ticket_id) * 100.0 / f.total_seats, 2) AS occupancy_rate_pct,
    SUM(at.ticket_price)                                AS flight_revenue
FROM flights f
LEFT JOIN airline_tickets at ON f.flight_id = at.flight_id AND at.status = 'Completed'
GROUP BY f.flight_id, f.flight_number, f.origin, f.destination, f.departure_date, f.total_seats
ORDER BY occupancy_rate_pct DESC;

-- 1.6  Top Routes by Revenue
SELECT
    f.origin,
    f.destination,
    COUNT(at.ticket_id)                                 AS bookings,
    SUM(at.ticket_price)                                AS total_revenue,
    ROUND(AVG(at.ticket_price), 2)                      AS avg_fare,
    ROUND((SUM(at.ticket_price - at.base_fare) / NULLIF(SUM(at.ticket_price),0)) * 100, 2) AS route_margin_pct
FROM airline_tickets at
JOIN flights f ON at.flight_id = f.flight_id
WHERE at.status = 'Completed'
GROUP BY f.origin, f.destination
ORDER BY total_revenue DESC
LIMIT 10;

-- 1.7  Loyalty Tier Booking Frequency
SELECT
    ac.loyalty_tier,
    COUNT(DISTINCT ac.customer_id)                      AS unique_customers,
    COUNT(at.ticket_id)                                 AS total_bookings,
    ROUND(COUNT(at.ticket_id) * 1.0 / NULLIF(COUNT(DISTINCT ac.customer_id), 0), 2) AS avg_bookings_per_customer,
    SUM(at.ticket_price)                                AS total_revenue
FROM airline_customers ac
JOIN airline_tickets at ON ac.customer_id = at.customer_id
WHERE at.status = 'Completed'
GROUP BY ac.loyalty_tier
ORDER BY total_revenue DESC;


