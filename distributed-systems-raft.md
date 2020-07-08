# Raft

## References
1. [In Search of an Understandable Consensus Algorithm](https://pdos.csail.mit.edu/6.824/papers/raft-extended.pdf)
2. [Implementing Fault-Tolerant Services Using the State Machine Approach: A Tutorial](https://www.cs.cornell.edu/fbs/publications/SMSurvey.pdf)

## What is Raft?

Raft is a consensus algorithm for managing a replicated log. It behaves similarly to other replicated logs schemes, but attempts to be simpler and 
easier to implement. Raft accomplishes this by decomposing critical pieces of infrastructure such as leader election, log replication and 
safety in to easy to understand components. It also reduces the state space, which reduces the degree of non-determinism between servers.

A consensus algorithm allows a group of machines to work as a group that can survive the failure of members. Consensus algorithms are 
critical in ensuring the fault-tolerance of a system.

### Specific Raft features

1. **Strong leadership** - Log entries only flow from the leader to other servers. This simplifies the management of the replicated log.
2. **Leader election** - Randomized timers are used to elect leaders, allowing conflicts to be resolved rapidly.
3. **Membership changes** - Server continues operating normally during membership changes due to a *joint concensus* approach to replication.

## Replicated State Machines

- Identical state machines on a collection of servers compute identical copies of the same state.
- Usually implemented with a replicated log which is kept in the same order on all members of the cluster.
  - Comamnds are replicated to all servers, and then executed on all servers.
  - Commands can complete as soon as a **majority** of servers reply. Slow servers should not hold back the
    performance of the cluster.

## The Raft Algorithm

**Distinct Subproblems**

1. **Leader election**
2. **Log replication**
3. **Safety**

**The process**
1. **Elect a distinguished leader**
   1. Leaders manage the replicated log.
   2. Log entries accepted from clients, then replicated to other servers. Finally, the commands are executed on each node.

