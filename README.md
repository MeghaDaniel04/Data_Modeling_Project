# Power BI Data Modeling: From a Messy 23-Table Dataset to a Clean Star Schema

## Business Context

A global B2B company operating across North America, Europe and Asia manages customers, products, sales, orders, invoices, payments, shipments, inventory and marketing activities.

The source data consisted of 23 interconnected tables with duplicate records, inconsistent structures, many-to-many relationships and redundant information. These issues made reporting unreliable and increased model complexity.

The objective was to transform the raw data into a clean, reliable and efficient analytical data model using Power Query and Power BI.

## Objectives

- Transform the existing complex model into a star schema.
- Maintain accurate totals throughout data transformations.
- Remove duplicate, redundant and unnecessary data.
- Establish consistent naming conventions and business-friendly values.
- Create a shared date dimension and reusable DAX measures.
- Implement dynamic row-level security (RLS).

## Tools & Technologies

- Power BI Desktop
- Power Query (M)
- DAX
- Dimensional Modeling
- Star Schema Design
- Row-Level Security

## Source Data Summary (23 Tables)

| Area                    | Raw tables                                                                   | Issues found                                                                                                           |
| ----------------------- | ---------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| Customers and geography | customer_master, customer_contacts, user_details, addresses, cities, regions | one entity under two names (customer and user), several contacts per customer, headers in first row, test customer 999 |
| Products                | products, subcategories                                                      | duplicate products with different codes, test row, category and subcategory packed in one pipe-separated column        |
| Sales                   | orders 2025, orders 2026, order_line_items                                   | header and detail structure, one event split by year, legacy and empty columns, numeric channel codes                  |
| Billing and delivery    | invoices, invoice_lines, payments, shipments, sheet1                         | same money repeated across tables, sheet1 is an exact duplicate of shipments                                           |
| Marketing               | campaign_log, campaign_skus                                                  | static attributes mixed with daily transactions, product list stored as comma-separated text                           |
| Other                   | inventory, sales_targets, exchange_rate, security, dimension order           | months stored as column headers, unconnectable table, garbage table                                                    |


The source tables contained duplicate entities, test records, split transaction data and unstructured columns.

![Initial Data Model](<Screenshot 2026-11-07 171235.png>)
## Approach

1. **Data Preparation:** Inspected all 23 tables and organised Power Query into staging, dimensions, facts and support folders.
2. **Dimension Building:** Combined related source tables, resolved duplicates, cleaned attributes and established consistent keys.
3. **Fact Building:** Created fact tables at the appropriate grain, using surrogate keys and validating control totals after transformations.
4. **Model Development:** Established one-to-many relationships, shared dimensions and appropriate filter directions.
5. **Final Implementation:** Created the date dimension, DAX measures and dynamic row-level security.

## Key Modeling Decisions

| Challenge                                        | Solution                                                |
| ------------------------------------------------ | ------------------------------------------------------- |
| Multiple contacts per customer                   | Retained the primary contact to preserve customer grain |
| Orders split between headers and details         | Built sales fact at order-line grain                    |
| Duplicate product names                          | Used source identifiers to resolve duplicates           |
| Unrelated order attributes                       | Created a junk dimension                                |
| Ship-to and bill-to geography                    | Used a role-playing geography dimension                 |
| Repeated financial values across order processes | Created an accumulating snapshot fact                   |
| Campaign products stored as lists                | Created a factless coverage fact                        |
| Inventory months stored as columns               | Unpivoted months into rows                              |
| Repeated order totals                            | Recalculated totals from order-line values              |

## Final Data Model

The redesigned model contains:

- **6 Dimensions:** Customer, Product, Order Flags, Geography, Campaign and Date.
- **6 Fact Tables:** Sales, Order Process, Inventory, Campaign Spend, Promotion Coverage and Sales Targets.
- **1 Security Table:** Regional user access.
- **1 Measures Table:** Centralised DAX measures.

All fact tables connect to shared dimensions through one-to-many relationships with single-direction filtering. No fact-to-fact relationships are used.

![Final Data Model](<Screenshot 2026-11-07 170530.png>)

## DAX & Date Dimension

- Created a dynamic date dimension using `CALENDARAUTO()`.
- Centralised reusable measures in a dedicated `_measures` table.
- Implemented sales, order, customer and order-to-payment calculations.

## Row-Level Security

Implemented dynamic regional access using `USERPRINCIPALNAME()`.

The security table filters the customer dimension, which in turn filters the relevant fact tables. Access was tested using Power BI's View As functionality.

## Results & Validation

- Reduced 23 raw tables to 6 dimensions, 6 facts and supporting tables.
- Validated customer and product counts at 60 records each.
- Reconciled control totals throughout the transformation process.
- Checked duplicates, row counts and relationship cardinality.
- Established a consistent analytical model for reporting.

## Key Learnings

1. **Understand Grain Before Merging:** Identify what one row represents in each table and check relationship cardinality before merging to avoid unintended row duplication.

2. **Maintain a Single Source of Truth:** Store descriptive information in appropriate dimensions and avoid repeating attributes across tables when they can be accessed through keys.

3. **Transform Data for Reporting:** Reshape data by unpivoting period columns, splitting delimited values and creating mapping tables to make it suitable for analytical reporting.

4. **Understand Dimensions vs Facts:** Dimensions describe business entities and events, while fact tables record business activities, transactions and measurable outcomes.

5. **Choose the Right Fact Table Design:** Select transaction, periodic snapshot or accumulating snapshot facts based on the business process, required grain and analytical questions.

6. **Understand Factless Fact Tables:** Use factless facts to capture relationships or occurrences, such as campaign-to-product mappings, without requiring numerical measures.

7. **Use Dimensional Modeling Patterns:** Apply junk dimensions, role-playing dimensions and shared dimensions to simplify the model and support consistent analysis.

8. **Connect Facts Through Shared Dimensions:** Keep fact tables independent and connect them through shared dimensions using appropriate one-to-many relationships and filter directions.

9. **Establish Modeling Standards:** Define consistent naming conventions, table prefixes, key suffixes and data formatting rules before building the model.

## Acknowledgements

This is a guided learning project based on the dataset and project brief from **Data with Baraa**, through the YouTube tutorial **[[Video Title and Link](https://youtu.be/0A2k62YEbfI?si=b2Ourg1xOskf7cbT)]**.

The data modeling implementation, documentation and learning notes in this repository represent my own understanding of the process.
