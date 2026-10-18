# DataMind Manufacturing — Database Requirements

## 1. Purpose

Define the database requirements for DataMind Manufacturing, a conversational analytics system that answers manufacturing questions using actual data stored in Snowflake.

The system uses Google Gemma to interpret natural-language questions and Snowflake SQL to retrieve relevant data. Answers must be supported by query results and explainable evidence.

## 2. Prototype Scope

- Production lines: 5
- Simulated history: 30 days
- Data source: Synthetic manufacturing data
- Database platform: Snowflake
- Primary responsibility: Database design, SQL analytics, data validation and testing

## 3. Required Business Questions

The database should support these questions:

1. Which machines had the most downtime during a selected period?
2. Which production orders finished late, and by how much?
3. Which raw materials are below their reorder levels?
4. Which products have the highest defect rates?
5. Which customer orders were delivered late?
6. Which production runs overlapped with machine downtime?
7. Which production lines had the lowest output compared with their targets?
8. What recorded evidence may help explain a production delay?

The system must not claim that an event caused a delay unless the available data supports that conclusion.

## 4. Proposed Database Tables

1. `PRODUCTION_LINES` — production line details.
2. `MACHINES` — machine details and production line assignments.
3. `PRODUCTS` — manufactured product details.
4. `PRODUCTION_ORDERS` — planned and actual production information.
5. `DOWNTIME_EVENTS` — machine downtime periods and recorded reasons.
6. `MATERIALS` — raw material details and reorder levels.
7. `INVENTORY_MOVEMENTS` — material receipts, consumption and adjustments.
8. `QUALITY_INSPECTIONS` — inspected quantities and defective units.
9. `CUSTOMER_ORDERS` — ordered quantities and promised versus actual delivery.

These tables are a proposed design and must be validated before implementation.

## 5. Metric Definitions

### Downtime

Calculate downtime within the selected time interval. Clip events to the interval boundaries. Avoid double-counting overlapping downtime intervals when calculating total machine downtime.

### Production delay

For completed orders:

`Production delay = Actual completion time - Planned completion time`

Report overdue unfinished orders separately. Do not invent completion times.

### Defect rate

`Defect rate (%) = Total defective units / Total inspected units × 100`

If the inspected quantity is zero, return an undefined result rather than dividing by zero.

### Current inventory

`Current stock = Opening stock + Signed inventory movements`

Receipts increase stock and consumption decreases stock. Document the treatment of adjustments and opening balances.

Flag materials whose current stock is strictly below their reorder level.

### Late deliveries

A delivered order is late when its actual delivery time is later than its promised delivery time. Report undelivered orders separately.

### Production output achievement

`Output achievement (%) = Actual quantity / Planned quantity × 100`

Only calculate this percentage when planned quantity is greater than zero.

### Downtime overlap

Identify the time interval shared by a production run and a machine downtime event. Overlap alone does not prove that downtime caused a production delay.

## 6. Data Integrity Rules

- Use consistent timestamps and document the timezone.
- Use a fixed, documented 30-day reference period for reproducible tests.
- Treat missing values as unknown, not automatically as zero.
- Define valid status values for production and customer orders.
- Validate table relationships and identifier uniqueness.
- Document opening inventory balances and inventory movement signs.
- Do not present synthetic records as real factory data.

## 7. Acceptance Tests

1. Rank machines correctly by downtime.
2. Calculate a completed order's delay correctly.
3. Identify materials below reorder levels.
4. Calculate 10 defective units out of 200 inspected as a 5% defect rate.
5. Classify late and on-time deliveries correctly.
6. Identify production runs that overlap with downtime.
7. Avoid double-counting overlapping downtime.
8. Handle incomplete production orders without inventing completion times.
9. Return a clear result when a selected period contains no matching records.
10. Avoid unsupported root-cause claims when evidence is insufficient.

## 8. Implementation Status

- [ ] Requirements reviewed by the team.
- [ ] Schema design approved.
- [ ] Synthetic data plan defined.
- [ ] Snowflake tables created.
- [ ] SQL queries validated.
- [ ] Acceptance tests passed.

## 9. Open Design Decisions

- Exact timestamp timezone and reference dates.
- Allowed status and event-reason values.
- Inventory opening balance strategy.
- Treatment of inventory adjustments.
- Final primary keys, foreign keys and constraints.
- Snowflake warehouse and access configuration.

These decisions must be settled before the database is considered ready for integration.
