# Scripting Over-The-Air: Towards Containers on Low-end Devices in the Internet of Things

## In general terms, what is the paper about?

Implementing runtime containers that are updatable over-the-air on embedded
devices.

## What problem is being addressed?

The problem is that IoT devices sometimes need to contain logic that isn't
known before they get deployed (e.g. sensor calibration settings). In such case
a need for runtime upgradability emerges. The authors propose over-the-air scripting
as a solution for this issue. The deployed devices will be running VMs capable
of executing the scripting logic that gets loaded over the network.

## Questions arising after reading the abstract

Heterogenous IoT devices?

## In what way is it relevant for the project?

Very relevant -> a precursor study for Femto-Containers

## Key insights

Two reasons motivating over-the-air scripting:

- some part of the logic (e.g. pre-processing logic or some sensitive data) might
  need to be transfered after deployment because of privacy or performance reasons.
    - privacy: the data might be sensitive and thus cannot be hard coded in the device's
    source code.
- another reason is adjusting some parameters of the logic (sensitivity of sensor,
  frequency of taking measurements)

## Research methods

Built the system, measured its ROM and RAM requirements and compared to the
ROM and RAM sizes of the off-the-shelf devices as well as alternative solutions.

## Alternative solutions

Perform firmware updates over the network (either full image replacement or
partial firmware updates with dynamic linking or diff-based updates).

## Results of the study

Key improvements include a reduction in required bytes over-the-air to transfer
in order to send an update (scripts aren't that big)

## Discussion points and evaluation

Evaluated w.r.t
- RAM and ROM requirements
- compatibility with RIOT supported boards

## Conclusions

Future work needs to provide more sandboxing guarantees and explore being able
to host multiple containers on a single microcontroller.

## General background insights relevant for the project

Very useful to get additional motivation for hosting lightweight containers
on microcontrollers.
