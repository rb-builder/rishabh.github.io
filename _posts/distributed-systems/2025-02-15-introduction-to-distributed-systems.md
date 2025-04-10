---
title: "Introduction To Distributed Systems"
categories: [distributed-systems]
tags: [distributed-systems, swe, basics, academic, software-engineering, design, formal, reasoning]
date: 2025-02-15 10:00:00 -0700
---

This blog contains the introduction of distributed systems essentials, a starting point for someone starting up in the 
journey of distributed systems.

## What is distributed systems?
Dictionary definition of "Distributed" is shared or spread out and for "Systems" is a set of things working together 
as parts of a mechanism. A "Distributed System" is how a collection of computers working together to perform the
requested computation.<br>

As a result of distribution we end up with:
1) distance between the computers is non-zero. That implies that transferring bits of information between computers
will result is bounded by physics. As of now it cannot travel faster than speed of light.
2) failures in these computers can happen for uncorrelated* reasons.

<span style="color:grey" > ( For point (2) - We like to think that these independent systems should fail independently,
but it's not always the case. I have seen many instances of correlated failures in systems built/designed with
assumption that it's not possible. ) </span>

All distributed system is about dealing with implications of distribution - distance and having more than one computer.

## Why do we need distributed systems?
In general customers don't really care about if you are using multiple or single computer to work on their requests. 
Customers care about system to be highly available, performant, and scalable in terms of both storage and computation.
These demands are practically impossible to have on single computer. We will need infinite resources to build this 
magic computer that never fails, have infinite storage and computation power. Since we don't have the magic computer or
infinite resources, we need to use multiple computers to distribute the storage and computation needs cheaply.

The main point here is customers want the **experience of interacting with single computer** from your
distributed system.

## What are the things that will not change in customer expectation from software?
Inspired from Jeff Bezos, it's important to work on things that won't change. I can guarantee that even after 10 
years customers will ask for:
1) highly scalable systems
2) high performance and low latency 
3) high availability and fault tolerance

Distributed systems can help you achieve all these things but you still need to deal with physical limitations
( physics :) ).

## How is distributed systems different from single computer applications?
I was advised long ago by a tenured PE that before jumping into distributed programing. Think about how different it is
from single computer applications. This is very important as jumping directly into distributed programming will set you
for failure as everything would feel same in terms of semantics, but you will not develop critical thinking required 
for distributed programming. Here are the few differences I love to talk about:
1) partial failures - Single computer applications does not have those, but its painful reality of distributed. 
Single computer application can either process request or system dies, nothing in between.
2) shared storage - Single computer applications have the complete view of the world is in its storage (one place). When
it comes to distributed system the storage is shared, and need to be collected from different computers. This means it 
would be very hard to know the state of the world at that point of time, as things (states) will change independently 
in different storages.
3) shared clock - Single system have single clock but when it comes to distributed system every computer have their own
clock. This means we need to find a way to coordinate between different systems to agree on one thing. That's
hard and core to many problems.


## How to think about distributed systems?
I have been coding problem-solving and tricky pattern questions since I was 12 year old. This has wired my brain to
think logically and work backward from example in question. I use to starting with solving happy scenarios and then 
think of edge cases.<br>
In the world of distributed systems this is completely inverted. Distributed system is all about failure handling that 
requires invariant based thinking.<br>
Invariant based thinking is about thinking in conditions - 
- System constraints: "What needs to go right?"
- System boundaries: "What is the system allowed to do?"
- System guarantees: "What must the system eventually do?"

<span style="color:grey" > (I don't have a good definition to explain this in simple terms.) </span>

## Conclusion
Distributed systems are fundamental to modern computing, enabling scalability, reliability, and performance that 
single systems cannot achieve. Success in distributed systems requires:
- Understanding physical limitations
- Thinking in terms of invariants
- Handling partial failures
- Managing consistency challenges
- Dealing with time and ordering

## What's next ?
In upcoming blog I will talk about:
- Invariant based thinking.

## References
1. https://scholar.harvard.edu/files/waldo/files/waldo-94.pdf
2. https://www.somethingsimilar.com/2013/01/14/notes-on-distributed-systems-for-young-bloods/
3. https://queue.acm.org/detail.cfm?id=2953944
4. https://lamport.azurewebsites.net/tla/formal-methods-amazon.pdf
