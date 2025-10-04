## Problem
The Leader and Followers pattern can lead to **leader overload and increased latency**, especially in multi-datacenter or multi-region setups where clients must route requests to the leader.

## Design
- Serve read requests from followers to achieve better throughput and lower latency.
- Write requests must go to the leader for consistency, but read-only requests can be served by the nearest follower, reducing latency.
- This approach is especially beneficial for read-heavy workloads.
- Clients reading from followers may receive stale data due to replication lag, Reading from followers is suitable only when applications can tolerate slightly outdated values.

## Key Aspects
**Finding nearset replica** -  Cluster nodes maintain additional metadata about their location. Cluster client can then pick up the local replica based its own region
**Follower with minimum latency** - The cluster client or a co-ordinating cluster node can also track latencies observed with cluster nodes. It can send period heartbeats to capture the latencies, and use that to pick up a follower with minimum latency
**Disconnected or Slow followers** - Followers may stop serving requests if they lose contact with the leader or fall too far behind due to slow updates or hardware issues (slow disk IO).
**Causal Consistency** - When an event A in a system happens before another event B, it is said to have causal relationship. This causal relationship means that A might have some role in causing B. For a data storage system, the events are about writing and reading values. To provide causal consistency, the storage system needs to track happens-before relationship between read and writeevents
**Linearizable Reads** -  The replication lag cannot be tolerated. In these cases, the read requests need to be redirected to the leader mostly using consistent core pattern. 


## Examples
- **MongoDB** maintains **causal consistency** in its replica sets. The write operations return an operationTime; this is passed in the subsequent read requests to make sure read requests return the writes which happened before the read request.
- **Kafka** allows consuming the messages from the follower brokers. The followers know **High-Water Mark at the leader** aka offset (up to which replication is completed). In kafka's design, instead of waiting for the latest updates, the broker returns a OFFSET_NOT_AVAILABLE error to the consumers and expects consumers to retry.