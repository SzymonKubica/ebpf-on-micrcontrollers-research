# CHERIoT: Complete Memory Safety for Embedded Devices

## What problem is being addressed?

The paper aims to address the issue of memory safety in embedded devices. With
the recent growth in the usage of those devices (especially in the IoT applications)
the attack surface has grown substantially. There are two main factors contributing
to the complexity of the problem. Firstly, such devices often operate with
constrained resources, which limits the viability of the usual memory safety
solutions one might find in a large OS like Windows or Linux. Furthermore, those
systems often need to meet particular real-time requirements, therefore they are
running Real-Time Operating Systems (RTOS) with complicated multitasking workflows.

The main issue is that the solutions incorporated into the desktop operating systems
don't translate well into an embedded environment. The authors of the paper claim
that all of the existing solutions violate at least one of the three key requirements:
real-time, code size, power and compatibility.

## What are the main claimed contributions?

(1)
An embedded-systems architecture designed to support a
(co-designed) RTOS offering complete and deterministic com-
partmentalized memory safety.

An important point here is that the architecture is claimed to ensure **deterministic**
memory safety, which is much better compared to the probabilisitic solutions.

The idea of compartmentalization is to allow runnign multiple mutually distrusted
programs owned by different parties on a single embedded device without having
to introduce physical separation by deploying one device per party.

(2) Architectural extensions and hardware acceleration for tem-
poral memory safety of cross-compartment references and
the RTOS’s shared heap allocator.

In this contribution the main idea is that they build on top of the existing
CHERI (Capability Hardware Enhanced RISC Instructions) to provide a custom solution
which can be used in an embedded IoT setting.

(3) Performance evaluation of two embedded cores with differ-
ent design tradeoffs.

This contribution can be analysed further in the evidence section later.

(4) Evaluation of area, power and critical-path of hardware as-
sists for a production-quality core.

This contribution will be useful for analysing the adequacy of the experiments
to support the claimed contributions of the paper.

## What evidence is offered to support the claims?

## Does the experimental work test the claims adequately?

Important Quotes:

*"Provide abstractions that can be surfaced directly in C-like languages, for
example protecting objects, not pages"*

-> Important, the aim is to provide high level language abstractions on top
of the proposed architecture to make it more applicable to the real world
applications.

*"Avoid requiring any structures in hardware that would in- troduce
nondeterministic latency"*

-> This is due to the real-time requirements of the embedded devices, for
some applications we need to ensure that the time it takes to execute a
given instruction is deterministic and doesn't vary because of complicated
architectural decisions.

*"Avoid requiring any structures in hardware that would sig- nificantly increase
the area or power consumption"*

-> Again, one of the characteristic features of embedded devices is that
that their resources are limited, therefore a significant overhead
introduced by the solution would impact the applicability of the design.
(Examples of power-intensive solutions are associative lookups in an MPU
or TLB)

*"A real-time system is one in which the latency of operations is bounded and
can be reasoned about"*

-> Important definition of real-timeness. It doesn't directly imply that the
latency of operations must be low, it says that whatever we do needs to be
of fixed time and the time it takes to execute a given instruction shouldn't
depend on the state of the cache or TLB. CHERIoT ensures that none of the
hardware operations have nondeterministic latency, however some operations
might have a small variation in the cycle time.

*"In today’s systems, this compartmentalization is often achieved by providing
multiple microcontrollers, each with separate SRAM, which both adds to the cost
and limits flexibility"*

-> Indicates the current way of ensuring compartmentlization right now, the
main limitation is the cost of deploying multiple devices and the lack of
flexibility i.e. adding a new compartment requires modifying the physical
setup of the deployed system (adding a new microcontroller).

*"In an embedded system, partic- ularly one without atomic operations (optional
in RISC-V), software often needs to disable interrupts for short periods"*

-> Important motivation for disabling interrupts, useful context for fyp.

*"CHERIoT addresses these inefficiencies, arriving at an encoding (fig. 1) that
optimizes for the typical embedded programming model."*

-> CHERIoT ensures that the memory overhead of having capabilities ingrained
into pointers is minimised.

*"Instead, our CPU pipeline offers hardware-assisted temporal memory safety
with a stronger security model and without need of an MMU."*

-> Understand what is actually meant by the temporal memory safety.

Overview of compartments

-> compartment = code + data (some of which is private)
-> mutual distrust
-> compartments may attack each other
-> compartments interface with each other by providing exports and consuming imports
-> real world examples include bugs, low quality drivers or even software supply
   chain attacks

Trusted Computing Base (TCB) -> important

This introduces a precondition that the owners of separate compartments need to
trust the provider of the compartmentalized system that whatever they are doing
ensures proper separation and is bug-free.

In case of CHERIoT, the TCB includes: microarchitecture, hardware offloads and
software. The microarchitecture enforces local invariants (guarantees true on
the level of a single instruction), whereas the software ensures that global
invariants are satisfied.

## Design

Because of the resource constraints of the embedded devices, the authors decided
to remove the unused expressiveness from the CHERI architecture before building
CHERIoT. Things removed:

- ```cinvoke``` instruction and permission
- capabilities may not simultaneously permit execution and stores (i.e. you
  can't modify the code while executing it.

Bounds encoding -> the authors propose a modified design which sacrifices some
of the compatibility in exchange for increased precision and decreased
complexity

## Tricky terms

**sealed** and **unsealed** entries

*"More generally, CHERI provides a ‘sealing’ mechanism for constructing opaque
capabilities, which may later be ‘unsealed’; again, both actions are authorized
by capabilities"*

-> not quite sure what is meant by sealing and unsealing

**jump-and-link** instruction -> what is this?

**Memory fragmentation** how does one calculate it?

## Performance evaluation



