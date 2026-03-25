[sql_portfolio.sql](https://github.com/user-attachments/files/26256300/sql_portfolio.sql)
-- ============================================================
--  SQL QUERY PORTFOLIO
--  7 Real-World Projects with Analytics & Business Intelligence
--  Author: [Your Name]  |  Portfolio Version 1.0
-- ============================================================

-- ============================================================
--  PROJECT 1: AIRLINE FLIGHT TICKET SALES
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


-- ============================================================
--  PROJECT 2: HEALTHCARE APPOINTMENT MANAGEMENT
-- ============================================================

CREATE TABLE patients (
    patient_id       INT PRIMARY KEY,
    full_name        VARCHAR(100),
    gender           VARCHAR(10),
    date_of_birth    DATE,
    city             VARCHAR(60),
    insurance_type   VARCHAR(40),  -- Private, Government, None
    blood_group      VARCHAR(5)
);

CREATE TABLE doctors (
    doctor_id        INT PRIMARY KEY,
    full_name        VARCHAR(100),
    specialty        VARCHAR(60),
    department       VARCHAR(60),
    years_experience INT,
    consultation_fee DECIMAL(8,2)
);

CREATE TABLE appointments (
    appointment_id   INT PRIMARY KEY,
    patient_id       INT REFERENCES patients(patient_id),
    doctor_id        INT REFERENCES doctors(doctor_id),
    appointment_date DATE,
    appointment_time TIME,
    appointment_type VARCHAR(30),  -- Walk-in, Scheduled, Emergency
    status           VARCHAR(20),  -- Completed, No-Show, Cancelled, Pending
    diagnosis_code   VARCHAR(20),
    treatment_cost   DECIMAL(10,2),
    insurance_paid   DECIMAL(10,2),
    patient_paid     DECIMAL(10,2),
    follow_up_req    BOOLEAN
);

-- ── HEALTHCARE ANALYTICS QUERIES ──────────────────────────────

-- 2.1  Appointments by Day / Week / Month
SELECT
    DATE_TRUNC('day',   appointment_date) AS appt_day,
    DATE_TRUNC('week',  appointment_date) AS appt_week,
    DATE_TRUNC('month', appointment_date) AS appt_month,
    COUNT(appointment_id)                 AS total_appointments,
    COUNT(CASE WHEN status = 'Completed' THEN 1 END) AS completed,
    COUNT(CASE WHEN status = 'No-Show'   THEN 1 END) AS no_shows,
    COUNT(CASE WHEN status = 'Cancelled' THEN 1 END) AS cancellations,
    SUM(treatment_cost)                   AS total_revenue
FROM appointments
GROUP BY 1, 2, 3
ORDER BY appt_day;

-- 2.2  Patient Demographics
SELECT
    p.gender,
    CASE
        WHEN EXTRACT(YEAR FROM AGE(p.date_of_birth)) BETWEEN 0  AND 17 THEN 'Under 18'
        WHEN EXTRACT(YEAR FROM AGE(p.date_of_birth)) BETWEEN 18 AND 34 THEN '18-34'
        WHEN EXTRACT(YEAR FROM AGE(p.date_of_birth)) BETWEEN 35 AND 54 THEN '35-54'
        WHEN EXTRACT(YEAR FROM AGE(p.date_of_birth)) BETWEEN 55 AND 69 THEN '55-69'
        ELSE '70+'
    END                                   AS age_group,
    p.insurance_type,
    p.city,
    COUNT(a.appointment_id)               AS total_visits,
    ROUND(AVG(a.treatment_cost), 2)       AS avg_treatment_cost,
    SUM(a.patient_paid)                   AS total_patient_spend
FROM patients p
JOIN appointments a ON p.patient_id = a.patient_id
WHERE a.status = 'Completed'
GROUP BY p.gender, age_group, p.insurance_type, p.city
ORDER BY total_visits DESC;

-- 2.3  Completed vs Incomplete Appointments
SELECT
    status,
    COUNT(appointment_id)                               AS count,
    ROUND(COUNT(appointment_id) * 100.0 / SUM(COUNT(appointment_id)) OVER (), 2) AS percentage,
    SUM(treatment_cost)                                 AS revenue_value
FROM appointments
GROUP BY status
ORDER BY count DESC;

-- 2.4  Doctor Performance & Revenue
SELECT
    d.full_name,
    d.specialty,
    COUNT(a.appointment_id)                             AS total_appts,
    COUNT(CASE WHEN a.status = 'Completed' THEN 1 END)  AS completed_appts,
    ROUND(COUNT(CASE WHEN a.status = 'Completed' THEN 1 END) * 100.0 / NULLIF(COUNT(a.appointment_id),0), 2) AS completion_rate,
    SUM(CASE WHEN a.status = 'Completed' THEN a.treatment_cost ELSE 0 END) AS total_revenue,
    ROUND(AVG(CASE WHEN a.status = 'Completed' THEN a.treatment_cost END), 2) AS avg_revenue_per_appt
FROM doctors d
JOIN appointments a ON d.doctor_id = a.doctor_id
GROUP BY d.doctor_id, d.full_name, d.specialty
ORDER BY total_revenue DESC;

-- 2.5  No-Show Rate by Demographics (churn risk)
SELECT
    p.gender,
    p.insurance_type,
    p.city,
    COUNT(a.appointment_id)                             AS total_scheduled,
    COUNT(CASE WHEN a.status = 'No-Show' THEN 1 END)   AS no_shows,
    ROUND(COUNT(CASE WHEN a.status = 'No-Show' THEN 1 END) * 100.0 / NULLIF(COUNT(a.appointment_id),0), 2) AS no_show_rate_pct
FROM patients p
JOIN appointments a ON p.patient_id = a.patient_id
GROUP BY p.gender, p.insurance_type, p.city
ORDER BY no_show_rate_pct DESC;

-- 2.6  Insurance Revenue vs Out-of-Pocket
SELECT
    p.insurance_type,
    COUNT(a.appointment_id)                             AS appointments,
    SUM(a.treatment_cost)                               AS total_cost,
    SUM(a.insurance_paid)                               AS insurance_covered,
    SUM(a.patient_paid)                                 AS patient_paid,
    ROUND(SUM(a.insurance_paid) * 100.0 / NULLIF(SUM(a.treatment_cost),0), 2) AS insurance_coverage_pct
FROM patients p
JOIN appointments a ON p.patient_id = a.patient_id
WHERE a.status = 'Completed'
GROUP BY p.insurance_type;

-- 2.7  Follow-Up Appointment Rate by Specialty
SELECT
    d.specialty,
    COUNT(a.appointment_id)                             AS total_completed,
    COUNT(CASE WHEN a.follow_up_req = TRUE THEN 1 END)  AS follow_ups_required,
    ROUND(COUNT(CASE WHEN a.follow_up_req = TRUE THEN 1 END) * 100.0 / NULLIF(COUNT(a.appointment_id),0), 2) AS follow_up_rate_pct
FROM doctors d
JOIN appointments a ON d.doctor_id = a.doctor_id
WHERE a.status = 'Completed'
GROUP BY d.specialty
ORDER BY follow_up_rate_pct DESC;


-- ============================================================
--  PROJECT 3: CINEMA TICKET SALES
-- ============================================================

CREATE TABLE cinema_customers (
    customer_id      INT PRIMARY KEY,
    full_name        VARCHAR(100),
    gender           VARCHAR(10),
    age              INT,
    city             VARCHAR(60),
    membership_type  VARCHAR(20)   -- Standard, Premium, VIP
);

CREATE TABLE movies (
    movie_id         INT PRIMARY KEY,
    title            VARCHAR(150),
    genre            VARCHAR(40),
    rating           VARCHAR(10),  -- G, PG, PG-13, R
    duration_mins    INT,
    release_date     DATE,
    distributor      VARCHAR(80)
);

CREATE TABLE screenings (
    screening_id     INT PRIMARY KEY,
    movie_id         INT REFERENCES movies(movie_id),
    screen_number    INT,
    show_date        DATE,
    show_time        TIME,
    format           VARCHAR(20),  -- 2D, 3D, IMAX
    total_seats      INT,
    ticket_price     DECIMAL(8,2),
    cost_per_seat    DECIMAL(8,2)
);

CREATE TABLE cinema_tickets (
    ticket_id        INT PRIMARY KEY,
    customer_id      INT REFERENCES cinema_customers(customer_id),
    screening_id     INT REFERENCES screenings(screening_id),
    purchase_date    DATE,
    seat_category    VARCHAR(20),  -- Standard, Premium, VIP Box
    price_paid       DECIMAL(8,2),
    concession_spend DECIMAL(8,2),
    status           VARCHAR(20),  -- Completed, Refunded, No-Show
    booking_channel  VARCHAR(20)   -- Online, In-Person, App
);

-- ── CINEMA ANALYTICS QUERIES ──────────────────────────────────

-- 3.1  Box Office Revenue by Day / Week / Month
SELECT
    DATE_TRUNC('day',   ct.purchase_date)  AS sale_day,
    DATE_TRUNC('week',  ct.purchase_date)  AS sale_week,
    DATE_TRUNC('month', ct.purchase_date)  AS sale_month,
    COUNT(ct.ticket_id)                    AS tickets_sold,
    SUM(ct.price_paid)                     AS ticket_revenue,
    SUM(ct.concession_spend)               AS concession_revenue,
    SUM(ct.price_paid + ct.concession_spend) AS total_revenue
FROM cinema_tickets ct
WHERE ct.status = 'Completed'
GROUP BY 1, 2, 3
ORDER BY sale_day;

-- 3.2  Customer Demographics
SELECT
    cc.gender,
    CASE
        WHEN cc.age BETWEEN 13 AND 17 THEN '13-17'
        WHEN cc.age BETWEEN 18 AND 24 THEN '18-24'
        WHEN cc.age BETWEEN 25 AND 34 THEN '25-34'
        WHEN cc.age BETWEEN 35 AND 49 THEN '35-49'
        ELSE '50+'
    END                                    AS age_group,
    cc.city,
    cc.membership_type,
    COUNT(ct.ticket_id)                    AS tickets_purchased,
    SUM(ct.price_paid + ct.concession_spend) AS total_spend,
    ROUND(AVG(ct.price_paid + ct.concession_spend), 2) AS avg_spend_per_visit
FROM cinema_customers cc
JOIN cinema_tickets ct ON cc.customer_id = ct.customer_id
WHERE ct.status = 'Completed'
GROUP BY cc.gender, age_group, cc.city, cc.membership_type
ORDER BY total_spend DESC;

-- 3.3  Completed vs Refunded vs No-Show
SELECT
    status,
    COUNT(ticket_id)                                    AS count,
    ROUND(COUNT(ticket_id) * 100.0 / SUM(COUNT(ticket_id)) OVER (), 2) AS pct_share,
    SUM(price_paid)                                     AS revenue_value
FROM cinema_tickets
GROUP BY status;

-- 3.4  Profit Margin by Movie & Format
SELECT
    m.title,
    m.genre,
    s.format,
    COUNT(ct.ticket_id)                                 AS tickets_sold,
    SUM(ct.price_paid)                                  AS total_revenue,
    SUM(s.cost_per_seat * COUNT(ct.ticket_id))          AS estimated_cost,
    SUM(ct.price_paid) - SUM(s.cost_per_seat)           AS gross_profit,
    ROUND((SUM(ct.price_paid) - SUM(s.cost_per_seat)) / NULLIF(SUM(ct.price_paid),0) * 100, 2) AS profit_margin_pct
FROM cinema_tickets ct
JOIN screenings s  ON ct.screening_id = s.screening_id
JOIN movies m      ON s.movie_id = m.movie_id
WHERE ct.status = 'Completed'
GROUP BY m.title, m.genre, s.format
ORDER BY profit_margin_pct DESC;

-- 3.5  Seat Occupancy / Turnover Ratio
SELECT
    m.title,
    s.show_date,
    s.format,
    s.total_seats,
    COUNT(ct.ticket_id)                                 AS seats_sold,
    ROUND(COUNT(ct.ticket_id) * 1.0 / s.total_seats, 4) AS turnover_ratio,
    ROUND(COUNT(ct.ticket_id) * 100.0 / s.total_seats, 2) AS occupancy_pct
FROM screenings s
JOIN movies m ON s.movie_id = m.movie_id
LEFT JOIN cinema_tickets ct ON s.screening_id = ct.screening_id AND ct.status = 'Completed'
GROUP BY m.title, s.show_date, s.format, s.total_seats
ORDER BY occupancy_pct DESC;

-- 3.6  Top Genres by Revenue and Footfall
SELECT
    m.genre,
    COUNT(ct.ticket_id)                                 AS total_tickets,
    SUM(ct.price_paid)                                  AS ticket_revenue,
    SUM(ct.concession_spend)                            AS concession_revenue,
    ROUND(AVG(ct.concession_spend), 2)                  AS avg_concession_per_head
FROM cinema_tickets ct
JOIN screenings s ON ct.screening_id = s.screening_id
JOIN movies m     ON s.movie_id = m.movie_id
WHERE ct.status = 'Completed'
GROUP BY m.genre
ORDER BY total_tickets DESC;

-- 3.7  Booking Channel Efficiency
SELECT
    booking_channel,
    COUNT(ticket_id)                                    AS bookings,
    SUM(price_paid)                                     AS revenue,
    ROUND(AVG(price_paid), 2)                           AS avg_ticket_value,
    COUNT(CASE WHEN status = 'Refunded' THEN 1 END)     AS refunds,
    ROUND(COUNT(CASE WHEN status = 'Refunded' THEN 1 END) * 100.0 / COUNT(ticket_id), 2) AS refund_rate_pct
FROM cinema_tickets
GROUP BY booking_channel
ORDER BY revenue DESC;


-- ============================================================
--  PROJECT 4: FACEBOOK ADS – WIRELESS SERVICE COMPANY
-- ============================================================

CREATE TABLE ad_campaigns (
    campaign_id      INT PRIMARY KEY,
    campaign_name    VARCHAR(120),
    objective        VARCHAR(60),  -- Lead Generation, Awareness, Conversion
    start_date       DATE,
    end_date         DATE,
    total_budget     DECIMAL(12,2),
    status           VARCHAR(20)   -- Active, Paused, Completed
);

CREATE TABLE ad_sets (
    ad_set_id        INT PRIMARY KEY,
    campaign_id      INT REFERENCES ad_campaigns(campaign_id),
    ad_set_name      VARCHAR(120),
    target_gender    VARCHAR(10),
    target_age_min   INT,
    target_age_max   INT,
    target_location  VARCHAR(80),
    daily_budget     DECIMAL(10,2),
    bid_strategy     VARCHAR(40)
);

CREATE TABLE ad_performance (
    perf_id          INT PRIMARY KEY,
    ad_set_id        INT REFERENCES ad_sets(ad_set_id),
    report_date      DATE,
    impressions      INT,
    clicks           INT,
    leads_generated  INT,
    spend            DECIMAL(10,2),
    reach            INT,
    frequency        DECIMAL(6,2),
    conversions      INT           -- leads that became customers
);

CREATE TABLE leads (
    lead_id          INT PRIMARY KEY,
    ad_set_id        INT REFERENCES ad_sets(ad_set_id),
    captured_date    DATE,
    gender           VARCHAR(10),
    age              INT,
    location         VARCHAR(80),
    plan_interest    VARCHAR(60),  -- Basic, Standard, Premium, Business
    lead_status      VARCHAR(30),  -- New, Contacted, Qualified, Converted, Lost
    contract_value   DECIMAL(10,2)
);

-- ── FACEBOOK ADS ANALYTICS QUERIES ───────────────────────────

-- 4.1  Campaign Performance by Day / Week / Month
SELECT
    DATE_TRUNC('day',   ap.report_date)    AS report_day,
    DATE_TRUNC('week',  ap.report_date)    AS report_week,
    DATE_TRUNC('month', ap.report_date)    AS report_month,
    SUM(ap.impressions)                    AS total_impressions,
    SUM(ap.clicks)                         AS total_clicks,
    SUM(ap.leads_generated)                AS total_leads,
    SUM(ap.conversions)                    AS total_conversions,
    SUM(ap.spend)                          AS total_spend,
    ROUND(SUM(ap.clicks) * 100.0 / NULLIF(SUM(ap.impressions),0), 4) AS ctr_pct,
    ROUND(SUM(ap.spend) / NULLIF(SUM(ap.leads_generated),0), 2)      AS cost_per_lead
FROM ad_performance ap
GROUP BY 1, 2, 3
ORDER BY report_day;

-- 4.2  Click-Through Rate by Gender & Age Demographics
SELECT
    ads.target_gender                      AS gender,
    ads.target_age_min || '-' || ads.target_age_max AS age_range,
    ads.target_location                    AS location,
    SUM(ap.impressions)                    AS impressions,
    SUM(ap.clicks)                         AS clicks,
    SUM(ap.leads_generated)                AS leads,
    ROUND(SUM(ap.clicks) * 100.0 / NULLIF(SUM(ap.impressions),0), 4) AS ctr_pct,
    ROUND(SUM(ap.leads_generated) * 100.0 / NULLIF(SUM(ap.clicks),0), 2) AS lead_conversion_rate_pct
FROM ad_sets ads
JOIN ad_performance ap ON ads.ad_set_id = ap.ad_set_id
GROUP BY ads.target_gender, age_range, ads.target_location
ORDER BY ctr_pct DESC;

-- 4.3  Lead Status Funnel
SELECT
    lead_status,
    COUNT(lead_id)                                      AS lead_count,
    ROUND(COUNT(lead_id) * 100.0 / SUM(COUNT(lead_id)) OVER (), 2) AS pct_of_total,
    SUM(CASE WHEN lead_status = 'Converted' THEN contract_value ELSE 0 END) AS contracted_revenue
FROM leads
GROUP BY lead_status
ORDER BY lead_count DESC;

-- 4.4  Cost Per Acquisition & ROAS by Campaign
SELECT
    ac.campaign_name,
    ac.objective,
    SUM(ap.spend)                                       AS total_spend,
    SUM(ap.leads_generated)                             AS total_leads,
    SUM(ap.conversions)                                 AS total_conversions,
    ROUND(SUM(ap.spend) / NULLIF(SUM(ap.leads_generated),0), 2) AS cost_per_lead,
    ROUND(SUM(ap.spend) / NULLIF(SUM(ap.conversions),0), 2)     AS cost_per_acquisition,
    SUM(l.contract_value)                               AS total_contract_value,
    ROUND(SUM(l.contract_value) / NULLIF(SUM(ap.spend),0), 2)  AS roas
FROM ad_campaigns ac
JOIN ad_sets ads      ON ac.campaign_id = ads.campaign_id
JOIN ad_performance ap ON ads.ad_set_id = ap.ad_set_id
LEFT JOIN leads l     ON ads.ad_set_id = l.ad_set_id AND l.lead_status = 'Converted'
GROUP BY ac.campaign_id, ac.campaign_name, ac.objective
ORDER BY roas DESC;

-- 4.5  Lead Demographics (Gender & Age)
SELECT
    l.gender,
    CASE
        WHEN l.age BETWEEN 18 AND 24 THEN '18-24'
        WHEN l.age BETWEEN 25 AND 34 THEN '25-34'
        WHEN l.age BETWEEN 35 AND 44 THEN '35-44'
        WHEN l.age BETWEEN 45 AND 54 THEN '45-54'
        ELSE '55+'
    END                                                AS age_group,
    l.plan_interest,
    COUNT(l.lead_id)                                   AS total_leads,
    COUNT(CASE WHEN l.lead_status = 'Converted' THEN 1 END) AS converted,
    ROUND(COUNT(CASE WHEN l.lead_status = 'Converted' THEN 1 END) * 100.0 / NULLIF(COUNT(l.lead_id),0), 2) AS conversion_rate_pct,
    SUM(CASE WHEN l.lead_status = 'Converted' THEN l.contract_value ELSE 0 END) AS revenue
FROM leads l
GROUP BY l.gender, age_group, l.plan_interest
ORDER BY conversion_rate_pct DESC;

-- 4.6  Budget Utilisation & Profit Margin
SELECT
    ac.campaign_name,
    ac.total_budget,
    SUM(ap.spend)                                       AS actual_spend,
    ROUND(SUM(ap.spend) * 100.0 / NULLIF(ac.total_budget,0), 2) AS budget_utilisation_pct,
    SUM(l.contract_value)                               AS gross_revenue,
    SUM(l.contract_value) - SUM(ap.spend)               AS net_profit,
    ROUND((SUM(l.contract_value) - SUM(ap.spend)) / NULLIF(SUM(l.contract_value),0) * 100, 2) AS profit_margin_pct
FROM ad_campaigns ac
JOIN ad_sets ads ON ac.campaign_id = ads.campaign_id
JOIN ad_performance ap ON ads.ad_set_id = ap.ad_set_id
LEFT JOIN leads l ON ads.ad_set_id = l.ad_set_id AND l.lead_status = 'Converted'
GROUP BY ac.campaign_id, ac.campaign_name, ac.total_budget
ORDER BY net_profit DESC;

-- 4.7  Frequency vs CTR (Ad Fatigue Analysis)
SELECT
    ads.ad_set_name,
    ROUND(AVG(ap.frequency), 2)                        AS avg_frequency,
    ROUND(SUM(ap.clicks) * 100.0 / NULLIF(SUM(ap.impressions),0), 4) AS avg_ctr_pct,
    SUM(ap.leads_generated)                            AS total_leads
FROM ad_sets ads
JOIN ad_performance ap ON ads.ad_set_id = ap.ad_set_id
GROUP BY ads.ad_set_id, ads.ad_set_name
ORDER BY avg_frequency DESC;


-- ============================================================
--  PROJECT 5: INVENTORY HANDLING & SALES – E-COMMERCE
-- ============================================================

CREATE TABLE ecom_customers (
    customer_id      INT PRIMARY KEY,
    full_name        VARCHAR(100),
    gender           VARCHAR(10),
    age              INT,
    city             VARCHAR(60),
    country          VARCHAR(60),
    customer_segment VARCHAR(20)  -- New, Returning, VIP
);

CREATE TABLE products (
    product_id       INT PRIMARY KEY,
    product_name     VARCHAR(150),
    category         VARCHAR(60),
    sub_category     VARCHAR(60),
    brand            VARCHAR(80),
    unit_cost        DECIMAL(10,2),
    unit_price       DECIMAL(10,2),
    reorder_level    INT,
    supplier_id      INT
);

CREATE TABLE inventory (
    inventory_id     INT PRIMARY KEY,
    product_id       INT REFERENCES products(product_id),
    stock_date       DATE,
    quantity_in      INT,
    quantity_out     INT,
    closing_stock    INT,
    warehouse        VARCHAR(40)
);

CREATE TABLE ecom_orders (
    order_id         INT PRIMARY KEY,
    customer_id      INT REFERENCES ecom_customers(customer_id),
    order_date       DATE,
    status           VARCHAR(20),  -- Completed, Returned, Cancelled, Pending
    payment_method   VARCHAR(30),
    shipping_cost    DECIMAL(8,2),
    discount_pct     DECIMAL(5,2)
);

CREATE TABLE order_items (
    item_id          INT PRIMARY KEY,
    order_id         INT REFERENCES ecom_orders(order_id),
    product_id       INT REFERENCES products(product_id),
    quantity         INT,
    unit_price       DECIMAL(10,2),
    unit_cost        DECIMAL(10,2),
    discount_amount  DECIMAL(10,2)
);

-- ── E-COMMERCE ANALYTICS QUERIES ─────────────────────────────

-- 5.1  Sales by Day / Week / Month
SELECT
    DATE_TRUNC('day',   eo.order_date)     AS sale_day,
    DATE_TRUNC('week',  eo.order_date)     AS sale_week,
    DATE_TRUNC('month', eo.order_date)     AS sale_month,
    COUNT(DISTINCT eo.order_id)            AS total_orders,
    SUM(oi.quantity * oi.unit_price - oi.discount_amount) AS gross_revenue,
    SUM(oi.quantity * oi.unit_cost)        AS total_cost,
    SUM((oi.unit_price - oi.unit_cost) * oi.quantity - oi.discount_amount) AS gross_profit
FROM ecom_orders eo
JOIN order_items oi ON eo.order_id = oi.order_id
WHERE eo.status = 'Completed'
GROUP BY 1, 2, 3
ORDER BY sale_day;

-- 5.2  Customer Demographics & Gender
SELECT
    ec.gender,
    CASE
        WHEN ec.age BETWEEN 18 AND 24 THEN '18-24'
        WHEN ec.age BETWEEN 25 AND 34 THEN '25-34'
        WHEN ec.age BETWEEN 35 AND 44 THEN '35-44'
        WHEN ec.age BETWEEN 45 AND 54 THEN '45-54'
        ELSE '55+'
    END                                   AS age_group,
    ec.country,
    ec.customer_segment,
    COUNT(DISTINCT eo.order_id)           AS total_orders,
    SUM(oi.quantity * oi.unit_price - oi.discount_amount) AS total_spend,
    ROUND(AVG(oi.quantity * oi.unit_price - oi.discount_amount), 2) AS avg_order_value
FROM ecom_customers ec
JOIN ecom_orders eo  ON ec.customer_id = eo.customer_id
JOIN order_items oi  ON eo.order_id = oi.order_id
WHERE eo.status = 'Completed'
GROUP BY ec.gender, age_group, ec.country, ec.customer_segment
ORDER BY total_spend DESC;

-- 5.3  Completed vs Incomplete Orders
SELECT
    status,
    COUNT(order_id)                                     AS order_count,
    ROUND(COUNT(order_id) * 100.0 / SUM(COUNT(order_id)) OVER (), 2) AS pct_share
FROM ecom_orders
GROUP BY status
ORDER BY order_count DESC;

-- 5.4  Profit Margin by Category
SELECT
    p.category,
    p.sub_category,
    SUM(oi.quantity)                                    AS units_sold,
    SUM(oi.quantity * oi.unit_price - oi.discount_amount) AS total_revenue,
    SUM(oi.quantity * oi.unit_cost)                     AS total_cost,
    SUM((oi.unit_price - oi.unit_cost) * oi.quantity - oi.discount_amount) AS gross_profit,
    ROUND(SUM((oi.unit_price - oi.unit_cost) * oi.quantity - oi.discount_amount)
        / NULLIF(SUM(oi.quantity * oi.unit_price - oi.discount_amount),0) * 100, 2) AS profit_margin_pct
FROM order_items oi
JOIN products p       ON oi.product_id = p.product_id
JOIN ecom_orders eo   ON oi.order_id = eo.order_id
WHERE eo.status = 'Completed'
GROUP BY p.category, p.sub_category
ORDER BY profit_margin_pct DESC;

-- 5.5  Inventory Turnover Ratio
SELECT
    p.product_name,
    p.category,
    SUM(i.quantity_out)                                 AS total_units_sold,
    ROUND(AVG(i.closing_stock), 0)                      AS avg_inventory,
    ROUND(SUM(i.quantity_out) * 1.0 / NULLIF(AVG(i.closing_stock),0), 2) AS turnover_ratio,
    SUM(i.quantity_out) * p.unit_cost                   AS cogs
FROM inventory i
JOIN products p ON i.product_id = p.product_id
GROUP BY p.product_id, p.product_name, p.category, p.unit_cost
ORDER BY turnover_ratio DESC;

-- 5.6  Low Stock Alert (Reorder Trigger)
SELECT
    p.product_id,
    p.product_name,
    p.category,
    i.closing_stock                                     AS current_stock,
    p.reorder_level,
    CASE WHEN i.closing_stock <= p.reorder_level THEN 'REORDER NOW' ELSE 'OK' END AS stock_status
FROM products p
JOIN (
    SELECT product_id, closing_stock
    FROM inventory
    WHERE stock_date = (SELECT MAX(stock_date) FROM inventory)
) i ON p.product_id = i.product_id
ORDER BY stock_status DESC, i.closing_stock ASC;

-- 5.7  Customer Lifetime Value & Retention
SELECT
    ec.customer_id,
    ec.full_name,
    ec.customer_segment,
    COUNT(DISTINCT eo.order_id)                         AS total_orders,
    SUM(oi.quantity * oi.unit_price - oi.discount_amount) AS lifetime_value,
    MIN(eo.order_date)                                  AS first_order,
    MAX(eo.order_date)                                  AS last_order,
    CURRENT_DATE - MAX(eo.order_date)                   AS days_since_last_order
FROM ecom_customers ec
JOIN ecom_orders eo  ON ec.customer_id = eo.customer_id
JOIN order_items oi  ON eo.order_id = oi.order_id
WHERE eo.status = 'Completed'
GROUP BY ec.customer_id, ec.full_name, ec.customer_segment
ORDER BY lifetime_value DESC;


-- ============================================================
--  PROJECT 6: REAL ESTATE PROPERTY SALES
-- ============================================================

CREATE TABLE property_buyers (
    buyer_id         INT PRIMARY KEY,
    full_name        VARCHAR(100),
    gender           VARCHAR(10),
    age              INT,
    income_bracket   VARCHAR(30),
    city             VARCHAR(60),
    buyer_type       VARCHAR(20)  -- First-Time, Investor, Upgrader
);

CREATE TABLE properties (
    property_id      INT PRIMARY KEY,
    property_type    VARCHAR(40),  -- Condo, House, Townhouse, Commercial
    city             VARCHAR(60),
    neighbourhood    VARCHAR(80),
    bedrooms         INT,
    bathrooms        INT,
    sqft             INT,
    listing_price    DECIMAL(14,2),
    cost_basis       DECIMAL(14,2),
    listing_date     DATE,
    status           VARCHAR(20)  -- Available, Under Contract, Sold
);

CREATE TABLE property_sales (
    sale_id          INT PRIMARY KEY,
    property_id      INT REFERENCES properties(property_id),
    buyer_id         INT REFERENCES property_buyers(buyer_id),
    agent_id         INT,
    listing_date     DATE,
    sale_date        DATE,
    sale_price       DECIMAL(14,2),
    commission_pct   DECIMAL(5,2),
    closing_costs    DECIMAL(10,2),
    financing_type   VARCHAR(30),  -- Cash, Mortgage, Bridge Loan
    status           VARCHAR(20)   -- Completed, Fallen Through, Pending
);

-- ── REAL ESTATE ANALYTICS QUERIES ────────────────────────────

-- 6.1  Sales Revenue by Day / Week / Month
SELECT
    DATE_TRUNC('day',   sale_date)         AS sale_day,
    DATE_TRUNC('week',  sale_date)         AS sale_week,
    DATE_TRUNC('month', sale_date)         AS sale_month,
    COUNT(sale_id)                         AS properties_sold,
    SUM(sale_price)                        AS total_sales_value,
    ROUND(AVG(sale_price), 2)              AS avg_sale_price,
    SUM(sale_price * commission_pct / 100) AS commission_earned
FROM property_sales
WHERE status = 'Completed'
GROUP BY 1, 2, 3
ORDER BY sale_day;

-- 6.2  Buyer Demographics
SELECT
    pb.gender,
    CASE
        WHEN pb.age BETWEEN 25 AND 34 THEN '25-34'
        WHEN pb.age BETWEEN 35 AND 44 THEN '35-44'
        WHEN pb.age BETWEEN 45 AND 54 THEN '45-54'
        ELSE '55+'
    END                                    AS age_group,
    pb.income_bracket,
    pb.buyer_type,
    COUNT(ps.sale_id)                      AS purchases,
    SUM(ps.sale_price)                     AS total_invested,
    ROUND(AVG(ps.sale_price), 2)           AS avg_purchase_price
FROM property_buyers pb
JOIN property_sales ps ON pb.buyer_id = ps.buyer_id
WHERE ps.status = 'Completed'
GROUP BY pb.gender, age_group, pb.income_bracket, pb.buyer_type
ORDER BY total_invested DESC;

-- 6.3  Completed vs Fallen Through
SELECT
    status,
    COUNT(sale_id)                                      AS count,
    ROUND(COUNT(sale_id) * 100.0 / SUM(COUNT(sale_id)) OVER (), 2) AS pct
FROM property_sales
GROUP BY status;

-- 6.4  Profit Margin by Property Type & Neighbourhood
SELECT
    p.property_type,
    p.neighbourhood,
    COUNT(ps.sale_id)                                   AS sales,
    SUM(ps.sale_price)                                  AS total_revenue,
    SUM(p.cost_basis)                                   AS total_cost,
    SUM(ps.sale_price - p.cost_basis - ps.closing_costs) AS net_profit,
    ROUND((SUM(ps.sale_price - p.cost_basis - ps.closing_costs) / NULLIF(SUM(ps.sale_price),0)) * 100, 2) AS profit_margin_pct,
    ROUND(AVG(ps.sale_date - ps.listing_date), 0)       AS avg_days_on_market
FROM property_sales ps
JOIN properties p ON ps.property_id = p.property_id
WHERE ps.status = 'Completed'
GROUP BY p.property_type, p.neighbourhood
ORDER BY profit_margin_pct DESC;

-- 6.5  Inventory Turnover (Days on Market)
SELECT
    p.property_type,
    p.city,
    COUNT(ps.sale_id)                                   AS sold,
    ROUND(AVG(ps.sale_date - ps.listing_date), 1)       AS avg_days_on_market,
    MIN(ps.sale_date - ps.listing_date)                 AS fastest_sale_days,
    MAX(ps.sale_date - ps.listing_date)                 AS slowest_sale_days
FROM property_sales ps
JOIN properties p ON ps.property_id = p.property_id
WHERE ps.status = 'Completed'
GROUP BY p.property_type, p.city
ORDER BY avg_days_on_market ASC;

-- 6.6  Price vs Listed Value (Negotiation Gap)
SELECT
    p.property_type,
    ROUND(AVG(p.listing_price), 2)                      AS avg_list_price,
    ROUND(AVG(ps.sale_price), 2)                        AS avg_sale_price,
    ROUND(AVG(ps.sale_price - p.listing_price), 2)      AS avg_price_diff,
    ROUND(AVG((ps.sale_price - p.listing_price) / NULLIF(p.listing_price,0)) * 100, 2) AS avg_price_diff_pct
FROM property_sales ps
JOIN properties p ON ps.property_id = p.property_id
WHERE ps.status = 'Completed'
GROUP BY p.property_type;

-- 6.7  Agent Leaderboard
SELECT
    agent_id,
    COUNT(sale_id)                                      AS properties_sold,
    SUM(sale_price)                                     AS total_sales_volume,
    SUM(sale_price * commission_pct / 100)              AS total_commission,
    ROUND(AVG(sale_price), 2)                           AS avg_sale_price,
    ROUND(AVG(sale_date - listing_date), 1)             AS avg_days_to_close
FROM property_sales
WHERE status = 'Completed'
GROUP BY agent_id
ORDER BY total_commission DESC;


-- ============================================================
--  PROJECT 7: SUBSCRIPTION REVENUE – SaaS PLATFORM
-- ============================================================

CREATE TABLE saas_customers (
    customer_id      INT PRIMARY KEY,
    company_name     VARCHAR(120),
    industry         VARCHAR(60),
    company_size     VARCHAR(20),  -- SMB, Mid-Market, Enterprise
    country          VARCHAR(60),
    signup_date      DATE,
    primary_contact  VARCHAR(100),
    gender           VARCHAR(10)
);

CREATE TABLE subscription_plans (
    plan_id          INT PRIMARY KEY,
    plan_name        VARCHAR(60),  -- Starter, Growth, Pro, Enterprise
    billing_cycle    VARCHAR(20),  -- Monthly, Annual
    monthly_price    DECIMAL(10,2),
    annual_price     DECIMAL(10,2),
    feature_set      VARCHAR(200),
    cost_to_serve    DECIMAL(10,2)
);

CREATE TABLE subscriptions (
    subscription_id  INT PRIMARY KEY,
    customer_id      INT REFERENCES saas_customers(customer_id),
    plan_id          INT REFERENCES subscription_plans(plan_id),
    start_date       DATE,
    end_date         DATE,
    status           VARCHAR(20),  -- Active, Churned, Paused, Trial
    mrr              DECIMAL(10,2),
    arr              DECIMAL(12,2),
    discount_pct     DECIMAL(5,2),
    next_renewal     DATE
);

CREATE TABLE saas_events (
    event_id         INT PRIMARY KEY,
    customer_id      INT REFERENCES saas_customers(customer_id),
    event_date       DATE,
    event_type       VARCHAR(40),  -- Login, Feature Use, Support Ticket, Upgrade, Downgrade
    feature_name     VARCHAR(80),
    session_minutes  INT
);

-- ── SaaS ANALYTICS QUERIES ────────────────────────────────────

-- 7.1  MRR / ARR by Day / Week / Month
SELECT
    DATE_TRUNC('day',   start_date)        AS period_day,
    DATE_TRUNC('week',  start_date)        AS period_week,
    DATE_TRUNC('month', start_date)        AS period_month,
    COUNT(subscription_id)                 AS new_subscriptions,
    SUM(mrr)                               AS new_mrr,
    SUM(arr)                               AS new_arr
FROM subscriptions
WHERE status IN ('Active', 'Churned')
GROUP BY 1, 2, 3
ORDER BY period_day;

-- 7.2  Customer Demographics (Industry, Company Size, Gender)
SELECT
    sc.industry,
    sc.company_size,
    sc.gender,
    sc.country,
    COUNT(DISTINCT sc.customer_id)          AS customers,
    SUM(s.mrr)                              AS total_mrr,
    ROUND(AVG(s.mrr), 2)                    AS avg_mrr_per_customer
FROM saas_customers sc
JOIN subscriptions s ON sc.customer_id = s.customer_id
WHERE s.status = 'Active'
GROUP BY sc.industry, sc.company_size, sc.gender, sc.country
ORDER BY total_mrr DESC;

-- 7.3  Active vs Churned vs Trial (Completed / Incomplete)
SELECT
    status,
    COUNT(subscription_id)                              AS count,
    ROUND(COUNT(subscription_id) * 100.0 / SUM(COUNT(subscription_id)) OVER (), 2) AS pct_share,
    SUM(mrr)                                            AS mrr_impact
FROM subscriptions
GROUP BY status
ORDER BY count DESC;

-- 7.4  Profit Margin by Plan
SELECT
    sp.plan_name,
    sp.billing_cycle,
    COUNT(s.subscription_id)                            AS subscribers,
    SUM(s.mrr)                                          AS total_mrr,
    SUM(sp.cost_to_serve)                               AS total_cost,
    SUM(s.mrr - sp.cost_to_serve)                       AS gross_profit,
    ROUND((SUM(s.mrr - sp.cost_to_serve) / NULLIF(SUM(s.mrr),0)) * 100, 2) AS profit_margin_pct
FROM subscriptions s
JOIN subscription_plans sp ON s.plan_id = sp.plan_id
WHERE s.status = 'Active'
GROUP BY sp.plan_name, sp.billing_cycle
ORDER BY profit_margin_pct DESC;

-- 7.5  Churn Rate & Revenue Churn
SELECT
    DATE_TRUNC('month', end_date)           AS churn_month,
    COUNT(CASE WHEN status = 'Churned' THEN 1 END) AS churned_customers,
    SUM(CASE WHEN status = 'Churned' THEN mrr ELSE 0 END) AS churned_mrr,
    COUNT(CASE WHEN status = 'Active'  THEN 1 END) AS active_customers,
    ROUND(COUNT(CASE WHEN status = 'Churned' THEN 1 END) * 100.0
        / NULLIF(COUNT(subscription_id),0), 2) AS churn_rate_pct
FROM subscriptions
GROUP BY DATE_TRUNC('month', end_date)
ORDER BY churn_month;

-- 7.6  Product Engagement Score (Usage Depth)
SELECT
    sc.customer_id,
    sc.company_name,
    sc.company_size,
    COUNT(se.event_id)                                  AS total_events,
    SUM(se.session_minutes)                             AS total_session_mins,
    COUNT(DISTINCT se.feature_name)                     AS features_used,
    ROUND(AVG(se.session_minutes), 1)                   AS avg_session_mins,
    MAX(se.event_date)                                  AS last_active_date,
    CURRENT_DATE - MAX(se.event_date)                   AS days_since_active
FROM saas_customers sc
JOIN saas_events se ON sc.customer_id = se.customer_id
GROUP BY sc.customer_id, sc.company_name, sc.company_size
ORDER BY total_session_mins DESC;

-- 7.7  Net Revenue Retention (NRR) by Plan
WITH base_mrr AS (
    SELECT plan_id, SUM(mrr) AS start_mrr
    FROM subscriptions
    WHERE DATE_TRUNC('month', start_date) = DATE_TRUNC('month', CURRENT_DATE - INTERVAL '1 month')
    GROUP BY plan_id
),
current_mrr AS (
    SELECT plan_id, SUM(mrr) AS end_mrr
    FROM subscriptions
    WHERE status = 'Active'
    GROUP BY plan_id
)
SELECT
    sp.plan_name,
    b.start_mrr,
    c.end_mrr,
    ROUND(c.end_mrr / NULLIF(b.start_mrr,0) * 100, 2) AS nrr_pct
FROM base_mrr b
JOIN current_mrr c     ON b.plan_id = c.plan_id
JOIN subscription_plans sp ON b.plan_id = sp.plan_id
ORDER BY nrr_pct DESC;
