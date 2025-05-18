---
title: "Getting distributed Consensus using quorum based commit protocol"
author: Rishabh Bhatia
categories: [distributed-systems]
tags: distributed systems design paper Consensus
date: 2025-05-18 05:00:00 -0700
---

We are on a path to build a strong foundation in distributed systems. We have already gone over distributed time; the
next topic we will cover is Distributed Consensus. To build the foundation on distributed consensus, we will go over the
the paper 'A Formal Model of Crash Recovery in a Distributed System' published in 1983 by Skeen and Stonebraker. The purpose
to understand this paper is to learn how to formalize the crash recovery problem in distributed database environment. This 
will set foundation on how to think and build mathematical framework for crash recovery problems.

## Pre-requisite
Please go over basics of transactions, 2PC and Quorum based commit protocol for better understanding. My explanation 
and learnings on the topic can be found [here](https://relentless-leader.com/a-quorum-based-commit-protocol.html).

## Background
A transaction in distributed database systems is a logically atomic operations i.e. it must be present in all nodes 
or at none of them. Handling atomic operations of commit and abort in a single node is a well understood problem.

In multiple node (distributed environment) it the task of the **commit protocol** to enforce global atomicity. The basic 
purpose of commit protocol is getting consensus for all nodes to come to one decision of either committing or aborting 
the transaction.
The challenge that commit protocol has to solve is getting the consensus in the face of failures e.g. node failures,
network failures, transaction deadlock with another transaction, etc.

The fundamental idea in this paper is **before any node can commit the transaction, all nodes must give up the right to 
unilaterally abort it**. Once node gives up the right, it can abort the transaction only in concordance with other sites.

## Motivation
The most basic commit protocol that allows nodes to unilaterally abort the transaction is 2-phase commit protocol. As 
discussed in [previous blog](https://relentless-leader.com/a-quorum-based-commit-protocol.html), it is a centralized
protocol with system getting blocked until the failures in coordinator is repaired. 

Author argues that certainly blocking is an option to preserve consistency. It is an undesirable because the locks 
acquired by blocked transactions can never be relinquished.   
This motivates the author to focus on non-blocking protocol where operational nodes never suspend because of failure. 
Author argues that the resilient protocol should always terminate irrespective of failures.  


## Transaction Model
### Network Assumptions
The network provides point to point communication i.e. any node can send a message to any other node. It assumes
that either the message is delivered successfully within the time T or it reports a timeout to the sender.

### Finite state Automata
The commit protocol can be specified in terms of non-deterministic finite state automata for each node. The final state
will be either abort or commit to end the transaction.

![CrashRecovery-definition of transaction model.drawio.png](/assets/distributed%20system/consensus/CrashRecovery-definition%20of%20transaction%20model.drawio.png)

### FSA for 2-phase commit protocol's Coordinator node

![CrashRecovery-2PC Coordinator Node FSA.drawio.png](/assets/distributed%20system/consensus/CrashRecovery-2PC%20Coordinator%20Node%20FSA.drawio.png)

### FSA for 2-phase commit protocol's Worker node

![CrashRecovery-2PC Worker Node FSA.drawio.png](/assets/distributed%20system/consensus/CrashRecovery-2PC%20Worker%20Node%20FSA.drawio.png)

### Global Transaction State for 2-phase commit protocol 
Global state defines the complete process state of a transaction. It consists of all possible states that can happen at the same time.

![CrashRecovery-2PC - global states.drawio.png](/assets/distributed%20system/consensus/CrashRecovery-2PC%20-%20global%20states.drawio.png)

## Learnings


## References
1. [A Formal Model of Crash Recovery in a Distributed System by Skeen and Stonebraker in 1983](https://www.inf.fu-berlin.de/lehre/SS10/DBS-TA/Reader/3PCSkeenStonebr.pdf)