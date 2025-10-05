## The Problem
In a Leader and Followers setup, a temporarily disconnected leader may unknowingly send outdated replication requests after rejoining, risking data inconsistency if a new leader has already been elected; both the cluster and the old leader must detect and handle this situation to maintain correctness and cosistency.

## Design
- Use a monotonically increasing generation number to track leader elections.
- Store the generation in the Write-Ahead Log so it persists across server restarts.
- On startup, servers read the last generation from the log to stay consistent.
- During leader election, servers exchange and update the generation number to ensure agreement.
- The leader includes the generation in all heartbeat and replication messages to followers.
- Followers and leaders persist the generation with every log entry for durability.
- If a follower receives a message with an outdated generation, it rejects it; the sender then steps down and becomes a follower.

## Example
- **Apache Cassandra** each server stores a generation number which is incremented every time a server restarts. The generation information is persisted in the system keyspace and propagated as part of the gossip messages to other servers. The servers receiving the gossip message can then compare the generation value it knows about and the generation value in the gossip message. If the generation in the gossip message is higher, it knows that the server was restarted and then discards all the state it has maintained for that server and asks for the new state.