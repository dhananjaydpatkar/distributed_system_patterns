Show a server is available by periodically sending a message to all the other servers in the cluster.

## The Problem
- Servers in a cluster each store and serve a portion of data according to partitioning and replication or can process/compute on the subset of data.
- Fast, reliable failure detection is needed so the cluster can reassign responsibility and continue serving requests.
- Timely detection enables corrective actions (e.g., retry, failover, replica promotion) to maintain availability and data access.


## Brief Design

Periodically send a request to all the other servers indicating liveness of the sending server. Select the request interval to be more than the network round trip time between the servers. All the servers wait for the timeout interval, which is multiple of the request interval to check for the heartbeats. In general, Timeout Interval > Request Interval > Network round trip time between the servers.
e.g. If the network round trip time between the servers is 20ms, the heartbeats can be sent every 100ms, and servers check after 1 second to give enough time for multiple heartbeats to be sent and not get false negatives. If no heartbeat is received in this interval, then it declares the sending server as failed.

The implementation of when to mark a server as failed depends on various criterias. There are different trade offs. In general, the smaller the heartbeat interval, the quicker the failures are detected, but then there is higher probability of false failure detections. So the heartbeat intervals and interpretation of missing heartbeats is implemented as per the requirements of the cluster

###  Small Clusters - e.g. Consensus Based Systems like RAFT, Zookeeper aka centralized
Heartbeats are sent from the leader server to all followers servers. Every time a heartbeat is received, the timestamp of heartbeat arrival is recorded.
If no heartbeat is received in a fixed time window, the leader is considered crashed, and a new server is elected as a leader. 
There are chances of false failure detection because of slow processes or networks. So Generation Clock needs to be used to detect the stale leader.
- Head-of-line blocking on a single socket can delay heartbeat processing and cause false failure detections.
- Use a request pipeline so heartbeats aren't blocked waiting for earlier request responses.
- Long-running tasks (e.g., disk writes in a single update queue) can delay timing interrupts; send heartbeats from a separate asynchronous thread.
- Many frameworks (for example, Consul and Akka) send heartbeats asynchronously to avoid these problems.

###  Large Clusters. Gossip Based Protocols
In large clusters, two things need to be considered:
 - Fixed limit on the number of messages generated per server
 - The total bandwidth consumed by the heartbeat messages. It should not consume a lot of network bandwidth. There should be an upper bound on heartbeat message body, making sure that too many heartbeat messages do not affect actual data transfer across the cluster.

Keeping above point in mind,
- These clusters prefer correct failure detection over speed and will tolerate bounded delays when taking corrective actions (e.g., moving data).
- The goal is to avoid false positives from network latency or slow processes.
- Each process is assigned a suspicion score that increases if it isn’t observed in gossip within a bounded time window.
- The suspicion increment is tuned using historical statistics to adapt to normal delays.
- A process is declared failed only when its suspicion score reaches a configured threshold, reducing erroneous failovers.


## Example
- **Akka Actors and Apache Cassandra upto 3.x** uses **Phi Accrual failure detector**.
- **Apache Cassandra** adopted **Rapid** for membership since version 4.x
- **Hashicorp consul** use gossip based failure detector **SWIM**