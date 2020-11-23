.. _register-file:

Register File
=============

Source files: :file:`rtl/cv32e40p_register_file_ff.sv` :file:`rtl/cv32e40p_register_file_latch.sv`

CV32E40P has 31 32-bit wide registers which form registers ``x1`` to ``x31``.
Register ``x0`` is statically bound to 0 and can only be read, it does not
contain any sequential logic.

The register file has three read ports and two write ports. Register file reads are performed in the ID stage.
Register file writes are performed in the WB stage.

There are three flavors of register file available.

 * Flip-flop based (:file:`rtl/cv32e40p_register_file_ff.sv`)
 * Latch-based (:file:`rtl/cv32e40p_register_file_latch.sv`)
 * FPGA-optimized (:file:`rtl/cv32e40p_register_file_fpga.sv')

All three flavors have their own benefits and trade-offs.
The latch-based register file is recommended for ASICs, as it results in the smallest footprint for an ASIC implementation.
The flip-flop based register file is recommended for simulation purposes.
The FPGA optimized register file is intended to be used for FPGA synthesis.
While the latch-based register file is recommended for ASICs, the flip-flop based register file is with either synthesis target.
Note the flip-flop based register file is significantly larger than the latch-based register-file for an ASIC implementation.
The FPGA-optimized register file should not be used for ASIC implementations, since this implementation would result in flip-flops lacking asynchronous reset capability.


Flip-Flop-Based Register File
-----------------------------

The flip-flop-based register file uses regular, positive-edge-triggered flip-flops to implement the registers.
This makes it the **first choice when simulating the design using Verilator**.
To select the flip-flop-based register file, make sure to use the source file ``cv32e40p_register_file_ff.sv`` in your project.

Latch-based Register File
-------------------------

The latch-based register file uses level-sensitive latches to implement the registers.

This allows for significant area savings compared to an implementation using regular flip-flops and
thus makes the latch-based register file the **first choice for ASIC implementations**.
Simulation of the latch-based register file is possible using commercial tools.

.. note:: The latch-based register file cannot be simulated using Verilator.

The latch-based register file can also be used for FPGA synthesis, but this is not recommended as FPGAs usually do not well support latches.

To select the latch-based register file, make sure to use the source file ``cv32e40p_register_file_latch.sv`` in your project.
In addition, a technology-specific clock gating cell must be provided to keep the clock inactive when the latches are not written.
This cell must be wrapped in a module called ``cv32e40p_clock_gate``.
For more information regarding the clock gating cell, checkout :ref:`getting-started`.

FPGA-Optimized Register File
-----------------------------
The FPGA register file leverages synchronous-write / asynchronous-read RAM design elements, where available on FPGA targets.

For Xilinx FPGAs, synthesis results in an implementation using RAM32M primitives.
Using this design with a Xilinx Artya7-100 FPGA conserves around 1600 LUTS and 950 flip-flops at the expense of 144 LUTRAMs for the 31-entry register file as compared to the flip-flop based register file.
This makes it the **first choice for FPGA synthesis**.
To select the FPGA-optimized register file, make sure to use the source file ``cv32e40p_register_file_fpga.sv``.

FPU Register File
-----------------

In case the optional FPU is instantiated, the register file is extended
with an additional register bank of 32 registers ``f0``-``f31``. These registers
are stacked on top of the existing register file and can be accessed
concurrently with the limitation that a maximum of three operands per
cycle can be read. Each of the three operands addresses is extended with
an fp_reg_sel signal which is generated in the instruction decoder
when a FP instruction is decoded. This additional signals determines if
the operand is located in the integer or the floating point register
file.

Forwarding paths, and write-back logic are shared for the integer and
floating point operations and are not replicated.
