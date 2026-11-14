## 15. Questions for Project Review

This section explains the important technical and business decisions made during the Airline Flight Operations Analytics project.


### 1. Why do we keep the raw data in Bronze instead of cleaning the file directly?
We keep the original data in the Bronze layer to preserve the source data without losing any information.
**Reasons:**
- Maintains the original source data for future reference.
- Helps identify and investigate data-quality issues.
- Allows us to reprocess the data if cleaning rules change.
- Provides traceability throughout the data pipeline.
The Bronze layer stores raw data, the Silver layer cleans and validates it, and the Gold layer prepares business-ready summary tables.


### 2. Why is a negative delay accepted, but a flight with more passengers than seats rejected?
A negative delay means that a flight departed earlier than its scheduled departure time. It is valid data and is considered an On Time flight.
However, when the number of passengers exceeds the aircraft's seat capacity, it violates a business rule and is treated as invalid data.
**Project rules:**
- Negative delay: Accepted and classified as On Time.
- Passengers greater than seat capacity: Rejected.
- Negative average fare: Rejected.
- 

### 3. Why can `flight_id` find duplicates but `flight_number` cannot?
`flight_id` uniquely identifies an individual flight on a particular day, whereas `flight_number` can be reused by an airline on different dates.
**Example:**
| flight_id | flight_number | flight_date |
|---|---|---|
| FL400001 | SJ-472 | 2025-07-11 |
| FL400025 | SJ-472 | 2025-07-12 |
These are two different flights even though they have the same flight number.
Therefore, `flight_id` is used to identify duplicate flight records because it is unique for each flight.


### 4. How does `TIMESTAMPDIFF` fill a blank delay, and what happens if you swap the two timestamps?
`TIMESTAMPDIFF` calculates the time difference between two timestamps in the specified unit.
In this project, we calculate the delay in minutes using:
```sql
TIMESTAMPDIFF(
    MINUTE,
    scheduled_departure,
    actual_departure
)
```
**Example:**
- Scheduled departure: 08:30
- Actual departure: 09:00
- Calculated delay: 30 minutes.
If the actual departure is 08:15, the result is -15 minutes, which means the flight departed early.
If we swap the timestamps, the result becomes negative for a flight that departed late. Therefore, the correct timestamp order is important.
We use `COALESCE()` to fill missing delay values:
```sql
COALESCE(
    delay_minutes,
    TIMESTAMPDIFF(MINUTE, scheduled_departure, actual_departure)
)
```


### 5. Why do we calculate load factor as SUM(passengers) ÷ SUM(seats) instead of averaging each flight's load factor?
We calculate the overall load factor using total passengers divided by total available seats because different flights may have different seat capacities.
This gives us a more accurate overall seat-occupancy percentage.
**Formula:**
Load Factor (%) = (Total Passengers / Total Seat Capacity) × 100
**Example:**
| Flight | Passengers | Seats | Load Factor |
|---|---:|---:|---:|
| A | 90 | 100 | 90% |
| B | 50 | 200 | 25% |
Correct overall load factor:
```text
((90 + 50) / (100 + 200)) * 100 = 46.67%
```
A simple average of the two flight percentages would give 57.5%, which does not accurately represent the overall seat occupancy.


### 6. Why do we multiply by 100.0 and not 100 when calculating percentages?
We use `100.0` to encourage decimal arithmetic and preserve fractional percentage values during calculations.
**SQL example:**
```sql
ROUND(
    SUM(passengers) * 100.0 / SUM(seat_capacity),
    1
) AS load_factor_pct
```
This helps avoid unintended integer arithmetic and supports more accurate percentage calculations.


### 7. Which airline has the best on-time performance, and what is the most common delay reason?
To identify the airline with the best on-time performance, we use the `gold_airline_summary` table.
**SQL Query – Best On-Time Performing Airline**
```sql
SELECT
    airline,
    on_time_pct
FROM gold_airline_summary
ORDER BY on_time_pct DESC
LIMIT 1;
```
To identify the most common delay reason, we use the `gold_delay_summary` table.
**SQL Query – Most Common Delay Reason**
```sql
SELECT
    delay_reason,
    SUM(flights) AS total_flights
FROM gold_delay_summary
WHERE delay_category <> 'On Time'
GROUP BY delay_reason
ORDER BY total_flights DESC
LIMIT 1;
```
These queries help identify the airline with the highest on-time percentage and the most frequent reason for flight delays.
*Note: The actual airline name, on-time percentage, and most common delay reason should be added after executing these queries and verifying the results.*


### 8. Which dashboard tool did you choose, and why?
The project supports two dashboard options: Databricks AI/BI Dashboard and Power BI.
**If using Databricks AI/BI Dashboard:**
I chose Databricks AI/BI Dashboard because the project is developed within the Databricks environment. It allows me to connect directly to Gold-layer tables, create KPI cards and charts, add interactive filters, and visualize business insights without moving the data to another platform.


**Key Takeaway:**

Through this project, I learned how to build a structured data pipeline using the Medallion Architecture, apply data-cleaning and validation rules, create business-ready Gold tables, calculate meaningful KPIs, and present airline operational insights through an interactive dashboard.
