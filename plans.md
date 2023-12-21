# Project Plan

## Problem description:


## Already done
- initial literature overview
- target [microcontroller](./microcontrollers.md) chosen
- rbpf was forked and modified to run on esp32
- initial [benchmarks](./explorations/fletcher32-bench.md) conducted on esp32
- dev environment set up for stm32

## Next Steps:
1. Complete the background research
2. Get hands on experience with eBPF (use the [book](https://cilium.isovalent.com/hubfs/Learning-eBPF%20-%20Full%20book.pdf?utm_campaign=2023-03-E%3Abook-Learning-eBPF-Full-book&utm_medium=email&_hsmi=251104540&_hsenc=p2ANqtz-8s6xYkhubDq-wC_JgJBZohMDshLTqvM7TIoQ64J6xe7a5RCKYQLSdGGO8J2FS8Bx-PwGP7z7ufwFq83_goyHgJzLZmNQ&utm_content=251104540&utm_source=hs_automation))
   (loading code into the VM is harder than expected).
3. Get benchmarks running on stm32:
   - stm32 is no-std so rbpf isn't as easy to port, possible solutons:
       - further modifications to the fork to build a separate no-std version
       - pivot to uBPF
4. Streamline the write-compile-load workflow for quicker iterations.
   - Currently the eBPF code needs to be compiled and the bytes need to be
     hard-coded into the flashed executable - not ideal
   - possilbe solutions:
     - in case of esp32: set up http server and accept the eBPF bytes through it
     - investigate the NVS or SPIFFS / littlefs file systems to load files onto
       the microcontroller so that the program can load them.
5. Investigate **prevail** verifier.
6. Further modify rBPF to allow for:
   - ARM jit compilation (nice to have on stm32)
   - XTENSA jit (possibly tricky)



