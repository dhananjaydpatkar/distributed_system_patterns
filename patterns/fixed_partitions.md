## The Problem
To distribute data across cluster nodes, a uniform mapping and direct node lookup are needed; hashing keys with modulo achieves this. However, when cluster size changes, most data must be remapped, causing large-scale data movement, which is undesirable.


## Design
- Data is first mapped to logical partitions, which are then assigned to cluster nodes.
- The number of partitions is fixed (e.g., 1024) and does not change when nodes are added or removed.
- Data-to-partition mapping remains stable, ensuring consistent access and minimal disruption.
- Partitions should be evenly distributed across nodes for balanced load.
- When nodes change (added/removed), only partitions are reassigned, resulting in limited and fast data movement; the partition count should allow for future growth.

## Key Aspects
**Choice of a hash function**  Use a platform-independent hashing algorithm like MD5 or Murmur to ensure consistent hash values across different runtimes.
**Mapping Partitions to Nodes** A dedicated Consistent Core coordinates the cluster by tracking nodes and managing the mapping of partitions to nodes, making this information accessible to clients.



## Examples
- In **Kafka** each topic is created with a fixed number of partitions.