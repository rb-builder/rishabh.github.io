---
title: "Getting distributed Consensus using quorum based commit protocol"
author: Rishabh Bhatia
categories: [distributed-systems]
tags: distributed systems design paper Consensus
date: 2025-05-17 05:00:00 -0700
---

We are on a path to build a strong foundation in distributed systems. We have already gone over distributed time; the
next topic we will cover is Distributed Consensus. To build the foundation on distributed consensus, we will go over the
the paper 'A Quorum-Based Commit Protocol' published in 1982 by Dale Skeen.
This paper builds on the improves on work done on 2 phase commit. Quorum-based commit protocols is a way to get 
distributed consensus between participating database nodes by requiring a minimum number of nodes (quorum) to agree on
transaction commit/abort decisions to ensure atomicity and consistency. 


## Understanding Transactions
A transaction is an atomic operation on a distributed database system. Either all changes by transactions are persisted 
i.e. committed or nothing is persisted i.e. aborted. It is the task of the commit protocol to ensure that a transaction is atomically executed.   

Distributed database need to build transactions that are resilient to multi failure scenarios inherited from 
distributed systems. The common failures are -
1. Arbitrary node failures
2. Lost messages 
3. Network partitioning

It is the responsibility of a **commit protocol** to ensure that all sub-transactions ( transaction request sent to 
worker nodes) are consistently committed or aborted. A **commit protocol** can be described as a set of state diagrams,
one for each participating nodes (example - coordinator and workers). We will Finite State Automata to draw these state
diagram, in which each state will define the local transactions state of that node. 

## Understanding 2 Phase Commit
The two phase commit protocol is a centralized protocol where coordinator gets all transactions requests, and it coordinates
with the worker nodes to come to a binary decision (commit or abort) on the transactions.   

The bellow diagram are self-explanatory -
### 2 PC protocol

![ 2 PC protocol ](/assets/distributed%20system/consensus/Quorum%20Based%20Commit%20Protocol-2%20PC%20protocol.drawio.png)    

### 2 PC State Diagram
![ 2 PC state diagram ](/assets/distributed%20system/consensus/Quorum%20Based%20Commit%20Protocol-2%20PC%20state%20diagram.drawio.png)


### Problem with 2 Phase Commit
The two-phase commit protocol support limited set of failures such as temporary network failures or 
worker node failing before vote.

**Failure scenarios that are not supported by 2 PC**
1. Coordinator Failures - If coordinator fails after collecting votes but before sending decision. Workers are 
blocked until coordinator recovers
2.  Network Partitions - If network split prevents communication between coordinator and worker. System cannot reach 
consensus, leading to a blocked state.
3. Simultaneous Failures - Multiple nodes failing at same time, for example - coordinator and worker failing at same time
leads to inconsistent states.
4. Byzantine Failures - 2PC assumes nodes to be honest and only fail by stopping.

The main limitation of 2PC is that it's blocking, if certain failures occur at critical points, the system must wait for
recovery rather than proceeding with the protocol.

## Quorum based commit protocol
As the name suggests, once the group of communicating nodes establishes a quorum, they are allowed to proceed with transaction.
Note that I said group of communicating nodes, as it allows protocol to gracefully handle network partitioning.

Each node will be assigned a non-negative integer of votes. If the node is 0 means it has no say on the voting process. 
Let   
- V = total number of votes
- V<sub>c</sub> = number of required votes for commit protocol
- V<sub>a</sub> = number of required votes for abort protocol

![Quorum Based Commit Protocol- algo properties.png](/assets/distributed%20system/consensus/Quorum%20Based%20Commit%20Protocol-%20algo%20properties.png)

> Skeen also defined sub-requirements for commit:
> - (2.1) Before the first site commits, a commit quorum of sites in committable states must be obtained, and
> - (2.2) After any site has committed, a commit quorum must be maintained.
> and similar requirements for abort protocol

The basic idea is to avoid tie, we need to quorum intersection i.e. V<sub>c</sub> + V<sub>a</sub> > V. First, a "prepare to commit" 
request is sent to all nodes. Each node will try to reach the committable state (completing all preparations and 
securing resources) and then cast their vote. When V<sub>c</sub> votes are collected through quorum and the voting 
constraint V<sub>c</sub> + V<sub>a</sub> > V is maintained, a commit decision can be made and sent to all nodes. 
The protocol ensures that nodes properly prepare, vote, and maintain consistency through the voting constraint before
any final commit decision.

![Quorum Based Commit Protocol-resilient commit protocol.drawio.png](/assets/distributed%20system/consensus/Quorum%20Based%20Commit%20Protocol-resilient%20commit%20protocol.drawio.png)

**Note:** In the traditional Two-Phase Commit (2PC) protocol, there isn't a specific "committable" state between the 
prepare and commit phases. **Skeen's contribution was significant** because he introduced this important intermediate
"committable" state concept, which occurs before the actual commit.

![Quorum Based Commit Protocol-resilient commit protocol state diagram.drawio.png](/assets/distributed%20system/consensus/Quorum%20Based%20Commit%20Protocol-resilient%20commit%20protocol%20state%20diagram.drawio.png)

## Quorum based recovery protocol
Skeen focused the paper on solving the most difficult failure scenario i.e. network partitioning. He introduced handling
of network partitions through:
1. Termination Protocol for achieving quorum toe terminate the transaction. 
2. Merge Protocol for handling cases when quorum can't be achieved

### Termination Protocol
The focus here is to reach successful termination of partially execute termination. When the group of nodes detects that it 
is partitioned. It will - 
1. Elect a new **surrogate coordinator**.
2. New coordinator will poll the state of each worker node.
3. If any worker node is in commit or abort state, new coordinator will ask other nodes to do the same.
4. If any worker node is in or has request to prepared to commit state. This tells every node is willing to commit -
- If at least V<sub>c</sub> are either committable or waiting state, then send "prepare for commit" request. 
- If number of votes is >= V<sub>c</sub> then commit else wait.
- Same quorum method can be for aborting.

![Quorum Based Commit Protocol-termination protocol.drawio.png](/assets/distributed%20system/consensus/Quorum%20Based%20Commit%20Protocol-termination%20protocol.drawio.png)


### Merge Protocol
Merge protocol is simple. If the quorum cannot be achieved, the nodes will wait till the communication between nodes is established.
Once the failure is repaired, just execute the termination protocol again. 

## Learnings
**Quorum based protocols is one of the most significant contributions in reaching distributed consensus.**
It is used all over the place and laid the foundation for quorum-based techniques used in:
- Paxos and Raft (consensus algorithms)
- Distributed databases like Spanner, Cassandra, and Dynamo
- Modern replication and coordination systems (e.g., ZooKeeper, etcd)
- Object storage systems (e.g., S3)

**Key takeaways**
- **Key idea I took away** is to have algorithm that can support decentralized decision making is powerful. Even though the paper
  was focused on coordinator/leader based system, and chose new one during partition.
- **Key idea I took away** is coordinator (leader) election technique to handle consensus or concurrent access to shared state is
a valid and **simple solution**. It also has a **Scalability issues** and **large blast radius** with complete system going 
into block state till failure is repaired. This makes us think more about the problem and trade-offs.

**Other learnings**
- "Prepare to commit" state is a very useful and practical way to reach consensus and is resilient to network partitions.
- Skeen’s protocol helped transition from coordinator-based blocking schemes to consensus-based fault-tolerant commit mechanisms.
- **Quorum Intersection** is a simple principle that all “commit” and “abort” quorums must overlap underpins virtually
every major distributed‑consensus and replication protocol since — Paxos, Raft etc.


## References
1. [A Quorum-Based Commit Protocol by Dale Skeen in 1982](https://ecommons.cornell.edu/server/api/core/bitstreams/a4d6f5b4-0d7d-444e-8994-752e9ab6ff8f/content#:~:text=The%20basic%20idea%20is%20that,quorum%20and%20an%20abort%20quorum.)