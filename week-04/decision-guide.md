# Week 4 - AWS Storage & Database Architectural Decision Guide

## 1. Storage & Database Comparison Matrix

| Service | Storage / Data Type | Access Pattern | Scaling & Performance | Key Limitations & Trade-offs | "Use this when..." |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Amazon S3** | Object Storage (Key-Value) | REST API / HTTP(S); Read-heavy, write-once | Virtually unlimited storage; High throughput, higher latency compared to block storage | Not a POSIX file system; cannot be mounted as an OS boot drive; not suitable for partial in-place file modifications | **Use this when** you need durable, scalable, and cost-effective storage for unstructured data (images, backups, logs, static assets) accessible via web URLs. |
| **Amazon EBS** | Block Storage | Attached directly to a single EC2 instance (within a single AZ) | Sub-millisecond latency, predictable and configurable IOPS (SSD-backed) | Bound to a specific Availability Zone and EC2 lifecycle; cannot be natively shared across multiple instances simultaneously | **Use this when** you need a fast, persistent root volume or high-performance local disk storage for a single EC2 instance. |
| **Amazon EFS** | Managed File Storage (POSIX NFS) | Mountable concurrently by thousands of EC2 instances and containers | Automatically scales storage up and down; concurrent multi-client read/write | Higher cost per GB than EBS and S3; slightly higher latency than direct-attached EBS | **Use this when** multiple EC2 instances or container tasks (ECS/EKS) require shared, simultaneous access to a common file directory. |
| **Amazon RDS (MySQL)** | Relational Database (SQL) | Structured SQL queries, complex multi-table joins, ACID transactions | Vertical scaling (compute/storage resizing); horizontal scaling via Read Replicas | Fixed schema; scaling write throughput requires database architecture complexity; higher management overhead | **Use this when** your application requires strict ACID transactional consistency, complex relational joins, or legacy SQL compatibility. |
| **Amazon DynamoDB** | NoSQL Key-Value & Document Store | Key-value lookups (`GetItem`) and indexed range queries | Single-digit millisecond latency at any scale; automatic horizontal partitioning | Access patterns must be known ahead of time; does not support multi-table joins or ad-hoc complex analytical aggregations | **Use this when** you need predictable, single-digit millisecond latency at high scale with zero server maintenance for well-defined key-based lookups. |

---

## 2. Practical Application: PropertyLite Architecture Mapping

* **Amazon S3**:
  * *Context*: Configured to store property images (`L_Photos`) and weekly operational reporting exports, with an automated 30-day lifecycle rule transitioning objects to Glacier Flexible Retrieval.
* **Amazon EBS**:
  * *Context*: Served as the persistent root block volume for the `property-api-01` Amazon Linux 2023 EC2 instance.
* **Amazon EFS**:
  * *Context*: Suitable for future scenarios where multiple autoscaling API worker tasks require shared read/write access to centralized configuration assets or local temporary image caches.
* **Amazon RDS (MySQL)**:
  * *Context*: Hosted the raw `rets_property` dataset (55,000+ rows) to execute structured data governance operations—dropping 17 sensitive PII columns, remapping real MLS IDs to synthetic identifiers (`R1xxxxx`) via `ROW_NUMBER()`, and running analytical aggregations.
* **Amazon DynamoDB**:
  * *Context*: Populated the `Properties` table with sanitized property data to support high-throughput, low-latency key lookups for single listing endpoints (`/properties/{listing_id}`) using the partition key `L_ListingID` (e.g., `R100001`).

---

## 3. Core Architectural Takeaways

1. **Storage Classification**: Use **EBS** for single-instance OS boot disks, **EFS** for shared network file systems, and **S3** for internet-accessible, durable unstructured object storage.
2. **Database Engine Selection**: Choose **RDS** when data structures require complex relational modeling, data integrity guarantees, and flexible analytical querying. Choose **DynamoDB** when horizontal scalability, fully managed operations, and millisecond key-value retrieval are paramount.