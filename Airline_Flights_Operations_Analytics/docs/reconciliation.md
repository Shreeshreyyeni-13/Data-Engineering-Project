## Data Reconciliation – Ensuring Data Consistency Across Medallion Layers
Data reconciliation is the process of comparing records between different stages of the Medallion Architecture (Bronze, Silver, and Gold) to ensure that no records are unexpectedly lost, duplicated, or incorrectly processed during data transformation.

In the Airline Flight Operations Analytics project, reconciliation ensures that every record ingested into the Bronze layer is properly accounted for when it moves into the Silver layer.

The primary objective is to maintain data accuracy, completeness, consistency, and traceability throughout the data pipeline.

### Why Is Data Reconciliation Important?
During the transformation of raw airline flight data, several issues may occur:
- Duplicate flight records may be identified and removed.
- Records with invalid passenger counts may be rejected.
- Flights with negative average fares may be excluded.
- Missing values may be corrected using predefined business rules.
- Valid records may be transformed into a standardized format.
Without reconciliation, it becomes difficult to determine whether the records removed during cleaning were genuinely invalid or accidentally lost during processing.

Data reconciliation helps answer the following questions:
1. How many records were originally ingested into the Bronze layer?
2. How many duplicate records were identified?
3. How many records passed the data quality checks?
4. How many records were rejected, and why?
5. Does the total number of processed records match the original Bronze count?
9.4.2 Data Reconciliation Formula
The following formula is used to verify record-level reconciliation:
\[
\text{Bronze Records} =
\text{Clean Silver Records} +
\text{Rejected Records} +
\text{Duplicates Removed}
\]

For example, consider the following hypothetical scenario:

ILLUSTRATIVE EXAMPLE — NOT ACTUAL PROJECT RESULTS
Processing Stage	Number of Records
Total Bronze records	1,365
Duplicate records removed	15
Records rejected	50
Clean Silver records	1,300
Total accounted records	1,365

Reconciliation calculation:
\[
1365 = 1300 + 50 + 15
\]

Illustrative reconciliation passed
This indicates that all original records have been accounted for.

Important: This formula assumes that duplicate removal and rejection are mutually exclusive categories. If a duplicate record also violates another validation rule, define the processing order and count it in only one category to avoid double-counting.
9.4.3 Reconciliation Across Medallion Architecture
Reconciliation is performed across the different layers of the data pipeline.
Bronze Layer
Raw Airline Flight Data

Preserves all ingested source records

Duplicate identification and data validation

Silver – Clean
Valid Records

Standardized and validated flights


Silver – Rejected
Invalid Records

Rejected records with reasons



Aggregation and reconciliation checks

Gold Layer
Data Quality Summary

Reconciliation status, record counts, rejection statistics and quality metrics


- Bronze: Maintains the original source records for traceability.
- Silver: Separates valid, rejected, and duplicate records according to the defined processing rules.
- Gold: Stores aggregated data quality metrics for monitoring and reporting.


### SQL Implementation of Data Reconciliation
The following SQL queries can be used in Databricks SQL to validate the reconciliation process.

Step 1: Count Bronze Records
SELECT COUNT(*) AS bronze_count
FROM bronze_airline_flights;

Step 2: Count Clean Silver Records
SELECT COUNT(*) AS clean_count
FROM silver_airline_flights;

Step 3: Count Rejected Records
SELECT COUNT(*) AS rejected_count
FROM silver_rejected_flights;

Step 4: Calculate Duplicate Records Removed
This query assumes that flight_id identifies a unique flight and that duplicate removal retains one record per flight ID.
SELECT
    COUNT(*) - COUNT(DISTINCT flight_id)
    AS duplicate_records_removed
FROM bronze_airline_flights;


Step 5: Perform Reconciliation
WITH bronze AS (
    SELECT COUNT(*) AS bronze_count
    FROM bronze_airline_flights
),

clean AS (
    SELECT COUNT(*) AS clean_count
    FROM silver_airline_flights
),

rejected AS (
    SELECT COUNT(*) AS rejected_count
    FROM silver_rejected_flights
),

duplicates AS (
    SELECT
        COUNT(*) - COUNT(DISTINCT flight_id)
        AS duplicate_count
    FROM bronze_airline_flights
)

SELECT
    bronze.bronze_count,
    clean.clean_count,
    rejected.rejected_count,
    duplicates.duplicate_count,

    clean.clean_count
      + rejected.rejected_count
      + duplicates.duplicate_count
      AS accounted_records,

    CASE
        WHEN bronze.bronze_count =
             clean.clean_count
             + rejected.rejected_count
             + duplicates.duplicate_count
        THEN 'PASS'
        ELSE 'FAIL'
    END AS reconciliation_status

FROM bronze
CROSS JOIN clean
CROSS JOIN rejected
CROSS JOIN duplicates;

### Expected Outcome
After implementing reconciliation:
- Every Bronze record is accounted for.
- Duplicate records are tracked separately.
- Invalid records are stored with rejection reasons.
- Valid records are available in the clean Silver layer.
- Gold tables provide data quality monitoring metrics.
- Reconciliation failures can be identified before publishing business insights.
### Key Takeaway: Data reconciliation acts as a checkpoint in the Medallion Architecture. It ensures that data moving from Bronze to Silver and Gold remains traceable, consistent, and reliable. This helps build confidence in the final Airline Flight Operations Analytics dashboard.
