# Minimal Virtual Machines on IoT Microcontrollers: The Case of Berkeley Packet Filters with rBPF

## In general terms, what is the paper about?
## What problem is being addressed?
## Questions arising after reading the abstract
## In what way is it relevant for the project?
## Key insights

Ideas for VM sandboxing:
- host address space is isolated from the sandbox. this is achieved by imposing
  access policy rules. Each memory access that occurrs in the VM is checked
  against those rules (this includes stack reads and writes).

- additionally, security measures on the executed code are in place to ensure that
  the VM doesn't start executing code that is outside of the bound of the supplied
  executable. This is important in preventing side channel vulnerabilities, as
  a malicious actor could try to get the VM to execute a gadget to perform a
  spectre-like side channel attack. This is achieved by guarding the branch
  and jump instructions. The eBPF ISA doesn't allow indirect jumps and the program
  counter register isn't writable (refer to the section in ebpf specification)
  and therefore it is sufficient to perform bounds checks on the direct jumps and
  branches which can be done during the pre-flight verification stage.

**Surprising observation**: WASM interpreter more than doubles the firmware size
with RIOT on ARM Cortex M based microcontroller whereas the rBPF VM only increases
the ROM requirement by roughly 10%.

**Key improvement over JerryScript containers**: rBPF offers memory isolation
guarantees which weren't provided by the previous solution and achieves this at
a much smaller firmware size.

they referenced Tock OS which could be an interesting point of study.

## Research methods

The authors adopt the following workflow:
- describe the experimental setup
- select test workflows which are roughly representative of possible use cases
  (Fletcher32 checksumming algorithm and CoAP response formatter)
- perform measurements w.r.t execution time, code and firmware size and instead
  of considering them in abstract terms, relate them to the prior work, or
  baseline firmware size of the os to put the magnitudes into perspective.

## Alternative solutions

The study refers to prior work on the JerryScript containers and benchmarks the
proposed solution using the same CoAP / SAUL workflow that was investigated in
the previous paper.


## Results of the study
## Discussion points and evaluation

Evaluation points include:
-

Further discussion points:
- add ahead-of-time or JIT capabilities to improve execution speed
- aim to reduce code size by performing instruction compression (as eBPF instructions
  are either 64 or 128 bits long and most of their data is wasted on the
  unused immediate fields.
- extend sandboxing guarantees by limiting the execution time (refer to the 5G
  paper showing how ebpf code can be instrumented)


## Conclusions
