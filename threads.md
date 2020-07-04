# Threads

## Why use threads?
- I/O Concurrency
  - A lot of operations on a computer benefit from being able to do other things while waiting for blocking operations like disk or network I/O.
    - e.g. Reading or writing to disk
    - e.g. Waiting for RPC responses from other services
- Multi-core [[parallelism]]
- Convenience
  - Sometimes it's just easier to use a thread if we need to throw something in the background, or have some "side" operations.

----
- One alternative to threads might be to use an event-driven system, where we can split our operations in to small pieces which are activated by events happening in other parts of the system
  - Often a lot easier to write a thread implementation as your code can be linear instead of being chopped up.
----
**golang implementation note**: Goroutines are often multiplexed on top of a single OS level thead.

## Thread challenges

- Shared data
  - e.g. a shared global variable `n` that a program wants to increment (`n = n + 1`)
  - **Race condition**
  - Uses of locks (Mutexes) to protect shared data across threads. Prevent parallel writes, parallel reads
- Coordination
  - Threads are generally not aware of other threads existance.
  - Sometimes we want ways to communicate with or wait for other threads.
  - **golang implementations**
    - Channels - Send and recieve data from other threads
    - sync.Cond - A sort of rendevous point for various goroutines. Allows a thread to wait for a condition to be met in another goroutine before continuing.
    - WaitGroup - Allows the programmer to wait for a set of goroutines to finish execution.
- Deadlock
  - Thread 1 is waiting for Thread 2
  - Thread 2 is waiting for Thread 1
    - e.g. T1 acquires lock A, T2 acquires lock B. Then T1 needs to acquire lock B, and T2 needs to acquire lock A.
- Unbounded thread creation - e.g. crawler use-case
  - When writing a web crawler, we may create a thread for each URL that we want to visit. This may end up creating nearly infinite threads.
  - Usage of worker pools might be more appropriate here?
  - Use a channel oriented approach, where a single controller thread manages the distribution of work to threads.
    - Channels tend to let us move away from using locks since we pass immutable messages from/to workers without sharing memory between them. **Message oriented approach**

## Mutexes

Allow for the arbitrary protection of data. Relies on the programmer to know when/where it is appropriate to
apply a mutex, and to ensure that the appropriate data is protected within the lock. Analysis tooling (e.g 
`-race` in golang) can be used to help detect where locks may be missing, but such tools are not perfect
as they require that the offending code-paths actually be executed during a run of the application to detect
the race. (Run time analysis instead of static analysis).

## Channels

Channels allow for inter-thread communication in a safe way. They avoid common problems with data sharing by
providing immutable, unique copies of data to threads. Channels use a message oriented approach to passing 
data where a channel can have "producers" and "consumers", which send and receive data in a thread safe way.
