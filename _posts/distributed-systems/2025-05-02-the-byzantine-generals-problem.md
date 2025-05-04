---
title: "Understanding distributed consensus from the Byzantine generals problem"
author: Rishabh Bhatia
categories: [distributed-systems]
tags: distributed systems design paper time clocks
date: 2025-05-02 10:00:00 -0700
---

We are on a path to build a strong foundation in distributed systems. We have already gone over distributed time; the
next topic we will cover is Distributed Consensus. To build the foundation on distributed consensus, we will start with
the paper 'The Byzantine Generals Problem' published in 1982 by Leslie Lamport, Robert Shostak, and Marshall Pease.
This paper introduces the concept of Byzantine faults and formally proves the conditions under which consensus 
is impossible in the presence of arbitrary (malicious or faulty) behavior.

## Models in Distributed system
While building any complex distributed system, we start with capturing assumption on how nodes/servers and networks 
connecting them behave. These assumptions are usually captured in system model. 
To understand system model we can use two classic thought experiments in distributed systems are: 
the two generals problem and the Byzantine generals problem.

## The Two Generals Problem
There are two armies trying to capture a city. Since city defences are strong, both the armies need to attack 
simultaneously to capture the city. The assumption is message will not be tempered on the way and same content will be delivered.

The problem here is message can be lost on the way.

Bellow is the image sourced from [sketch planations](https://sketchplanations.com/the-two-generals-problem) showcasing the thought
experiment -
![ The two generals problem ](/assets/distributed%20system/consensus/sketchplanations-the-two-generals-problem.png)

"The Two Generals Problem", highlights the core issue: **reliable agreement is impossible over an unreliable channel**
— even if both generals want to coordinate an attack.

This table shows the possible outcomes based on whether General A's message and the acknowledgment from General B are successfully delivered:

<table>
<caption>The Two Generals Problem – Truth Table</caption>
<thead>
  <tr>
    <th>Message from A to B Delivered?</th>
    <th>Acknowledgment from B to A Delivered?</th>
    <th>General A Attacks</th>
    <th>General B Attacks</th>
    <th>Coordinated Attack?</th>
    <th>Result</th>
  </tr>
</thead>
<tbody>
  <tr>
    <td>Yes</td>
    <td>Yes</td>
    <td>Yes</td>
    <td>Yes</td>
    <td>Yes ✅</td>
    <td>City Captured</td>
  </tr>
  <tr>
    <td>Yes</td>
    <td>No</td>
    <td>No</td>
    <td>Yes</td>
    <td>No ❌</td>
    <td>General B Army defeated </td>
  </tr>
  <tr>
    <td>No</td>
    <td>Yes</td>
    <td>Yes</td>
    <td>No</td>
    <td>No ❌</td>
    <td>General A Army defeated </td>
  </tr>
  <tr>
    <td>No</td>
    <td>N/A</td>
    <td>No</td>
    <td>No</td>
    <td>No ❌</td>
    <td>Nothing Happens </td>
  </tr>
</tbody>
</table>

Since the network is unreliable it is impossible for General A to tell if the message is delivery and then lost, or message
was not delivered.
![ The two generals problem ](/assets/distributed%20system/consensus/Distributed%20Consensus-the%20Two%20Generals%20Problem.drawio.png)


### **Thought Experiment**
What protocol should the two generals use to agree on a plan ?
1. If general A attack irrespective of acknowledgement received ? 
General A can try to send lots of messages to increase the probability that one will reach the other general.
If all messages are lost, the general A's army will go on attack alone.

2. General A only attacks if positive response from general B is received?
Now General A is saved, but General B does not know if the ack is received by general A. The situation is reversed and the
same problem is shifted to general B.

### Learnings from this thought experiment
**The core problem** is no matter how many messages are exchanged, generals cannot be certain of co-ordination.

**This thought experiment** demonstrates that in a distributed system, there is no way for one node to have certainty
about the state of another node. The only way how a node can know something is by having that knowledge communicated in
a message. 

### The practical Example of two generals problem
![ The two generals problem - practical example](/assets/distributed%20system/consensus/Distributed%20Consensus-The%20Two%20Generals%20Problem%20Practical%20Example.drawio.png)

The online shop has to dispatch the goods, if and only if payment goes through. To solve the problem in this scenario 
where customer gets charged but order did not go through. The bank will refund the customer i.e. rollback to previous state.

The fact that a payment is something that can be rolled back (unlike an army being defeated) makes the problem solvable.



## The Byzantine Generals Problem
The Byzantine generals problem is similar to two generals problem. In this case, there are three or more armies trying to
capture the city. Messages between the armies will be communicated by messengers. In this case the assumption is message
is always delivered.   
The problem here is the messages can be manipulated by the traitor general.


In the research paper by lamport they used commander, and lieutenants to describe the problem. So we will use the same
to describe the situation:

![ The Byzantine generals problem](/assets/distributed%20system/consensus/Distributed%20Consensus-The%20Byzantine%20Generals%20Problem.drawio.png)

### **Thought Experiment**

The generals must have an algorithm to guarantee that :
**A. All loyal generals decide upon the same plan of action.**
**B. A small number of traitors cannot cause the loyal generals to adopt a bad plan.**


and more formally defining Byzantine Generals Problem. A commanding general must send an order to
his n - 1 lieutenant generals such that -
IC1. All loyal lieutenants obey the same order.
IC2. If the commanding general is loyal, then every loyal lieutenant obeys the
order he sends.

**Impossibility Results**    
Lamport showed that if we use oral messages, no solution will work unless more than two-thirds of the generals are loyal.
For a viable solution we will need 3m + 1 generals where m is the number of traitors.

When we say oral message, we mean the message is not signed by the generals, and sender details cannot be verified .


### **A Solution With Oral Messages**    
Each general is supposed to execute some algorithm that involves sending messages to the other generals, and we assume
that a loyal general correctly executes his algorithm. The definition of an oral message is embodied in the following 
assumptions which we make for the generals' message system:   
A1. Every message that is sent is delivered correctly.   
A2. The receiver of a message knows who sent it.   
A3. The absence of a message can be detected.   

**Algorithm**   
To address the Byzantine Generals Problem, Lamport et al. proposed a recursive algorithm called OM(m), which stands for
Oral Messages with up to m traitors. It is designed to help loyal generals reach agreement even when some of the 
generals may be lying, silent, or sending inconsistent messages.

Recap of the setup
1. There is one commander and n - 1 lieutenants. 
2. The total number of generals n must be at least 3m + 1 to tolerate m traitors. 
3. Communication is synchronous and point-to-point; messages can be delayed or corrupted by traitors.

The goal is for all loyal lieutenants to agree on the same order, and if the commander is loyal, then all loyal lieutenants must follow the commander's order.

![ The Byzantine generals problem algorithm](/assets/distributed%20system/consensus/The%20Byzantine%20Generals%20Problem%20Algorithm.png)

The algorithm is **recursive with depth m**, which is the number of traitors the system wants to tolerate.   
**Base Condition**
If m = 0:
1. Commander sends value to all lieutenants 
2. Lieutenants use the value received 
3. If no value received, use DEFAULT value (e.g., "Retreat").

If m > 0 and you are the Commander:
1. Send value to each lieutenant
2. Wait for recursive process to complete

If m > 0 and you are a Lieutenant:
1. Collect values from recursive process
2. Use majority function on collected values

**Recursive Process OM(m)**   
1. Commander sends value v to all n lieutenants 
2. For each i (1 to n):
- Lieutenant i acts as commander for OM(m-1) 
- Sends the value received to all other lieutenants
3. Each lieutenant collects all n-1 values from other lieutenants 
4. Decision made using majority function on collected values

**Example: OM(1)**   

Suppose n = 4 generals (commander + 3 lieutenants), tolerating m = 1 traitor.
1. Round 1:
- Commander → All Lieutenants: sends value v

2. Round 2:
- Each Lieutenant → All other Lieutenants: forwards received value
- Each Lieutenant: makes decision based on majority of received values

```
                   Round 1: Commander sends value
                            Commander (C)
                               [v=1]
                                 │
                 ┌───────────────┼───────────────┐
                 ▼               ▼               ▼
            Lieutenant 1     Lieutenant 2     Lieutenant 3 ☠
               (L1)            (L2)            (L3)
               [v=1]           [v=1]           [v=1]
                                          (Receives but malicious)

                   Round 2: Lieutenants exchange values
            Lieutenant 1     Lieutenant 2     Lieutenant 3 ☠
               (L1)            (L2)            (L3)
                │               │                │
         ┌──────┴──────┐  ┌─────┴─────┐   ┌──────┴──────┐
         ▼             ▼  ▼           ▼   ▼             ▼
        L2            L3  L1          L3  L1           L2
      [v=1]         [v=1] [v=1]    [v=1] [v=0]     [v=0]
                                         LIES!    LIES!

                   Final Decision Making
        Lieutenant 1         Lieutenant 2         Lieutenant 3 ☠
     Values: [1,1,0]      Values: [1,1,0]      Values: [1,1,0]
     Majority(1,1,0)      Majority(1,1,0)      (Traitor/Ignored)
           ↓                    ↓                    ↓
         [v=1]                [v=1]             [Whatever]


```

### Why does the Algorithm works ?

The recursive spreading and majority-voting help isolate lies. Even if up to m traitors are present, the loyal generals will eventually converge on the same result because:
- The number of loyal messages outweighs the malicious ones due to the 3m + 1 requirement. 
- Traitors cannot forge messages from loyal nodes.
- Recursive reporting allows loyal generals to compare what others received and identify inconsistencies.

This technique ensures **consistency (agreement)** and **integrity (obey the loyal commander's order)** — the two key properties for consensus.

### **A Solution With Signed Messages**
The traitors' ability to lie makes the Byzantine Generals Problem so difficult. The problem becomes easier
to solve if we can restrict that ability. One way to do this is to allow the generals to send unforgeable signed messages.

The algorithm for signed messages relies on the assumption that messages:
- Cannot be forged (e.g., digital signatures), 
- Cannot be altered without detection, and 
- Can be verified by anyone (public-key cryptography model).

This means that Traitors can only choose not to send messages, but cannot alter signed messages

**Algorithm Overview**
0. Round 0: Commander Stage
- Commander signs and sends value (v) to all lieutenants
- Format: <v>C (where C is commander's signature)

1. Round 1: Lieutenant Stage
- Each lieutenant i receives message from commander
- Lieutenant i adds signature and forwards to all other lieutenants
- Format: <v>C,i

2. Round 2: Final Choice
- Each lieutenant collects all messages
- Makes decision based on rules below

**Decision Rules**
1. If commander's order received:
    - Use commander's value

2. If no order received from commander:
    - Use default value "RETREAT"

3. If conflicting orders detected:
    - Commander proven traitor
    - Use default value "RETREAT"


## Learnings

> The Byzantine Generals Problem taught us that agreement under faulty or adversarial conditions requires redundancy, 
> careful protocol design, and sometimes cryptography. These lessons underpin the design of modern consensus
> algorithms and fault-tolerant systems.


1. Byzantine fault tolerant systems need 3f+1 hosts.
2. Authentication (Signatures) changes everything and fairly reduces the complexity of problem.


## References
1. [The Byzantine Generals Problem by Lamport, others in 1982](https://lamport.azurewebsites.net/pubs/byz.pdf)
2. [Notes on Data Base Operating Systems by Jim Gray in 1977 ](https://jimgray.azurewebsites.net/papers/dbos.pdf)