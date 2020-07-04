# Fault Tolerance

Economies of scale reveal low probability issues quickly. 1000 computers may see multiple device failures per day.

## What does it mean to be fault tolerant?

- **Availability** - Under certain classes of failures, the system will continue to operate.
  - e.g. AZ outage in AWS
- **Recoverability** - After an issue has been resolved, the service may continue as though nothing had gone wrong.
  - Without loss of correctness.
  - e.g. databases writing to disk and replicating before confirming a transaction
- **Replication** - Keeping copies of data across multiple systems, keeping this data in sync.
  - How to we ensure that replicas don't drift?
- **Consistency** - How do you ensure that all replicas resolve the same value when things change?
  - How do we ensure that replicas remain consistent if a component crashes?
  - Strong vs weak consistency
    - Weak - You may see an old value for an unbounded amount of time
    - Strong - You will always see the most recent value.
      - Much harder to implement strong consistency.
      - Computationally more expensive.
      - A lot harder to replicate data to other availability zones.
