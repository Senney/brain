## GFS

**Reference** https://pdos.csail.mit.edu/6.824/papers/gfs.pdf

Serves as the underlying storage layer for [[distributed-systems-map-reduce]].

- Designed for [[fault-tolerance]]. At scale, system failures are the norm rather than the exception.
  - distributed file storage must be designed to handle this kind of failure.
  - monitoring, error detection, and recovery
- [[performance]] - large files are generally preferred for data-intensive workloads rather than many smaller files.
- most writes are append-only instead of random write, then many reads
  - gfs support concurrent appends from many producers
- Architecture
  - Single master, multiple chunkservers
  - Files divided in to fixed-size chunks
    - Each chunk given a 64 bit identifier (chunk handle)
    - Each chunk is replicated to multiple chunkservers
    - Chunk size is 64MB (quite large!)
      - Allows a client to generally request all chunk metadata about a file even if it's multiple GB or TB
      - 1 tb is only 16,384 chunks
      - Allows the master to keep all chunk data in memory since there's less chunks
    - Downside of large chunk sizes is that small files will generally only inhabit one chunk. If many consumers need that chunk, problems quickly arise at smaller replication factors as chunkservers are quickly overwhelmed by large numbers of requests
  - Master maintains all file system metadata, such as namespaces, ACLs, and chunk=>file maps.
  - Clients interact with the master to acquire metadata about files, but talk directly to chunkservers to retrieve actual file data.
    - Client asks for (filename, chunkIdx)
    - Master replies with (chunkHandle, chunkLocation)
      - chunkLocation will generally be a list of replicas which contain the chunk. Client determines the closest
  - Master Architecture
    - All metadata is kept in memory at all times
      - Under 64 bytes of metadata per chunk, so it's pretty reasonable to keep the state of an entire 300TB+ system in memory
    - Namespaces + file/chunk mapping are persited by an append-only log which is stored on disk and replicated to other machines
      - Chunk location information not stored persistently. Queries chunkservers at startup to determine what chunks they have
        - Avoid issues of having to keep master and all chunkservers in sync
        - Chunk data is periodically re-fetched from the chunkservers
    - Periodic background scans of metadata state
      - Garbage collection
      - Re-replication of data
      - Chunk migration
    - Operation log
      - Append-only log with checkpoints
      - Only respond to requests after the log record has been flushed to disk, locally and replicated.
      - Format on-disk is the same as the in-memory format
        - B-tree like form
      - Checkpoints can be created without halting mutations to the cluster
        - Writes to the log are switched to a new file and the old file is procssed in a separate thread
      - Recovery needs only the last checkpoint and the latest log file(s) since that checkpoint.
        - Uncompleted checkpoints are handled by the recovery logic
  - Consistency Model
    - In append-only mode, writes are delivered **at least once**, which means there is the possibility for duplicate chunks to be written.
    - Normal offset-based writes don't provide much in terms of consistency guarantees. If multiple writers are writing to a region concurrently, generally parts of all of their data end up in the file. Consistency of the result is maintained across all chunkservers, but the result itself is undefined.
    - Replicas which have missed events from the operation log are considered stale, and are garabge collected. Clients will not be served chunks from these servers, but some clients which may have cached the chunkservers for a file may access stale data in that time.

- System interactions
  - Leases and Mutation Order
    - **Mutation** - An operation which changes the contents or metadata of a chunk. E.g. a write or append.
    - The effects of mutations are performed on each of the replicas for a chunk.
    - Flow of a write
      - Client asks about the current leaseholder for a chunk. If no lease is currently granted, the master chooses a replica and grants it a lease.
        - Leases are granted for 60 seconds by default, and then extended on request from the lease-holder.
      - Master informs client of the primary, and location of the secondaries.
      - Client pushes data to all of the replicas. Data is stored in a cache until a "commit" is issued from the Primary for that data.
        - Replicas will push data to each other as well to speed up propagation.
        - Data is pipelined, so received bytes are immediately forwarded to other nodes.
      - Once writes to secondaries have been confirmed, a write command is sent to the primary, and sends the identity of the data which was written to the secondaries. Primary assigns consecutive serial numbers to all mutations, then applies all mutations against its local state. This can include writes from many writers.
      - Write request is then forwarded to all replicas and the same mutation application process is used.
      - Failures are forwarded to the client to allow the mutations to be re-sent to the secondaries. If the operation continues to fail, the mutation can be started again from the beginning.
        - Offloading the complexity to the client makes maintaining integrity of the data more complicated imo
        - This is especially interesting because it means that an append could occur multiple times, meaning that the copies of the file across all replicas is not **bytewise identical**, just that it contains each **record** at least once.
    - Lease expiration
      - Allows the master to designate a new primary
      - The server which has been granted the lease will stop accepting requests as the primary once the lease expires.

[//begin]: # "Autogenerated link references for markdown compatibility"
[fault-tolerance]: fault-tolerance "Fault Tolerance"
[performance]: performance "Performance"
[distributed-systems-map-reduce]: distributed-systems-map-reduce "Map Reduce"
[//end]: # "Autogenerated link references"