# Model Design

[Main Page](../README.md)

## Verification of the eBPF bytecode

Should we verify at all?
What are the verification guarantees?

Where does the verification happen?
- on the board
- on another machine

## JIT compilation support

## Isolation support

- Spatial?
- Temporal?

# Proposed Project Roadmap

As suggested in the decoupled eBPF paper, we should be aiming at a decoupled architecture
with verification happening on a vm running on a powerful computer.
In the ideal case, the eBPF bytecode should also be jit compiled into the desired
architecture at that step. This approach however comes with the following difficulties:

- need for relocation tables for both helper functions and maps
- need to ensure that the output of the jit bytecode is the same as if it was jitted
  on the target microcontroller

Proposed roadmap

1. Build a system where BPF bytecode is sent to a verifier present on a vm running
   on some PC. In conjunction, build infrastructure on the microcontrollers to
   load that verified and signed bytecode and then run it on:
   - first a userspace vm as it is done in case of rbpf
   - then either: add first class support for the microkernels (similar to what femto-containers
     and rbpf did in case of riot)
   - add support for jitting on esp32 (directly on the microcontroller)
   - once the above jitting works, try to decouple it so that it happens together with the
     verification stage.

   - Important concern about decoupling the JIT and the verification according to the
     in decoupled paper.
     *Currently verification and JIT are tightly coupled inside the
     kernel. For example, the maximum allowed number of tail calls
     in a BPF program is 33 [5], but we found that this check
     occurs inside the JIT code, rather than the verifier. The
     verifier and JIT work together to ensure BPF program safety,
     so it is nontrivial to separate them. We decided the solution
     to this is to decouple the JIT along with the verifier. 1*

2. MVP:
  - Infra for the verifier service with signatures (as described in decoupled ebpf paper)
  - Infra for loading the verified bytecode together with the signature into the microcontroller
    directly over the network (and have it run in the rbpf/ubpf vm)
  - Performance evaluation of the solution


3. Current limiations of the existing work:

- rBPF doesn't implement tail calls (one eBPF program calling the other)
- jit compilation for other architectures (ubpf does it for ARM, in case of rBPF ARM is missing and we would probably need to add Xtensa as well).
- support for arrays and BPF maps



