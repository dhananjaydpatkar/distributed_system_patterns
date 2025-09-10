## The Problem

In distributed systems, each node (server) has its own clock, and these clocks can drift apart. Even with synchronization protocols (like NTP), there is always some uncertainty about the exact time at each node.

If you rely on timestamps for ordering events or ensuring consistency, this uncertainty can cause problems, such as:

- Events appearing out of order
- Data conflicts
- Security issues (e.g., token expiry)

## The Bounded Clock Concept

Instead of assuming each node’s clock is perfectly accurate, each node keeps track of the earliest and latest possible real time (a time interval or "bound") that its clock could represent, given the known maximum error (skew):

- **Earliest time:** The node’s clock minus the maximum possible error.
- **Latest time:** The node’s clock plus the maximum possible error.

This way, every timestamp is not a single value, but a range.

## How Consistent Read/Write Operations Work in a Cluster

### Write Operations

**Assigning a Timestamp:**

- When a node wants to write data, it requests the current time interval: `[earliest, latest]`.
- The system assigns a commit timestamp that is guaranteed to be within this interval.

**Ensuring Consistency:**

- To guarantee that no other write with a later timestamp could have happened before, the system may wait out the uncertainty window (the ε) before confirming the write.
- This ensures that the commit timestamp truly reflects the order of events.

**Replication:**

- The write, along with its timestamp, is replicated to other nodes.
- All nodes use the timestamp to order writes, even if their local clocks differ.

### Read Operations

**Reading at a Timestamp:**

- When a client requests a read, it can specify a timestamp (e.g., “give me the value as of time T”).
- The system checks if it can safely serve the read at T, given the current uncertainty bounds.

**Serving Consistent Reads:**

- If the node’s latest possible time is after T, it knows all writes up to T have been seen, and it can serve the read.
- If not, it may wait until it is sure that all relevant writes have arrived (i.e., until the uncertainty window passes).

## Examples

- **Google's TrueTime API:** Provides a clock bound. Spanner uses it with commit-wait to implement consistency.
- **AWS Time Sync Service:** Ensures minimal clock drifts. The ClockBound API can be used to implement waits to order events across the cluster.
- **CockroachDB:** Implements read restart and has an experimental option to use commit-wait based on the configured maximum clock drift value.
- **YugabyteDB:** Implements read restart based on the configured maximum clock drift value.