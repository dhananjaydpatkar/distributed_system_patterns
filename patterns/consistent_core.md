## The Problem

When a cluster needs to handle a lot of data, it requires more servers. In such clusters, there are common requirements, such as:

- Selecting a special server to be the master for a particular task
- Managing group membership information
- Mapping data partitions to servers

These requirements need strong consistency guarantees, specifically **linearizability**. The implementation must also be fault tolerant. A common approach is to use fault-tolerant consensus algorithms based on **quorum**. However, in quorum-based systems, throughput degrades as the cluster size increases.

## Consistent Core Design

A practical solution is to implement a smaller cluster (3 to 5 nodes) that provides linearizability and fault tolerance. This small, consistent cluster manages metadata and makes cluster-wide decisions using primitives like **lease**. The larger data cluster can then scale to many servers, while still relying on the smaller metadata cluster for actions that require strong consistency.

## Key Aspects

- **Strong consistency** at the core, where it matters most
- **Scalability and speed** for data nodes
- **Fault tolerance** in core nodes to ensure critical metadata survives failures
- **Redundancy** in data nodes to ensure availability

## Examples

- **Kafka**: Previously used ZooKeeper to manage metadata and leader election for the cluster master. The latest versions use a Raft-based controller cluster.
- **Kubernetes**: Uses [etcd](https://etcd.io/) for coordination, cluster metadata management, and group membership.
- **Big Data Systems**: Systems like [HDFS](https://hadoop.apache.org/docs/r1.2.1/hdfs_design.html) and [ZooKeeper](https://zookeeper.apache.org/) use similar patterns for high availability and cluster coordination.
