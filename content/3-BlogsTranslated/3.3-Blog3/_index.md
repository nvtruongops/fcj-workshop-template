---
title: "Blog 3"
date: 2025-09-26
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# Scaling Cluster Manager and Admin APIs in Amazon OpenSearch Service

Amazon OpenSearch Service is a managed service that makes it easy to deploy, secure, and operate OpenSearch clusters at scale in the AWS Cloud. A typical OpenSearch cluster consists of cluster manager, data, and coordinator nodes. It is recommended to have three cluster manager nodes, with one node elected as the leader.

---

## Introduction

With OpenSearch Service version 2.17, Amazon OpenSearch Service now supports clusters of up to 1,000 nodes capable of handling 500,000 shards. For large clusters, several bottlenecks in the admin API interactions with the leader node were identified and optimized in this version.  

These improvements allow OpenSearch Service to maintain the same monitoring frequency for large clusters while using optimal resources (less than 10% CPU and under 75% JVM usage) on the leader node (16-core CPU with 64 GB JVM heap). It also ensures that metadata management operations can be performed on large clusters with predictable latency, without destabilizing the leader node.

---

## Cluster State Overview

To understand the bottlenecks related to the cluster manager, let’s review the cluster state, which is the core operation of the leader node. The cluster state contains the following metadata:

- Cluster settings  
- Index metadata (index settings, mappings, aliases)  
- Routing table and shard metadata (shard allocations)  
- Node information and attributes  
- Snapshot information and custom metadata  

For example, a cluster with six nodes (three managers and three data nodes), one index, and three shards has a cluster state size of around 15 KB. However, a cluster with 1,000 nodes, 10,000 indexes, and 50 shards per index can result in a cluster state size of approximately 250 MB.

---

## Bottleneck 1: Cluster State Communication

Admin APIs such as _cat, _cluster, and _nodes rely on the leader node to retrieve the latest cluster state. In very large clusters, these frequent calls can overwhelm the leader, leading to:

- Increased CPU usage from simultaneous admin API requests  
- High heap memory pressure due to repeated serialization of large cluster states  
- Delays and timeouts caused by busy transport threads  
- Missed heartbeat responses between nodes, triggering unnecessary recoveries  

### Solution: Using Local Cluster State

Each node caches the most recent version of the cluster state. By introducing versioning (term and version numbers), a coordinating node can check if its local state matches the leader’s before making a call. If it is up to date, it serves the API request locally without querying the leader.  

This reduces the load on the leader node and improves cluster stability.

**Impact:** Load tests show up to a 50% reduction in CPU usage on the leader with no increase in API latency, for clusters of 25,000 shards and larger.

---

## Bottleneck 2: Scatter-Gather Nature of Statistics APIs

Statistics APIs such as _cat/indices, _cat/shards, _cluster/stats, and _nodes/stats require data to be collected from multiple data nodes and aggregated by the coordinator node.  

In a 500,000-shard cluster, the combined response size can reach 2.5 GB, leading to:

- High network throughput  
- Memory pressure on coordinator nodes  
- Circuit breaker triggers and “429 Too Many Requests” responses  
- Increased CPU usage due to frequent garbage collection  

### Solution: Local Aggregation and Filtering

Instead of returning complete shard-level statistics, each data node now performs local aggregation and sends only the requested metrics (for example, document count or storage size) to the coordinator.  

This significantly reduces compute and memory overhead on the coordinator nodes.

**Impact:**  
| API | Previous Latency | Optimized Latency |
|------|------------------|------------------|
| _cluster/stats | 15s | 0.65s |
| _nodes/stats | 13.74s | 1.69s |
| _cluster/health | 0.56s | 0.15s |

---

## Bottleneck 3: Long-Running Stats Requests

Previously, even if a user canceled a request or it timed out, the coordinator node continued to process and send internal transport requests to data nodes. This wasted compute and network resources.

### Solution: Cancellation at the Transport Layer

The coordinator now cancels downstream requests once the client timeout expires. This prevents unnecessary load on data nodes and helps the cluster fail gracefully without cascading failures.

---

## Bottleneck 4: Large Response Sizes

Historically, the popular _cat APIs did not support pagination because the metadata size was small when they were designed. For modern large clusters, these unbounded responses can overwhelm coordinator nodes.

### Solution: Paginated List APIs

New paginated list APIs (_list/indices and _list/shards) were introduced as stable, scalable alternatives. They maintain pagination consistency using a combination of index creation timestamps and index names, even when indexes are added or removed.

**Impact:**  
Paginated APIs return consistent, bounded responses for clusters with up to 500,000 shards, ensuring stable response times and reduced resource usage.

---

## Conclusion

Admin APIs are critical for observability and metadata management in Amazon OpenSearch Service. If not optimized, they can become performance bottlenecks in large clusters.  

The improvements introduced in OpenSearch Service version 2.17 provide measurable performance gains for clusters of all sizes—small (20 nodes), medium (200 nodes), and large (1,000 nodes). These optimizations ensure that leader nodes remain stable even during heavy metadata and API operations.  

Features like pagination, cancellation, and local aggregation are extensible and can be adopted for future API enhancements.

---

For more information, see the original post on the AWS Big Data Blog:  
[Scaling cluster manager and admin APIs in Amazon OpenSearch Service](https://aws.amazon.com/blogs/big-data/scaling-cluster-manager-and-admin-apis-in-amazon-opensearch-service/)
