## The Problem
The challenge is to efficiently disseminate metadata to all nodes in a large cluster, minimizing network bandwidth (avoid flooding of messages) and ensuring delivery even if some network links are unreliable.

## Design
Each cluster node uses gossip-style communication: at regular intervals (e.g., every second), it randomly selects a small set of nodes (gossip fanout) to share its metadata.
- At startup, each node adds its own metadata (such as IP, port, and partition responsibility) and must know at least one seed node (introducer) to begin gossiping; any node can act as an introducer.
- Each node schedules a job to transmit its metadata at regular intervals, picking random targets from the metadata map; if nothing is known, it contacts a seed node.
- Upon receiving a gossip message, a node compares the incoming metadata with its own:
	- Adds values present in the message but missing locally.
	- Identifies values it has that are missing in the incoming message.
	- For overlapping keys, keeps the value with the higher version.
- The node updates its state with new values and returns any missing values to the sender as a response.
- The sender then updates its own state with any new values from the response.
- This periodic, randomized exchange ensures efficient, robust, and scalable metadata dissemination across all nodes in large clusters, even in the presence of failures.

## Key Aspects
- **Avoiding unnecessary state exchange**  The cluster node just needs to send the state changes since the last gossip. For achieving this, each node maintains a version number which is incremented every time a new metadata entry is added locally.
- **Criteria for node selection to Gossip**  Cluster nodes randomly select the node to send the Gossip message. There can be other considerations such as the node that is least contacted with (CockrochDB). Also there are network topology aware criterias to select nodes(like zones, regions)
- **Handling node restarts** Versioned values alone are insufficient after node crashes or restarts, as in-memory state is lost and nodes may have conflicting values for the same key. Using a **Generation Clock** with each value helps nodes detect and track state changes more reliably, even across restarts.

## Examples
- **Apache cassandra** uses Gossip protocol for the group membership and failure detection of cluster nodes. Metadata for each cluster node such as the tokens assigned to each cluster node, is also transmitted using Gossip protocol.
- **CockroachDB** uses Gossip protocol to propagate node metadata.
- **Blockchain** implementations such as **Hyperledger Fabric** use Gossip protocol for group membership and sending ledger metadata