---
title: "Understanding distributed time using Logical Clocks"
author: Rishabh Bhatia
categories: [distributed-systems]
tags: distributed systems design paper time clocks
date: 2025-04-26 10:00:00 -0700
---

To build foundation on distributed system, one must understand the distributed time. To embark on the journey of time we 
will start with going over the research paper "Time, Clocks, and the Ordering of Events in a Distributed System" 
published by Lamport in 1978. It is a must read paper who wants to build any distributed systems.

## What is time?
Time is an abstract concept and is the fundamental dimension of our universe. Though time in physical clock that we 
use every day (Example - Apple Watch) is an implementation of humans to bring order to the life on earth. 

The point I am trying to bring is how we implement the concept of time is dependent on the use case we are trying to solve.

## What problem does time solve in physical world?
Time's fundamental use case is to provide basic concept of order in which events occur.

## How we measure time in physical clock?
There are two types of physical clocks commonly used -
1. Quartz based clock - The most common clock on most devices and servers are quartz clock that uses a quartz crystal's 
vibrations to maintain time. Its accurate upto few seconds per month, and need to be adjusted to keep it in sync. 
The oscillations of quartz crystal is dependent on temperature. For example - In hot summer day or in hot server room,
the clock will gets slow and need to be corrected.
2. Atomic Clock - These are expensive clocks and are used to attain high accuracy. For example - Satellites uses atomic clock.
The concept of atomic clock is to use oscillations of atoms to calculate time. It is incredibly accurate with only gaining or
losing 1 second in millions of years.

## Why do we care about time?
We care about ordering of events, we want to know when one event happened before, ofter or concurrently with another.

## What is the problem?
We use quartz based clock for most of the devices, and we understood it either goes slow or fast based on temperature.
This means the clock need to be corrected to stay in sync with time defined by UTC.   
This also means if you have 10 servers with each having their own clock, then each will have their own time. The time in
server 1 will not be same as time in server 3.
   
This is a real problem, as time in different servers cannot be trusted. This means I cannot provide order to the events
happening in distributed system using time in physical clock.


![ Clock Skew Example ](/assets/distributed%20system/time/Distributed%20Time-Clock%20Skew.drawio.png)   

Logs at Server 1   
Server 1 logs Event A sent  at 10:00 local time   
Server 1 logs Event E received  at 10:08 local time    
Logs at Server 2   
Server 2 logs Event B received  at 10:10 local time   
Server 2 logs Event D sent  at 10:12 local time

Correct Total of Events - A -> B, D -> E.    
Order Ignoring clock skew - A, E, B, D   [Incorrect ordering of events, due to clock skew.]    

## Similar problem as Theory Of Relativity.

![ Theory Of Relativity Thought Experiment ](/assets/distributed%20system/time/Distributed%20Time-Theory%20of%20relativity.drawio.png)  
In Space between events two distinct objects, time is relative to the observer's frame of reference.   

Similarly a distributed system consists of a collection of distinct processes which are spatially separated, and which 
communicate  with one another by exchanging messages.   

There are some fascinating parallels between distributed systems and the theory of relativity. 
1. In both, time isn’t absolute: each node in a distributed system keeps its own clock, just as time in relativity depends
on the observer’s frame of reference, making synchronization a complex problem. 
2. Causality and event ordering also share similarities — distributed systems rely on logical clocks, like Lamport clocks, to track the order of events, while relativity shows that different observers might disagree on the sequence of events altogether. 
3. Both fields are limited by the speed of information travel — whether it’s network latency in distributed systems or the speed of light in physics. 
4. The idea of simultaneity is equally slippery: two events that seem simultaneous from one perspective might not be from another. 
5. And finally, consistency models in distributed systems echo relativity’s insight that observers might experience different realities, forcing systems to stitch together a coherent global view. 

These similarities highlight how both distributed systems and relativity wrestle with the deep challenges of managing time, causality, and communication across different points of view.

## The Partial Ordering.

As we can see above using the server's physical clock is not enough to tell the ordering of events. We need to define specification 
that explain the order of events.    
Lamport came up with the concept of "happened before" relation without using physical clock. We can use "->" or "hb" to denote the "happened before" relation.

### Definition of "happened before" relation

![ Theory Of Relativity Thought Experiment ](/assets/distributed%20system/time/Distributed%20Time-Happens%20Before.drawio.png)  

The relation "->" on the set of events of a system is the smallest relation satisfying the following three conditions: 
1. If a and b are events in the same process, and a comes before b, then a -> b. 
2. If a is the sending of a message by one process and b is the receipt of the same message by another process, then a -> b. 
3. If a -> b and b -> c then a -> c. 

Two distinct events a and b are said to be concurrent if a -/-> b and b -/-> a.     
**Note**: concurrent does not mean simultaneous. It just means event a does not have a causality relation with event b.

## Logical Clocks
We can now define clocks in the system. We define a clock C<sub>i</sub> for each process P<sub>i</sub> to be a function which assigns a 
number C<sub>i</sub>(a) to any event a in that process.

Clock Condition: For any events a, b: if a -> b then C(a) < C(b).

Using the happens before relation we can define clock conditions -
C1. If a and b are events in process P<sub>i</sub>, and a comes before b, then C<sub>i</sub>(a) < C<sub>i</sub>(b).
C2. If a is the sending of a message by process P<sub>i</sub> and b is the receipt of that message by process P<sub>i</sub>, then C<sub>i</sub>(a) < C<sub>i</sub>(b).

### Algorithm
Lets implement C1 and C2. 

To satisfy Condition C1, we can simply assign numbers in increasing order to successive events in a process.

Condition C2, will require communication of local time between process.   
Process P<sub>i</sub> will include its local time C<sub>i</sub>(a) while sending a message to P<sub>j</sub>.   
P<sub>j</sub> on receiving the message will assign its time with Max(C<sub>i</sub>(a), C<sub>j</sub>(a)) + 1.  

![ Example of logical clock in action ](/assets/distributed%20system/time/Distributed%20Time-Logical%20Clock.drawio.png)

## Limitation of logical clocks
Since it defines only scalar time, one cannot determine the ordering by simply examining their timestamps. 


## Learning to take away?
1. Events in a distributed system form a partial order.
2. Getting true total order is impractical without introducing serious bottlenecks or single point of failures.
3. Most large scale distributed systems uses the partial order into an arbitrary total order. Making sure the total 
order is deterministic is important for one to have predictable behaviour. Without deterministic order it will become
nightmare to debug and impossible to reason about the behaviour of distributed systems.


## References
1. [Time clocks research paper by Lamport published in 1978.](https://lamport.azurewebsites.net/pubs/time-clocks.pdf)