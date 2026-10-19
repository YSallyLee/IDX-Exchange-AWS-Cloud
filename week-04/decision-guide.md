# Week 4 - AWS Storage & Database Architectural Decision Guide

## 1. Storage & Database Comparison Matrix

| Service | Data Type | Access Pattern | Advantages & Performance | Limitation & Disadvantages | "Use this when..." |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Amazon S3** | Object Storage (Key-Value) | Upload and download via API/HTTP URL | Nearly unlimited capacity, extremely durable, very inexpensive | Cannot be used as a bootable hard drive, unsuitable for frequent local text editing | **Use this when** when you need to store large amounts of unstructured files (such as property photos, logs, historical backups) and access them via URL.
| **Amazon EBS** | Block Storage | Virtual disk mounted on a single EC2 instance | Extremely fast read/write speeds and ultra-low latency (SSD specifications) | Can only be bound to a single EC2 instance; cannot natively allow multiple machines to read/write simultaneously | **Use this when** when you need to provide a system boot disk or high-performance local cache for a single EC2 virtual machine. |
| **Amazon EFS** | Shared file system (File Storage) | Mounted through standard Network File Protocol (NFS) | Allows hundreds of EC2s to be connected at the same time for shared reading and writing | More expensive than S3 and EBS, and the read and write latency is slightly higher than local hard drives | **Use this when** Multiple EC2 or container servers need to access the same shared folder "simultaneously". |
| **Amazon RDS (MySQL)** | Relational Database (SQL Database) | Uses standard SQL syntax for querying and joining | Supports complex JOIN cross-table queries, transactional consistency, and powerful aggregation calculations | Difficult to scale (usually requires paying for server upgrades), high maintenance costs | Use this when: When the data structure is rigorous and complex join queries, cross-table calculations, or data cleaning and transformation are required.|
| **Amazon DynamoDB** | NoSQL key-value/file database | Fetch data directly with primary key (Partition Key) | Automatically expand on demand, query speed is stable within a few milliseconds, regardless of the underlying server | Not suitable for complex multi-table associations (JOIN) or temporary complex statistical analysis | **Use this when** The access mode is very fixed (such as using a single ID to query data), and it needs to withstand extremely high traffic and extremely low latency. |

---

## 2. Application scenarios of PropertyLite in actual projects

* **S3**:
Used to store real estate photos (`L_Photos`), and set to automatically transfer them to Glacier cryogenic storage after 30 days to save costs.
* **EBS**:
Use the system disk that our API server (`property-api-01`) boots up during Week 3.
* **EFS**:
If multiple API servers need to share the same cache or configuration file in the future, they can be mounted using EFS.
* **RDS (MySQL)**:
The original property data table of 55,000 entries was imported to perform data governance (removing 17 individual data fields, remapping and synthesizing IDs, and calculating the average price for each city).
* **Amazon DynamoDB**:
Create a `Properties` table so that when users enter a property code (e.g., `R100001`) on the front end, they can instantly view the property details.
