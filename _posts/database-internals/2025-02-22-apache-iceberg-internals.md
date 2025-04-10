---
title: "Apache Iceberg Internals"
author: Rishabh Bhatia
categories: [database-internals]
tags: distributed systems database internals swe dive deep academic software engineering design iceberg architecture
date: 2025-02-22 10:00:00 -0700
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
### Examples of how iceberg metadata store evolve with new write operations
For every write a new snapshot is created. This means a new manifest list is created which reference multiple manifest
files (old and/or new). The newly created manifest list is then added to new metadata file.


## Apache Iceberg Internal Working - Protocol


