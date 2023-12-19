# The K2 Architecture for Trustworthy Hardware Security Modules

## In general terms, what is the paper about?

K2 is a newly poposed architecture for HSMs (hardware security modules)
It introduces separation boundaries between IO, storage and computation on
secret data. It aims to allow for developing and verifying the software independent
from the hardware development. However it aims to provide correctness guarantees
for the composed system. For the key step of verification a new tool - Chroniton
is proposed to automatically prove the timing properties of a given piece of
software running on a particular hardware implementation. An additional side-effect
is that it eliminates the timing side channels at the cycle-accurate level.

## What problem is being addressed?

Building HSMs that are correct, secure and free of timing side-channel vulnerabilities
is difficult. Moreover, the current implementations of HSMs are susceptible to
bugs. The problem is that correct operation of HSMs involves a proper orchestration
of the software and hardware components, because of this our system can suffer
from both software-based and hardware-based bugs and is also susceptible to
timing side-channels.

## Questions arising after reading the abstract

- What exactly is a hardware security module?
  It is a stanalone hardware component that can be used by applications running
  on the architecture to offload some of the security critical computations onto
  that module. It involves storing the critical state and hosting operations
  managing that state directly on the module. An example could be using HSM to
  store signing keys and signed certificates.

- How does proving the timing properties eliminate the timing side channels?
  The idea behind the K2 architecture is that the outside behaviour of an HSM
  should be indistinguishable from a black box (in the paper: 'oracle') that
  implements the same interface. Because of this an attacker that gains control
  over the systems and can send signals to and read output from the HSM during
  every cycle isn't able to exploit any sidechannels in the execution time of the
  module which could lead to leaking sensitive data (e.g. Spectre / Meltdown like
  attacks.

- What is the cycle-accurate level of those timing side channels?

## In what way is it relevant for the project?

We could incorporate a similar design to ensure isolation of different eBPF
programs running on a single microcontroller. The main selling point of this
design is the removal of side channels. The problem with this approach is that
not all microcontrollers have an MPU and also we would need access to some
language primitives that can be used to clear all of the microarchitectural
state before each of the eBPF programs gets run.

Another interesting idea is the way the programming workflow works in the K2
architecture. The only programmable part of the system is the third part (the
app code). The programmer needs to supply the function which implements the
following signature:
```
void main(char *state, char *cmd, char *out)
```
Note that the programmer doesn't actually modify the storage directly, all they
need to do is to modify the state in memory (as in the things that are pointed to
by `char *state`.

## Key insights

The design of the K2 architecture splits the operation of the system into three
isolated components. Those can be thought of as three separate (logically) CPUs:
the IO CPU, the storage CPU and the app code CPU. In reality, the three machines
are being run on a single RISCV processor which runs them by alternating in a
round-robin fashion. A transition from one stage to the other is done by jumping
out of the U (user) mode back into the M (machine - highest privilege) mode
which then ensures that all of the CPU state is flushed (to prevent leaky side
channels) before the next stage gets run.

## Research methods

- Indicated the problem
- Proposed a solution
- Implemented a prototype and evaluated the key property of the system (verification
  of the constant execution time properties of the user-defined application code
  using Chroniton).
- The correctness of the solution was verified using a range of verification tools.

## Alternative solutions

**Knox** perform verification of HSM hardware by coupling the software and the
hardware parts and performing monolythic reasoning, by contrast K2 reasons about
the two parts separately and then composes the results to derive a proof of the
correctness of the whole system.

## Results of the study

The article doesn't showcase any quantitative data. It would be useful to see
some statistics if there are discrepancies between their proven execution times
and the actual data.

## Discussion points and evaluation

The main workflows supported by K2 are simple applications, where the state and
functionality are quite simple. The main premise of those applications is to
protect the security-related functionality which should be small compared to the
relative size of the rest of the system. Because of the current round-robin design,
the entire state of the HSM needs to be copied over during every stage which
is costly and doesn't scale well for more elaborate workflows.

The approach proposed in the K2 architecture has only been tested on in-order
CPUs, that is because it relies on the formal verification of the execution time,
measured in the number of cycles, therefor o-o-o execution could lead to some
problems with predictability of the system and it could be much harder to prove
the exact execution time.

When it comes to performance, the system works well provided that the request
rate to the HSM is relatively low. In order to achieve higher throughput we
would need to deploy multiple K2 modules.

## Conclusions

K2 architecture is an interesting solution, however the lack of experimental
results and the fact that it might not scale well to out-of-order architectures
may limit its applicability in general purpose hardware.

## General background insights relevant for the project

Side channel vulnerabilities are an important problem indeed. Proper avoidance
of those issues requires a massive performance overhead. In case of K2, a proper
safety w.r.t. timing side channels was achieved by means of temporal partitioning
of the IO, storage and application code functionality and flushing the microarchitecture
state between the different stages of the system operation.
