---
layout: post
title: "Reliability, Scalability and Maintainability"
tags: conceptual
image: 
---

In this post, I am listing down my brief notes on Chapter 1 of 'Designing Data-Intensive Applications' by Martin Kleppmann. 

This chapter is titled 'Reliable, Scalable, and Maintainable Applications'.


## Reliability

- System should work even in the face of adversity
- Systems that anticipate **faults** are called **fault-tolerant**
- **Fault=** a component of the system acting up (ie not as expected)
- **Failure=** entire system fails to work; it is impossible to reduce the chances of system failure (earth annihilation)
- Fault tolerance = making sure faults do not cause failures
- Faults can be caused by hardware or software
- Harware faults: if a HDD is expected to fail once every 10-50 years, and if our system uses 10,000 HDD, one disk will fail every day.
- Add redundancy to tackle hardware faults
- Modern systems rely on so many machines, that we have to make way for entire machines failing. Redundacny is not adequate for this. So we have software techniques to handle such failures
- Software faults: bugs. Bugs are often undeteced till an unexpected set of circumstances trigger them

## Scalability

- It is the system's capability to cope with increased load
- What is load? It is calculated using certain **load parameters**. The selection of load parameters depend  on the architecture of the system
- Scalability is also dependent on how the system performs, calculation of which is again dependent on the system itself.
- There are two ways to deal with increased load: **scale up (**use more powerful machines) or **scale out** (use more machines). Also called, veritcal and horizontal scaling
- Scaling out stateless services is straightforward. Scaling out stateful services is always complex. Until recently, scaling up of data base used to be avoided, unless absolutely necessary
- But with better tools and abstractions, scaling out of databases are now fairly common and becoming the norm even for systems which can do without them.

## Maintainability

- Different people will work with the system; to both maintain the system and adapt it to new circumstances (new uses for example). Goal: build systems which are easier to maintain
- Majority of costs in softwares: maintaining them and not building them
- Three principles to ensure maintainability: **operability, simplicity, evolvability**
- Operability: make it easy for the operations team to run the system smoothly
- Evolveability: Making changes to the system should be easy
- Simplicity: Building systems which are simple:

    - More complex a system, greater the risk of introducing bugs. Why? When a system becomes harder to comprehend or reason about, the hidden assumptions, unexpected interactions becomes overlooked
    - Does that mean we should compromise on functionality to reduce complexity and keep things simple?  No; the goal should be to avoid **accidental complexity**. AKA, complexity that is not tied to the problem we are solving; but that which is arising out of our implementation of the solution
    - One tool to avoid accidental complexity: abstraction. By hiding implementation details, it is easier to reason about. Plus, we can reuse the same code, and avoid uncessary replication of the same implementation. Also, when we improve the abstracted component, it benefits all the systems using it.

