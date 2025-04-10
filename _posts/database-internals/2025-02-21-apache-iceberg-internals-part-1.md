---
title: "Apache Iceberg Internals - Part 1"
author: Rishabh Bhatia
categories: [database-internals]
tags: distributed systems database internals swe dive deep academic software engineering design iceberg architecture
date: 2025-02-21 10:00:00 -0700
---

This blog dives deep into the internal workings of Apache Iceberg.

## Prerequisite
This blog is useful to you if you understand table formats, data warehouses, and data lakes.

## What is Apache Iceberg?
Apache Iceberg is an open table format that provides a set of APIs for managing large analytic datasets. It was
created in 2017 by Netflix’s Ryan Blue and Daniel to overcome the challenges to maintain Hive Table format ( developed 
by facebook in 2009). Iceberg was originally built with several goals - 
1. Consistency
2. Performance
3. Easy to use
4. Evolvability
5. Scalability


## Apache Iceberg High Level Design - Table Specification
Apache Iceberg is a specification with set of libraries supporting it. It gives you the illusion of managing single 
table, behind the scenes the library is responsible for managing the metadata such as tracking table names, data files. 
The specification standardizes the representation of the table in metadata and data files. The Iceberg library defines
and implements the protocol (via APIs) for manipulating these files.

Apache Iceberg as any other table format has catalog, metadata store, data store.
- catalog - Pointing to the latest metadata file in metadata store.
- metadata store - Manages a log of snapshots which references the data files, providing a centralized way to track the
table's state and history.
- data store - Contains data.

![ Table Format High Level View ](/assets/apache%20iceberg/TableFormat%20High%20Level%20View.png)

As represented in above simplified view of iceberg, data is visible to user through catalog. In Iceberg writer 
1. writes the data to storage, 
2. validates for conflicts against the metadata file, 
3. creates the new snapshot in metadata files.
4. commit the new snapshot to the catalog.

Post the commit to catalog, the new data is visible to readers. This means readers are isolated from concurrent writes
to the data layer ( serializable isolation ).

### Iceberg Table Specification
Iceberg creates a tree like structure to manage the metadata. It has catalog which tracks the location of 
current metafile. Metadata file tracks the log of snapshots. Each snapshot tracks one manifest list. Each manifest 
list tracks list of manifest files for the snapshot. Each manifest file tracks list of data files.

![ Iceberg Table Specification ](/assets/apache%20iceberg/ApacheIceberg-Iceberg%20Table%20Spec.drawio.png)

### Detailed representation of iceberg metadata store.
Bellow does not contain the detailed schema, please check [iceberg official documentation](https://iceberg.apache.org/) 
for this.
```
<table-metadata-location>/metadata/v<version-number>.metadata.json
┌───────────────────────────────────────────────────┐
│                  Metadata                         │
│ (table metadata file tracking log of snapshots.)  │ 
│-------------------------------------------------  │
│ - Table schema                                    │
│ - Snapshot list                                   │
│ - Current snapshot ID                             │
│ - Partition spec & sort order                     │
│ - Table properties                                │
└───────────────────────────────────────────────────┘
            │ │ │  
            ▼ ▼ ▼
Snapshot list is present in above metadata file, 
and shown here as separate block just for explaining.
┌──────────────────────────────────┐
│        Snapshot                  │
│ (snapshot in metadata            │
│    tracks manifest list)         │  
│----------------------------------│   
│ - Snapshot ID                    │
│ - Manifest list location         │
│ - Timestamp                      │
│ - Summary (commit metadata)      │
└──────────────────────────────────┘
            │  
            ▼
<table-metadata-location>/metadata/snap-<snapshot-id>-<random-uuid>.avro
┌──────────────────────────────────────────┐
│        Manifest List                     │
│ (list of manifest files per snapshot)    │
│-------------------------------------     │ 
│ - Location of the manifest file          │
│ - added_snapshot_id                      │
└──────────────────────────────────────────┘
            │ │ │
            ▼ ▼ ▼
<table-metadata-location>/metadata/<manifest-file-id>-<random-uuid>.avro
┌──────────────────────────────────┐
│        Manifest Files            │
│----------------------------------│
│ - References to data files       │
│ - Partition-level stats          │ 
│ - File existence (added/deleted) │
│ - Data file format (Parquet, ORC)│
└──────────────────────────────────┘
            │ │ │
            ▼ ▼ ▼
<table-location>/data/<partition-path>/<unique-file-name>.<format>
┌──────────────────────────────────┐
│          Data Files              │
│----------------------------------│
│ - Actual data storage (e.g.,     │ 
│   Parquet, ORC, Avro files)      │
│ - Column statistics              │
└──────────────────────────────────┘

````
### Understanding how iceberg metadata store evolve with new write operations
For every write operation in an Apache Iceberg table, a **new snapshot** is created. This process starts with 
generating new data files and updating or adding manifest files that track these changes. A **new manifest list**
is then created, referencing both newly added manifest files and existing ones that remain unchanged. This ensures 
efficient tracking of table modifications without rewriting unchanged metadata. Once the manifest list is prepared, 
Iceberg records it as part of the new snapshot, assigning it a unique snapshot ID. Finally, a **new table metadata 
file** (metadata.json) is written, pointing to the latest snapshot, making it the current state of the table while 
preserving all previous snapshots for **time travel and rollback capabilities**.

### Understanding Apache Iceberg Table Formats: Merge-on-Read (MoR) vs Copy-on-Write (CoW)

Iceberg supports two write approaches -
1. Copy on write
2. Merge On read 

### Copy On Write
In Copy-on-Write (CoW), when a row is updated or deleted, instead of modifying the original data file, a new data file 
is created containing the modified records while preserving the original file. This approach ensures data consistency 
and enables time travel capabilities by maintaining previous versions of the data.

Pros
- Optimized for read heavy workloads. The data is in final state and there is no work of merging data at query time.

Cons
- Not efficient for frequent updates/deletes.
- Higher storage cost due to complete file rewrites.

![ Iceberg COW Simplified Example ](/assets/apache%20iceberg/ApacheIceberg-Iceberg%20table%20Write%20COW%20Simplified%20Example.drawio.png)


### Detailed Example for Metadata store evolution During COW
To understand how write impacts the metadata store we will run the following queries on an empty table
and see how the metadata store evolves-

```
INSERT INTO my_iceberg_table (name, team)
VALUES
('Steve', 'product'),
('Elon', 'engineering'),
('Jeff', 'business');

INSERT INTO my_iceberg_table (name, team)
VALUES
('Warren', 'account');

UPDATE my_iceberg_table
SET team = 'innovator' WHERE name = 'Jeff';

UPDATE my_iceberg_table
SET team = 'space' WHERE name = 'Elon';

DELETE FROM my_iceberg_table
WHERE name = 'Steve';
```

![ Iceberg COW Example ](/assets/apache%20iceberg/ApacheIceberg-Iceberg%20table%20Write%20COW.drawio.png)

### Merge On Read
In Merge-on-Read (MoR), when data is modified (updated or deleted), the changes are initially recorded in delta files
rather than creating new base data files immediately. These delta files store the modifications separately until a 
compaction operation merges them with the base files. This approach optimizes write performance by deferring the 
expensive merge operations while maintaining data consistency. 

Iceberg supports two types of delete operations in MoR tables - Positional deletes and Equality deletes.

Positional deletes in MoR work by maintaining position-based references to the records that need to be deleted. 
These delete files store the file and position information of the records to be removed, rather than the actual record
data. During read operations, these positional delete files are used to filter out the deleted records from the base 
data files.

![ Iceberg MOR Simplified Positional Delete Example ](/assets/apache%20iceberg/ApacheIceberg-Iceberg%20table%20Write%20MOR%20Position%20Deletes%20Simplified%20Example.drawio.png)

Equality deletes in MoR operate by storing the equality conditions that identify which records should be deleted. 
Instead of tracking positions, these delete files contain the values of specific columns that match the records to be
removed. When reading data, the system applies these equality conditions to filter out deleted records, making it
particularly useful for scenarios where multiple records might match the delete criteria.

![ Iceberg MOR Simplified Equality Delete Example ](/assets/apache%20iceberg/ApacheIceberg-Iceberg%20table%20Write%20MOR%20Equality%20Deletes%20Simplified%20Example.drawio.png)


### Detailed Example for Metadata store evolution During MOR - Positional Deletes

Will reuse the same queries and understand how the metadata store evolve with MOR.

```
INSERT INTO my_iceberg_table (name, team)
VALUES
('Steve', 'product'),
('Elon', 'engineering'),
('Jeff', 'business');

INSERT INTO my_iceberg_table (name, team)
VALUES
('Warren', 'account');

UPDATE my_iceberg_table
SET team = 'innovator' WHERE name = 'Jeff';

UPDATE my_iceberg_table
SET team = 'space' WHERE name = 'Elon';

DELETE FROM my_iceberg_table
WHERE name = 'Steve';
```

Note: For simplification I have not represented metadata file but only new snapshots created for every write. 


![ Iceberg MOR Example ](/assets/apache%20iceberg/ApacheIceberg-Iceberg%20table%20Write%20MOR.drawio.png)


### Up Next
- How Apache Iceberg does compaction ?
- How Apache Iceberg handle data conflicts ?


## References
- 