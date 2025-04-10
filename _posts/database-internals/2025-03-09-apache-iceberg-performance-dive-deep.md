---
title: "Apache Iceberg Internals Dive Deep On Performance"
author: Rishabh Bhatia
categories: [database-internals]
tags: distributed systems database internals swe dive deep academic software engineering design iceberg architecture
date: 2025-03-09 10:00:00 -0700
---

In this blog I will explain the performance of Apache Iceberg in great details. Apache Iceberg is a high-performance 
table format designed for large-scale analytics workloads. While its consistency and schema evolution features are 
covered in previous blog, its impact on **query performance** is equally transformative. This document provides an 
**in-depth** analysis of Iceberg’s **read optimizations**, focusing on **metadata efficiency, file pruning, 
predicate push down, vectorized reads, data layout strategies, caching mechanisms, and integration with compute engine 
like Apache Spark**.   
By the end of this document, you will have a deep understanding of how Iceberg enhances performance, 
the trade-offs involved, and best practices for maximizing efficiency in read-heavy workloads.


## Performance Mental Model
Performance of any table format is quantified by running same set of queries (usually TPC-DS) against benchmark and
experiment. Major thing to note here is driver of performance comes from compute engine. Compute engines like Apache
Spark have different planning states using which it optimizes the SQL plan. The table format facilitates the 
optimization done by compute engine by providing specific information.  
**Mental model for performance** is to find the minimum unavoidable cost and then try to find ways to remove the extra
work done by the system. Bruce Lee was on point for performance mental model by saying **Hack away the unessential.**  
Lets apply the same mental model for iceberg performance. The bare minimum work that needs to be done is to read the
exact rows requested by the query. All the work required to reach to those query should be optimized/removed to reach
the performant workload.

## Why do database developers obsess over performance
The performance gains triggers a virtuous cycle. Faster compute time will lead to
1. Lower Cost - Less compute time means less resources are used which reduces your cloud bill.
2. Makes Ideation Faster - Less compute time means software / data engineer will be able to run more experiments 
within the same time period.
3. Quicker Analytics Insights – Reduces the lag in Analytics Insights reaching the front end of your business. 
4. Faster Operations as 

## Prerequisite


## Reference
- [Iceberg Code Base](https://github.com/apache/iceberg) for understanding iceberg protocol
- [Iceberg Official docs](https://iceberg.apache.org/terms/) for understanding iceberg specification
- [Apache Iceberg The Definitive Guide](https://www.dremio.com/wp-content/uploads/2023/02/apache-iceberg-TDG_ER1.pdf) for highly understanding for Iceberg.
