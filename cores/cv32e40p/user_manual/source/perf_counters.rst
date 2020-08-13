.. _performance-counters:

Performance Counters
====================

CV32E40P implements performance counters according to the RISC-V Privileged Specification, version 1.11 (see Hardware Performance Monitor, Section 3.1.11).
The performance counters are placed inside the Control and Status Registers (CSRs) and can be accessed with the ``CSRRW(I)`` and ``CSRRS/C(I)`` instructions.

CV32E40P implements the clock cycle counter ``mcycle(h)``, the retired instruction counter ``minstret(h)``, as well as the parameterizable number of event counters ``mhpmcounter3(h)`` - ``mhpmcounter31(h)`` and the corresponding event selector CSRs ``mhpmevent3`` - ``mhpmevent31``, and the ``mcountinhibit`` CSR to individually enable/disable the counters.

Unimplemented counters always read 0.

.. _event_selector:

Event Selector
--------------

The following events can be monitored using the performance counters of CV32E40P.


+-------------+-----------------+-------------------------------------------+
| Bit #       | Event Name      |                                           |
+=============+=================+===========================================+
| 0           | CYCLES          | Number of cycles                          |
+-------------+-----------------+-------------------------------------------+
| 1           | INSTR           | Number of instructions retired            |
+-------------+-----------------+-------------------------------------------+
| 2           | LD_STALL        | Number of load use hazards                |
+-------------+-----------------+-------------------------------------------+
| 3           | JMP_STALL       | Number of jump register hazards           |
+-------------+-----------------+-------------------------------------------+
| 4           | IMISS           | Cycles waiting for instruction fethces,   |
|             |                 | excluding jumps and branches              |
+-------------+-----------------+-------------------------------------------+
| 5           | LD              | Number of loads                           |
+-------------+-----------------+-------------------------------------------+
| 6           | ST              | Number of stores                          |
+-------------+-----------------+-------------------------------------------+
| 7           | JUMP            | Number of jumps (unconditional)           |
+-------------+-----------------+-------------------------------------------+
| 8           | BRANCH          | Number of branches (conditional)          |
+-------------+-----------------+-------------------------------------------+
| 9           | BRANCH_TAKEN    | Number of branches taken (conditional)    |
+-------------+-----------------+-------------------------------------------+
| 10          | COMP_INSTR      | Number of compressed instructions retired |
+-------------+-----------------+-------------------------------------------+
| 11          | PIPE_STALL      | Cycles from stalled pipeline              |
+-------------+-----------------+-------------------------------------------+
| 12          | APU_TYPE        | Numbe of type conflicts on APU/FP         |
+-------------+-----------------+-------------------------------------------+
| 13          | APU_CONT        | Number of contentions on APU/FP           |
+-------------+-----------------+-------------------------------------------+
| 14          | APU_DEP         | Number of dependency stall on APU/FP      |
+-------------+-----------------+-------------------------------------------+
| 15          | APU_WB          | Number of write backs on APUB/FP          |
+-------------+-----------------+-------------------------------------------+

The event selector CSRs ``mhpmevent3`` - ``mhpmevent31`` define which of these events are counted by the event counters ``mhpmcounter3(h)`` - ``mhpmcounter31(h)``.
If a specific bit in an event selector CSR is set to 1, this means that events with this ID are being counted by the counter associated with that selector CSR.
If an event selector CSR is 0, this means that the corresponding counter is not counting any event.

Controlling the counters from software
--------------------------------------

By default, all available counters are disabled after reset in order to provide the lowest power consumption.

They can be individually enabled/disabled by overwriting the corresponding bit in the ``mcountinhibit`` CSR at address ``0x320`` as described in the RISC-V Privileged Specification, version 1.11 (see Machine Counter-Inhibit CSR, Section 3.1.13).
In particular, to enable/disable ``mcycle(h)``, bit 0 must be written. For ``minstret(h)``, it is bit 2. For event counter ``mhpmcounterX(h)``, it is bit X.

The lower 32 bits of all counters can be accessed through the base register, whereas the upper 32 bits are accessed through the ``h``-register.
Reads of all these registers are non-destructive.

Parametrization at synthesis time
---------------------------------

The ``mcycle(h)`` and ``minstret(h)`` counters are always available and 64 bit wide.
The optional performance counters ``mhpmcounter3(h)`` - ``mhpmcounter31(h)`` are parametrizable in width.
All counters are 64 bit wide.

The number of event counters is determined by the parameter ``NUM_MHPMCOUNTERS`` with a range from 0 to 29 (default value of 1).
Their width is determined by the parameter ``WIDTH_MHPMCOUNTERS``.

An increment of 1 to the ``NUM_MHPCOUNTERS`` results in the addition of the following:

[//]: # (should it not be 16 flops for ``mhpmeventX`` ? )
   - ``WIDTH_MHPMCOUNTERS`` flops for ``mhpmcounterX``
   - 15 flops for ``mhpmeventX``
   -  1 flop  for ``mcountinhibit[X]``
   - Adder and event enablement logic

FPGA Targets
------------
For FPGA targets the performance counters constitute a particularly large structure.
For Xilinx FPGA devices featuring the DSP48E1 DSP slice or similar, counter logic can be absorbed into the DSP slice for widths up to 48 bits.
For 32 bit counters only, the corresponding flip-flops can be incorporated into the DSP’s output pipeline register, additionally resulting in a reduction of the number of flip-flops.
Implementing the maximum 29 event counters 32 bit wide using DSP slices results in a reduction of LUT utilization by 11% and FPGA utilization by 24% with respect to the corresponding non-DSP implementation.
This comes at the expense of 1 DSP slice per counter.
In order to infer DSP slices for performance counters, define the preprocessor variable ``TARGET_XILINX``.

Next  Previous
