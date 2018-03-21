Kudu memo
=========

* Externally consistent transactions in the sense where all transactions are linearizable

## Hybrid Time

* Assumption 1. Physical clock error from reference clock is bounded
* Assumption 2. Physical clock timestamps are monotonically increasing

Physical Clock API::

 i :server_id()

 PC.Now:
  physical = PC_i(now)
  ε = E_i (physical)
  return physical, ε


where ``PC_i(now)`` is a clock build in the server and ``E_i( )`` is an
upper bound of physical clock time skew, from reference time
``PC_ref()``.

HTC API::


 last_physical = 0
 next_logical = 0

 HTC.Now():
  Timestamp now
  int cur_physical = PC.Now().physical

  if cur_physical >= last_physical then
    now.physical = cur_physical
    now.logical = 0
    last_physical = cur_physical
    next_logical = 1
  else
    now.physical = last_physical
    now.logical = next_logical
    next_logical++
  end if
  return now

 HTP.Update(Timestamp in)
  Timestamp now = Now()
  if now.physical > in.physical then
    return
  end if
  last_physical = in.physical
  next_logical = in.logical + 1
  return


* Theorem 1. The HybridTime clock happened-before relation forms a total order of events
* Q. This is **not trivial**, you cannot order e and f when ``PC_i(e) -
  PC_j(f) < E_i(e) < E_j(f)`` ...
* Theorem 2. For any causal chain f the physical component of HTC
  timestamp approximates the "real" time the event occurred, with a
  bounded error ``[-E_j(e), E_i(f)]``

## Types of Transactions

* Read-write transactions
* Snapshot transactions
* Time-travel read

Write transactions that span multiple RG (Replica group)s needs one RG
leader that plays transaction manager.

1. *Prepare*: TM sends all related RGs making them aquiring locks, responding each HTCs
2. TM chooses highest HTC and sends them as *Apply* to RG leaders

My questions: bounded error of time at f does not account the
feasibility of Read-write or Write-write transactions. Details are not
dictated, which might be future work here. Also, how is interleaving
select-update on ``R(x)->W(y)`` at T1 and ``R(y)->W(x)`` going to be
handled?

## Spanner

* Lock-free distributed read transactions
* External Consistency - commit order respects global wall-time order (aka linearizability)
* TrueTime

* Transactions that writes use 2PL, 2PC
* Timestamp order == commit order
* **Commit wait** : Pick timestamp s = TT.now().latest where TT.now().earliest > s by average ε

+--------------------+--------------------+--------------------+--------------------+
|Operation           | Timestamp          | Concurrency Control|Replica Required    |
|                    | Discussion         |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
|Read-Write          |4.1.2               |pessimistic         |Leader              |
|Transactions        |                    |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
|Read-Only           |4.1.4               |lock-free           |leader for          |
|Transactions        |                    |                    |timesatmp; any for  |
|                    |                    |                    |read                |
+--------------------+--------------------+--------------------+--------------------+
|Snapshot Read,      |---                 |lock-free           |any                 |
|client-provided     |                    |                    |                    |
|timestamp           |                    |                    |                    |
+--------------------+--------------------+--------------------+--------------------+
|Snapshot Read,      |4.1.3               |lock-free           |any                 |
|client-provided     |                    |                    |                    |
|bound               |                    |                    |                    |
+--------------------+--------------------+--------------------+--------------------+

Table: types of reads and writes in Spanner, how they compare

### Invariants

* For single directory, all leaders' lease period is disjoint
* All timestamps that any leader issues are unique

### Read-only transactions

* Assign a read timestamp at group leader

### Read-write transactions

* 2PC between group (main group leader becomes the transaction manager)
* Wound-wait: later transactions wait for earlier transaction - (earlier transactions don't wait for them, just abort)
* All commits and writes are logged via Paxos
* The group leader does commit-wait (ensure the commit timestamp exceeds the ε)

## Calvin

* "Deterministic Distributed Transactions" scheduler on top of distributed storage
* half million TPS at TPC-C benchmark

Sequencer
* 10ms epoch, all requests are tied up as a batch at the end of epoch
* sequencer's unique ID, epoch number, all transaction inputs
* => every scheduler to have its own global view of transaction order per epoch

Replication
* asynchronous and synchronous (Paxos-based) replication
* one replica is designated as a *Master* replica
** Sequencer failure is complexed - which batch was the last valid batch sent out?, what transactions did that batch contained?
** ZooKeeper provided necessary throughput for replicating transactional inputs

Concurrency Control
* Virtual locking could be used ... future work
* Instead lock manager is responsible for data on the same data node
* The lock order must be strictly kept. Late transactions must wait
* *active* / *passive* participants
* local reads => remote reads => local writes (remote writes are done ate remote notes with partial view)
* simplify -> abort -> retry? / 2i is very simple

Pitfall
* A stalled transaction write may block unrelated following transacations
* If it's predicted, put an artificial delay before and perform 'warm-up' at disk->memory

* "No need to pay the overhead of physical REDO logging" eh? instead replaying history of transactional input...

Experiment
* Contention index?
* scale up to 4-8 nodes if contention = 100%

Questions
* Partial updates? That my happen if a transaction is the combination of partial updates
* What if during the transaction a master replica, or sequencer failed ... failover?
* Also, how to avoid deadlocks is not clear; clocks goes sometimes wrong and sequencer gives lock to later transaction



## Resouces

* [Repo](https://github.com/cloudera/kudu)
* [Hybrid Time paper](http://pdsl.ece.utexas.edu/david/hybrid-time-tech-report-01.pdf)
* [Transaction Semantics](https://github.com/cloudera/kudu/blob/master/docs/transaction_semantics.adoc)
* [Calvin: Calvin: Fast Distributed Transactions for Partitioned Database Systems](http://cs-www.cs.yale.edu/homes/dna/papers/calvin-sigmod12.pdf)
* [Spanner paper](http://research.google.com/archive/spanner.html)
