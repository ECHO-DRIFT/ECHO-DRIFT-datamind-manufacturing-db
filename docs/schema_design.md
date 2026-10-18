# DataMind Manufacturing — Schema Design

## 1. Design Goals

The database must support manufacturing analytics with traceable, reproducible SQL queries.

The initial prototype uses five production lines and 30 days of synthetic manufacturing history.

## 2. Tables and Columns

### 2.1 PRODUCTION_LINES

Purpose: Store production line details.

| Column | Suggested type | Description |
|---|---|---|
| production_line_id | VARCHAR | Primary key |
| line_name | VARCHAR | Unique line name |
| description | VARCHAR | Line description |

### 2.2 MACHINES

Purpose: Store machines assigned to production lines.

| Column | Suggested type | Description |
|---|---|---|
| machine_id | VARCHAR | Primary key |
| machine_name | VARCHAR | Machine name |
| production_line_id | VARCHAR | Foreign key to PRODUCTION_LINES |
| machine_type | VARCHAR | Machine category |
| status | VARCHAR | Machine status |

### 2.3 PRODUCTS

Purpose: Store products manufactured by the factory.

| Column | Suggested type | Description |
|---|---|---|
| product_id | VARCHAR | Primary key |
| product_name | VARCHAR | Product name |
| product_category | VARCHAR | Product category |

### 2.4 PRODUCTION_ORDERS

Purpose: Compare planned production with actual production.

| Column | Suggested type | Description |
|---|---|---|
| production_order_id | VARCHAR | Primary key |
| product_id | VARCHAR | Foreign key to PRODUCTS |
| machine_id | VARCHAR | Foreign key to MACHINES |
| planned_start_at | TIMESTAMP_NTZ | Planned start |
| planned_end_at | TIMESTAMP_NTZ | Planned completion |
| actual_start_at | TIMESTAMP_NTZ | Actual start; may be NULL |
| actual_end_at | TIMESTAMP_NTZ | Actual completion; may be NULL |
| planned_quantity | NUMBER | Planned units |
| actual_quantity | NUMBER | Actual units produced |
| status | VARCHAR | Order status |

### 2.5 DOWNTIME_EVENTS

Purpose: Record periods when a machine is unavailable.

| Column | Suggested type | Description |
|---|---|---|
| downtime_id | VARCHAR | Primary key |
| machine_id | VARCHAR | Foreign key to MACHINES |
| start_at | TIMESTAMP_NTZ | Downtime start |
| end_at | TIMESTAMP_NTZ | Downtime end; may be NULL |
| reason_code | VARCHAR | Standardized reason code |
| recorded_reason | VARCHAR | Recorded explanation |

### 2.6 MATERIALS

Purpose: Store raw material details.

| Column | Suggested type | Description |
|---|---|---|
| material_id | VARCHAR | Primary key |
| material_name | VARCHAR | Material name |
| unit_of_measure | VARCHAR | Unit, such as kg or pieces |
| reorder_level | NUMBER(12,3) | Reorder threshold |

### 2.7 INVENTORY_MOVEMENTS

Purpose: Track changes in material stock.

| Column | Suggested type | Description |
|---|---|---|
| movement_id | VARCHAR | Primary key |
| material_id | VARCHAR | Foreign key to MATERIALS |
| movement_at | TIMESTAMP_NTZ | Movement timestamp |
| movement_type | VARCHAR | Receipt, consumption or adjustment |
| quantity_change | NUMBER(12,3) | Signed stock change |
| reference_type | VARCHAR | Optional reference category |
| reference_id | VARCHAR | Optional related record identifier |

Opening balances must be documented separately or represented by an explicit opening-balance movement. Do not count the same opening balance twice.

### 2.8 QUALITY_INSPECTIONS

Purpose: Record inspected and defective quantities.

| Column | Suggested type | Description |
|---|---|---|
| inspection_id | VARCHAR | Primary key |
| production_order_id | VARCHAR | Foreign key to PRODUCTION_ORDERS |
| inspected_quantity | NUMBER | Units inspected |
| defective_quantity | NUMBER | Defective units found |
| inspection_at | TIMESTAMP_NTZ | Inspection timestamp |

### 2.9 CUSTOMER_ORDERS

Purpose: Track customer delivery commitments.

| Column | Suggested type | Description |
|---|---|---|
| customer_order_id | VARCHAR | Primary key |
| product_id | VARCHAR | Foreign key to PRODUCTS |
| ordered_quantity | NUMBER | Units ordered |
| promised_delivery_at | TIMESTAMP_NTZ | Promised delivery time |
| actual_delivery_at | TIMESTAMP_NTZ | Actual delivery time; may be NULL |
| status | VARCHAR | Order status |

## 3. Relationships

- One production line can have many machines.
- One machine can have many production orders.
- One machine can have many downtime events.
- One product can have many production orders.
- One product can appear in many customer orders.
- One material can have many inventory movements.
- One production order can have many quality inspections.

## 4. Integrity Rules

- Every primary key must uniquely identify a record.
- Every foreign-key value must refer to an existing parent record when present.
- Planned end time must not precede planned start time.
- Actual end time must not precede actual start time when both exist.
- Downtime end time must not precede downtime start time when an end time exists.
- Defective quantity must be between zero and inspected quantity.
- Quantities and units of measure must be consistent.
- Missing timestamps must remain NULL when the event has not occurred or its time is unknown.

## 5. Timestamp Policy

Use UTC for stored timestamps and document that convention in application and test documentation. `TIMESTAMP_NTZ` does not store timezone information, so ingestion and query logic must consistently interpret these values as UTC.

## 6. Validation Status

This is a proposed logical schema. Validate it against the supported business questions before implementing the physical Snowflake tables.
