# Enabling BPF Runtime policies for better BPF management

## In general terms, what is the paper about?

With the recent developments of applications of eBPF, operators of distributed
systems have been using eBPF orchestration frameworks. The problem is that
apart from some guarantee of eventual termination of the loaded eBPF programs,
there is a lack of insight into the runtime characteristics of the eBPF
programs while they are being executed.

## What problem does it address?

Problem: operators of the systems instrumented with eBPF programs have no
visibility into whether the performance of their program is affected by the
presence of the additional eBPF code.

Solution proposed: having a runtime estimate that will give some insight into
the impact of introducing eBPF. This additonal information is said to allow for
setting policies that prevent critical paths from being slowed down because of
the additional eBPF code being loaded.

Implementation of the solution:
    - use the verifier to statically track the runtime cost of all possible
      branches
    - dynamically determine the runtime estimates for the helper functions.
    - consider the impact of loop-based helpers on the control flow

The derived estimate is a range that is accurate albeit broad. It can then be
used for making runtime policy decisions.

## Questions arising after reading the abstract

What is special about the **loop-based helpers** that makes them impact the
control flow?

Guess: if there is a helper iterating with a for loop on some data structure,
if at some point the data structure is empty, it will run fast as there is
    nothing to iterate over

## In what way is it relevant for the project?

In case of embedded devices often running under real-time constraints, being
able to enforce that the program that has been instrumented with eBPF code
still meets the performance constraints it very important.

Possible limitations: the solution proposed in the paper relies on the static
analysis perfomed by the verifier.

Given that in our application the verifier might have to be severely constraint
(as we might want to run it on the device - although that is unlikely given
that we are considering the decopuled architecture). Another point is that as
discussed in the meeting on the 28th November, the linux kernel verifier is
able to give us a lot of static information because of how strict it is. For
our application we might go for a much simpler verifier in which case it might
not be possible to statically determine the runtime of all possible branches.

## Key insights:

- the broad range that the estimate covers is due to the IO-bound nature of
  some of the helper functions, the paper claims (page 1, column 2, par 4).
  Additionally, argument dependence and lock contention also play a role in
  this issue. Future work could capture that variability to further refine the
  estimate.

## Research methods

- Problem of the lack of runtime estimates for the eBPF code loaded on the
  critical paths was identified
- A solution involving a combination of static and dynamic analysis was
  proposed to produce a wide range of estimates.
- The solution was evaluated using some example eBPF functions originating from
  the linux kernel source code.

## Alternative solutions

- Dynamic benchmarking using fuzzers or the *bpftool test-run* feature can be
  used to get a holistic estimate of the runtime, however it suffers from
  incompleteness. Such analysis might miss a particularly costly branch, which
  after deployment might lead to unexpected worse-case runtimes.
- (page 3, par 2) Historically static analysis for the runtime estimation
  hasn't been very successful because of the soundness problems. Important
  issue is that function pointers and function arguments are only known at
  runtime and so limit the usefulness of the result of the analysis ->
  **insight** due to the constrained nature of eBPF, static analysis for WCET
  estimation becomes feasible but now poses some new difficulties that are
  specific to eBPF

## Results of the study

Improved verifier design: leveraging the existing CFG analysis that is currently
present in the verifier (it traverses all possible branches to ensure that there
are no safety breaches). We use that to iterate over all possible branches to
generate an estimate of the execution time. The problem here is that the verifier
cannot 'step into' the helper calls and therefore we need to compute estimates
for those externally and then combine the outcomes of the two methods to come up
with the final estimate.

Basically, the execution time of the helpers is determined using an experimental
approach at boot time, it is performed repeatedly under varying workloads and
then best, average and worst case execution times are reported.

For measuring the execution time *bpf_ktime_get_ns* helper is used and its
variance is assumed to be negligible.

Note: some sizable outliers were measured (100-400x slowdown) but those were
discarded as it was assumed that the slowdown was caused by some bursts of
interrupts from things like network packets which aren't specific to eBPF.

## Discussion points and evaluation

Experiments have indicated that for most code samples the actual runtime is
much lower than the WCET, however in case of code that relies on the map-based
helpers whose execution time is bound to the input size, it shows that those
samples are much closer to the worst-case estimate.

Experiments have indicated that for most code samples the actual runtime is
much lower than the WCET, however in case of code that relies on the map-based
helpers whose execution time is bound to the input size, it shows that those
samples are much closer to the worst-case estimate.

Experiments have indicated that for most code samples the actual runtime is
much lower than the WCET, however in case of code that relies on the map-based
helpers whose execution time is bound to the input size, it shows that those
samples are much closer to the worst-case estimate.

The key insight was that although the average case estimates could be used as
they are much tighter and still accurate in most cases, the authors decided to
stick to the worst case estimates as they provide a reasonable  and reliable
upper bound for the execution time which is what we are looking for when we
need to make decisions for the policies that determine if the code gets loaded
into the critical path or not

(page 5, section 5, par 1) Dynamic analysis for the execution time of the helpers
can be not quite reliable in case of complex helper functions with deep call
graphs. In such cases the average case observed from the dynamic measurements
might be much better than the actual worst case.

Two types of helper function behaviours:
1. *argument-dependent* - runtime varies depending on the argument provided.
   In case of those functions, the improved verifier could potentially decrease
   the uncertainty in the runtime estimate if it has access to the argument by
   means of the static verification.
2. *resource-dependent* - e.g. *bpf_map_update_elem* helper relies on getting
   access to a lock and then modifying the data stored in an eBPF map, such
   lock might be contended for by many eBPF programs, thuse the execution time
   of such helper may vary depending on the other programs that are running.
   Latency is caused by both lock contention and cache coherence in this case.
   **Idea** if a map can only be referenced by eBPF programs, the orchestrator
   might be able to know the other programs running at the same time at load time
   and therefore use that information to provide a better estimate by predicting
   the contention for that shared map.

**Vulnerabilities of the verifier** given that the eBPF verifier is currently
being developed, there are still some bugs that cause it to skip branches
sometimes. In such case the WCET analysis wouldn't be accurate and we are at
risk of loading a program which will slow down a given critical path.

## Conclusions

- importance of eBPF runtime estimation
- solution based on hybrid between static analysis and dynamic measurements was
  proposed
- key insights: helper execution time variation can be split into two classes:
  *argument-dependent*, *resource-dependent*.
- both of the above categories can have their estimates made more accurate (tighter
  bounds) by means of using either (in case of arg-dependent) static analysis
  to determine the arguments and adjust the estimation accordingly. or (in case
  of resource-dependent) by monitoring the state of the system by the orchestrator
  and predicting the resource contention at load time.
- the above two directions will be investigated in the future work


## General background insights relevant for the project

1. eBPF differs from the inherently unsafe kernel modules because it has the
static verifier that validates the code before it gets run.
2. How does an eBPF orchestrator work?
   - aim: provide efficient management of all BPF programs in the system
   - workflow: admin loads the eBPF code, it needs to pass the verifier, then
     it is run on a hook point and by means of **maps** it can store some data
     (e.g. results of computations or sensro readings) that can be later
     accessed by the programs running in the userspace.
   - a possible problem with that orchestration is that if we load a slow
     program on a critical path in the system, the verifier doesn't prevent
     that from becoming a bottleneck so we would have to inspect the behaviour
     of the system manually and then unload the problematic program should it
     cause issues with performance.
   - **policies available in an orchestrator**:
     - signature validation
     - restricting users to only be able to hook their eBPF programs to a
       subset of hook points in the system.
     - it is also possible to impose restrictions as to which maps the given
       user can access.
   - **summary**: dual aim of an orchestrator:
     - protecting the system from loading unwanted eBPF code
     - protecting the maps from unwanted reads/writes that can leak sensitive
       system information or cause the eBPF code to malfunction (if it depends
       on some data that is stored in a map).
3. Key point: currently the only runtime guarantee that the verifier gives is
that of the eventual termination of the system.

4. Challenges with static verification of eBPF programs:
   - multiple program paths (one eBPF function may call other ones that may live
     on separate object files.
   - eBPF programs don't capture the whole runtime as they rely on the execution
     time of helper functions which are opaque to the verifier and because of
     this impact the reliability of the results of the static analysis.
   - there are helper functions which impact the control flow e.g. *bpf_loop*
     which repeatedly calls a given eBPF function.
5. Useful context around the runtime terminaton of eBPF programs: *Runtime
   termination of eBPF programs is not trivial. eBPF pro- grams run in the
   kernel, but kernel code is not safe to abruptly terminate, as it may hold
   locks or references to kernel objects that must first be released. Indeed,
   some of the eBPF verifier checks ensure that locks and kernel references are
   released on every code path, assuming that the program will terminate (as is
   also guaranteed by the verifier).* - currently there is work being done
   in order to allow for dynamic termination and graceful freeing of locks
   when doing so.

6. runtime estimates and WCET comparison:
    *Worse Case Execution Time (WCET) is a widely used perfor- mance metric in
    real-time systems [20]. Due to the restrictive pro- gramming environment of
    eBPFs, estimating the runtime of eBPF is very similar to estimating WCET of
    real-time systems. Worse case estimation problem has been broadly dealt
    using two classes of approaches: static analysis, and dynamic measurements.
    The static analysis depends on Control Flow Analysis (CFA) to determine
    loop bounds and recursion depths to provide an upper bound on WCET [21],
    while dynamic measurements aim to provide estimates closer to the hardware
        level by performing end-to-end tests of code sub-components [22]*


