# Point-of-Sales-System

# Point-of-Sale (POS) System: From Relational Design to NoSQL & Cloud Deployment 🚀

## 🌟 Project Overview

This repository documents an in-depth academic project that involved designing, building, and deploying a fully functional Point of Sale (POS) System. The project journeyed through the complete data lifecycle across **10 key milestones**: from initial infrastructure setup on **AWS**, ingesting and cleaning raw data, designing a normalized **relational schema (MariaDB)**, to implementing advanced SQL features like **ETL processes, views, stored procedures, and triggers**.

The project further explored database scalability and high availability by deploying the system on **master-replica and Galera peer-to-peer clusters**. Finally, it bridged the gap to modern NoSQL solutions by **exporting relational data to JSON** and integrating it into a **MongoDB environment** for flexible querying and analysis. This project simulates real-world enterprise database scenarios, showcasing a robust understanding of both SQL and NoSQL paradigms.

*(Due to academic integrity policies, the raw source code (`.sql` files, scripts) and original CSV data files are not publicly shared in this repository. However, this document details the design, architecture, methodologies, and technologies used throughout each milestone.)*

---
## 🎯 Core Objectives & Key Features

* **End-to-End Database Design & Implementation:** Building a complete POS system from scratch.
* **Robust ETL Pipeline:** Extracting, transforming, and loading messy data into a structured relational database (MariaDB) and subsequently into NoSQL (MongoDB).
* **Advanced SQL Development:** Utilizing views, indexes, transactions, prepared statements, stored procedures, and triggers to optimize performance, ensure data integrity, and automate business logic.
* **Scalable Database Architectures:**
    * Implementing **MariaDB Master-Replica clustering** for read scalability.
    * Deploying a **MariaDB Galera Cluster** for multi-master replication and high availability.
* **Hybrid Data Environment Simulation:**
    * Exporting denormalized JSON data from the relational system.
    * Importing and querying this data in **MongoDB**, showcasing NoSQL data modeling and querying techniques.
* **Cloud-Based Deployment:** Leveraging **AWS EC2** instances for hosting the database servers.

---
## 🏗️ System Architecture Stages

1.  **Foundation (Relational - MariaDB on AWS):**
    * Secure AWS EC2 instance setup.
    * MariaDB installation and schema implementation based on a normalized ERD.
    * Initial data ingestion and cleansing (ETL).
2.  **Optimization & Automation (Relational):**
    * Creation of SQL views and indexes for query performance and simplified data access.
    * Implementation of transactions, stored procedures, and triggers to enforce business rules and automate data management tasks.
3.  **Scalability & High Availability (Relational):**
    * Deployment and testing of a Master-Replica MariaDB cluster.
    * Deployment and testing of a Peer-to-Peer MariaDB Galera Cluster.
4.  **Transition to NoSQL (MongoDB):**
    * Extraction of data from MariaDB into structured JSON formats.
    * Importing JSON data into MongoDB collections.
    * Performing analytical queries using MongoDB's query language.

---
## 📄 Entity-Relationship Diagram (ERD)

The diagram below represents the core relational structure of the Point of Sale (POS) System. It models key entities such as customers, orders, products, cities, and price history. This normalized schema ensures data integrity and served as the foundation for ETL, SQL development, and the eventual NoSQL migration.

![Entity-Relationship Diagram](https://github.com/AshokVarma77/Point-of-Sales-System/blob/38ab753d1aa7b83e2c1798df4a338cc0c96eef24/Architecture.png)



**Key Relational Entities (MariaDB):**

* **Product:** `id`, `name`, `currentPrice`, `availableQuantity`
* **City:** `zip` (PK), `city`, `state`
* **Customer:** `id` (PK), `firstName`, `lastName`, `email`, `address1`, `address2`, `phone`, `birthdate`, `zip` (FK)
* **Order:** `id` (PK), `datePlaced`, `dateShipped`, `customer_id` (FK), `OrderSubtotal`, `SalesTax`, `OrderTotal` (virtual column: `OrderSubtotal + SalesTax`)
* **OrderLine:** `order_id` (FK), `product_id` (FK), `quantity`, `UnitPrice`, `LineTotal` (virtual column: `Quantity * UnitPrice`)
* **PriceHistory:** `ChangeID` (PK, auto-increment), `product_id` (FK), `oldPrice`, `newPrice`, `changeDate` (default `NOW()`)
* **SalesTax:** `zip` (PK, FK to City), `rate` (DECIMAL for percentages)

---
##  milestones Journey 🗺️

This project was developed progressively through the following 10 milestones:

**Milestone 1: Infrastructure Setup (AWS & MariaDB)** ⚙️
* Launched and secured an Amazon EC2 instance (Amazon Linux 2023).
* Installed and configured MariaDB, including creating the `pos` database and user.
* Designed and implemented the initial relational schema (`structure.sql`) with primary/foreign keys and appropriate data types (e.g., `DECIMAL(5) ZEROFILL` for ZIP codes).

**Milestone 2: Extract, Transform, and Load (ETL)** 🔄
* Developed an `etl.sql` script to import synthetic CSV data (with intentional inconsistencies) into staging tables.
* Performed data cleaning and transformation using SQL:
    * Handled missing/invalid data (e.g., malformed dates to `NULL`).
    * Aggregated repeated `OrderLine` entries to calculate correct quantities.
    * Ensured data consistency with the ERD and correct ZIP code formatting.

**Milestone 3: Views and Indexes** 👓⚡
* Created logical SQL **views** (`views.sql`) for simplified data access (e.g., `v_CustomerNames`, `v_Customers`, `v_ProductBuyers`, `v_CustomerPurchases`).
* Developed **materialized views** (as actual tables like `mv_ProductBuyers` and `mv_CustomerPurchases`) for pre-computed results.
* Added **indexes** (e.g., `idx_CustomerEmail` on `Customer.Email`, `idx_ProductName` on `Product.Name`) to optimize query performance.

**Milestone 4: Transactions and Prepared Statements** 🛡️✍️
* Utilized `START TRANSACTION`, `COMMIT`, `ROLLBACK` for atomic operations and data consistency.
* Tested transaction isolation levels across different sessions, observing visibility of uncommitted changes.
* Implemented **prepared statements** for dynamic data insertion, demonstrating performance benefits and SQL injection prevention.

**Milestone 5: Stored Procedures** 🧩⚙️
* Altered tables to support calculated/derived fields (e.g., added `UnitPrice` and virtual `LineTotal` to `OrderLine`; renamed `OrderTotal` to `OrderSubtotal` and added new virtual `OrderTotal` and `SalesTax` columns to `Order`).
* Created **stored procedures** (`proc.sql`) to encapsulate business logic:
    * `proc_FillUnitPrice`: Populate `OrderLine.UnitPrice` from `Product.currentPrice`.
    * `proc_FillOrderTotal`: Calculate `Order.OrderSubtotal` by aggregating `LineTotal` from `OrderLine`.
    * `proc_RefreshMV`: Refresh materialized views (`mv_ProductBuyers`, `mv_CustomerPurchases`) transaction-safely.

**Milestone 6: Database Triggers** automating Updates & Integrity** 자동으로 업데이트 및 무결성 보장 🔔
* Created a `SalesTax` table to store tax rates by ZIP code.
* Altered `PriceHistory` (`ChangeID` to auto-increment, `changeDate` to default `NOW()`) and `Order` table structure for subtotal, sales tax, and final total.
* Developed `BEFORE` and `AFTER` **triggers** (`trig.sql`) for DML operations to:
    * Log price changes to `PriceHistory` only if the price actually changed.
    * Set default `OrderLine.Quantity = 1` if not provided.
    * Fetch `OrderLine.UnitPrice` from `Product.currentPrice` during `OrderLine` insert.
    * Update `Product.availableQuantity` (inventory management) on `OrderLine` insert, update, or delete.
    * Prevent `OrderLine` inserts if `quantity` exceeds `Product.availableQuantity`.
    * Recalculate `Order.OrderSubtotal` and `Order.SalesTax` (based on customer's ZIP and `SalesTax` table) after `OrderLine` changes or `Order`'s customer/shipping details change, using proper rounding.
    * Update materialized views incrementally by calling helper procedures.

**Milestone 7: Master-Replica Clustering (MariaDB)** 🔗🔗
* Set up a 3-node cluster on AWS EC2: 1 primary (writer), 2 replicas (read-only).
* Configured MariaDB replication (binary logging, unique `server_id`, `read_only` on replicas, private IPs for communication).
* Initialized the database on primary by running `views.sql` (which sources `etl.sql`).
* Verified data propagation from primary to replicas and tested that write operations were blocked on replicas.

**Milestone 8: Peer-to-Peer Clustering (MariaDB Galera Cluster)** 🔄🔄🔄
* Set up a 3-node synchronous multi-master Galera Cluster on AWS EC2 (Nodes A, B, C).
* Configured `wsrep` settings in `server.cnf` (cluster address, node address/name, `wsrep_sst_method=rsync`).
* Ensured proper startup/shutdown order (A→B→C on startup; C→B→A on shutdown).
* Initialized database on one node (e.g., by running `views.sql`) and verified schema/data replication across all nodes.
* Tested that writes to any node were synchronously replicated, ensuring high availability and data consistency.

**Milestone 9: JSON Export for NoSQL (MongoDB Preparation)** 📄➡️ documenti
* Created `json.sql` (sourcing `trig.sql` first) to export data from MariaDB into denormalized JSON structures using `JSON_OBJECT` and `JSON_ARRAYAGG`.
* Generated four distinct line-separated JSON files (`INTO OUTFILE` to `/var/lib/mysql/pos/`):
    * `cust1.json`: Customer information with combined full name and multi-line formatted address (excluding `Address2` if null).
    * `prod.json`: Products and an array of unique customers (object with id, name) who purchased each product.
    * `ord.json`: Orders with embedded buyer (customer object) and product objects (array of items with product id, name, quantity, unit price).
    * `cust2.json`: `cust1` details combined with an embedded array of each customer's order history (order id, dates, items with product details).
* Validated JSON structure and syntax.

**Milestone 10: MongoDB Integration & Analysis** 🍃💡
* Deployed MongoDB on a new AWS EC2 instance (Amazon Linux 2023).
* Imported the JSON files into MongoDB collections using `mongoimport`:
    * `cust1.json` → `Customers` collection
    * `cust2.json` → `CustomerOrders` collection
    * `prod.json` → `Products` collection
    * `ord.json` → `Orders` collection
* Verified imports by checking document counts and sampling documents in `mongosh`.
* Executed analytical queries in `mongosh` to answer business questions (e.g., customers in Texas, best customer by total spend, most frequently purchased product, products with low/no sales, list customers for product recall).
* Reflected on differences in data modeling, querying (SQL joins vs. document traversal/aggregation pipeline), and business logic implementation between SQL and MongoDB.

---
## 🛠️ Technologies & Tools

* **Cloud Platform:** AWS (Amazon Web Services) - EC2 for virtual servers.
* **Relational Database:** MariaDB
    * **Clustering:** MariaDB Replication (Master-Replica), MariaDB Galera Cluster (Peer-to-Peer).
* **NoSQL Database:** MongoDB
* **Operating System:** Amazon Linux 2023
* **Languages/APIs:** SQL (MariaDB dialect), Shell scripting (for setup/management), JSON.
* **Key SQL Features Used:** DDL, DML, ETL (LOAD DATA INFILE), Views, Materialized Views, Indexes, Transactions, Prepared Statements, Stored Procedures, Triggers, JSON functions (`JSON_OBJECT`, `JSON_ARRAYAGG`).
* **NoSQL Tools:** `mongoimport` (for data ingestion), `mongosh` (MongoDB Shell for querying and administration).
* **Security:** Key-based SSH access for EC2 instances.
* **Version Control (Conceptual):** Git & GitHub for project documentation (this README).

---
