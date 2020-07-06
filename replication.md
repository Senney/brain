# Replication

Helps distributed systems deal with "fail-stop" faults. Generally doesn't help 
with bugs or design defects in the system which is being replicated.

## Replication Schemes

### State Transfer

- The entire state of the system is replicated.

### Replicated State Machine

- State is not transfered between replicas, the replication is event driven instead. E.g. arriving input from the outside world.

#### VMWare FT
- A replicated service is run on separate VMM (Virtual Machine Manager).
- Both replicated services share a disk
- Raw events sent to the "primary" are replicated to the "secondary" service.
- Services are essentially run in lock-step
- Output from the "secondary" service is discarded
- If primary disappears, the secondary is immediately promoted to primary and new traffic is shuffled to the new primary service.
- Non-deterministic events are intercepted and the result from the primary are used.
  - We outlaw multi-core processing in this case, as multi-core execution is extremely non-deterministic. E.g. locking - core 1 on the primary gets a lock first, but on the secondary core 2 may get a lock.
