---
layout: documentation
title: Understanding gem5 statistics and output
doc: Learning gem5
parent: part1
permalink: /documentation/learning_gem5/part1/gem5_stats/
author: Jason Lowe-Power
---


Understanding gem5 statistics and output
========================================

In addition to any information which your simulation script prints out,
after running gem5, there are three files generated in a directory
called `m5out`:

**config.ini**
:   Contains a list of every SimObject created for the simulation and
    the values for its parameters.

**config.json**
:   The same as config.ini, but in json format.

**stats.txt**
:   A text representation of all of the gem5 statistics registered for
    the simulation.

Where these files are created can be controlled by

config.ini
----------

This file is the definitive version of what was simulated. All of the
parameters for each SimObject that is simulated, whether they were set
in the configuration scripts or the defaults were used, are shown in
this file.

Below is pulled from the config.ini generated when the `simple.py`
configuration file from simple-config-chapter is run.

    [root]
    type=Root
    children=system
    eventq_index=0
    full_system=false
    sim_quantum=0
    time_sync_enable=false
    time_sync_period=100000000000
    time_sync_spin_threshold=100000000

    [system]
    type=System
    children=clk_domain cpu dvfs_handler mem_ctrl membus
    boot_osflags=a
    cache_line_size=64
    clk_domain=system.clk_domain
    default_p_state=UNDEFINED
    eventq_index=0
    exit_on_work_items=false
    init_param=0
    kernel=
    kernel_addr_check=true
    kernel_extras=
    kvm_vm=Null
    load_addr_mask=18446744073709551615
    load_offset=0
    mem_mode=timing

    ...

    [system.membus]
    type=CoherentXBar
    children=snoop_filter
    clk_domain=system.clk_domain
    default_p_state=UNDEFINED
    eventq_index=0
    forward_latency=4
    frontend_latency=3
    p_state_clk_gate_bins=20
    p_state_clk_gate_max=1000000000000
    p_state_clk_gate_min=1000
    point_of_coherency=true
    point_of_unification=true
    power_model=
    response_latency=2
    snoop_filter=system.membus.snoop_filter
    snoop_response_latency=4
    system=system
    use_default_range=false
    width=16
    master=system.cpu.interrupts.pio system.cpu.interrupts.int_slave system.mem_ctrl.port
    slave=system.cpu.icache_port system.cpu.dcache_port system.cpu.interrupts.int_master system.system_port

    [system.membus.snoop_filter]
    type=SnoopFilter
    eventq_index=0
    lookup_latency=1
    max_capacity=8388608
    system=system

Here we see that at the beginning of the description of each SimObject
is first it's name as created in the configuration file surrounded by
square brackets (e.g., `[system.membus]`).

Next, every parameter of the SimObject is shown with it's value,
including parameters not explicitly set in the configuration file. For
instance, the configuration file sets the clock domain to be 1 GHz (1000
ticks in this case). However, it did not set the cache line size (which
is 64 in the `system`) object.

The `config.ini` file is a valuable tool for ensuring that you are
simulating what you think you're simulating. There are many possible
ways to set default values, and to override default values, in gem5. It
is a "best-practice" to always check the `config.ini` as a sanity check
that values set in the configuration file are propagated to the actual
SimObject instantiation.

stats.txt
---------

gem5 has a flexible statistics generating system. gem5 statistics is
covered in some detail on the [gem5 wiki
site](http://www.gem5.org/Statistics). Each instantiation of a SimObject
has it's own statistics. At the end of simulation, or when special
statistic-dumping commands are issued, the current state of the
statistics for all SimObjects is dumped to a file.

First, the statistics file contains general statistics about the
execution:

    ---------- Begin Simulation Statistics ----------
    sim_seconds                                  0.000346                       # Number of seconds simulated
    sim_ticks                                   345518000                       # Number of ticks simulated
    final_tick                                  345518000                       # Number of ticks from beginning of simulation (restored from checkpoints and never reset)
    sim_freq                                 1000000000000                       # Frequency of simulated ticks
    host_inst_rate                                 144400                       # Simulator instruction rate (inst/s)
    host_op_rate                                   260550                       # Simulator op (including micro ops) rate (op/s)
    host_tick_rate                             8718625183                       # Simulator tick rate (ticks/s)
    host_mem_usage                                 778640                       # Number of bytes of host memory used
    host_seconds                                     0.04                       # Real time elapsed on the host
    sim_insts                                        5712                       # Number of instructions simulated
    sim_ops                                         10314                       # Number of ops (including micro ops) simulated

---------- Begin Simulation Statistics ----------sim\_seconds 0.000508
\# Number of seconds simulated sim\_ticks 507841000 \# Number of ticks
simulated final\_tick 507841000 \# Number of ticks from beginning of
simulation (restored from checkpoints and never reset) sim\_freq
1000000000000 \# Frequency of simulated ticks host\_inst\_rate 157744 \#
Simulator instruction rate (inst/s) host\_op\_rate 284736 \# Simulator
op (including micro ops) rate (op/s) host\_tick\_rate 14017997125 \#
Simulator tick rate (ticks/s) host\_mem\_usage 642808 \# Number of bytes
of host memory used host\_seconds 0.04 \# Real time elapsed on the host
sim\_insts 5712 \# Number of instructions simulated sim\_ops 10313 \#
Number of ops (including micro ops) simulated

The statistic dump begins with
`---------- Begin Simulation Statistics ----------`. There may be
multiple of these in a single file if there are multiple statistic dumps
during the gem5 execution. This is common for long running applications,
or when restoring from checkpoints.

Each statistic has a name (first column), a value (second column), and a
description (last column preceded by \#).

Most of the statistics are self explanatory from their descriptions. A
couple of important statistics are `sim_seconds` which is the total
simulated time for the simulation, `sim_insts` which is the number of
instructions committed by the CPU, and `host_inst_rate` which tells you
the performance of gem5.

Next, the SimObjects' statistics are printed. For instance, the memory
controller statistics. This has information like the bytes read by each
component and the average bandwidth used by those components.

    system.clk_domain.voltage_domain.voltage            1                       # Voltage in Volts
    system.clk_domain.clock                          1000                       # Clock period in ticks
    system.mem_ctrl.pwrStateResidencyTicks::UNDEFINED    507841000                       # Cumulative time (in ticks) in various power states
    system.mem_ctrl.bytes_read::cpu.inst            58264                       # Number of bytes read from this memory
    system.mem_ctrl.bytes_read::cpu.data             7167                       # Number of bytes read from this memory
    system.mem_ctrl.bytes_read::total               65431                       # Number of bytes read from this memory
    system.mem_ctrl.bytes_inst_read::cpu.inst        58264                       # Number of instructions bytes read from this memory
    system.mem_ctrl.bytes_inst_read::total          58264                       # Number of instructions bytes read from this memory
    system.mem_ctrl.bytes_written::cpu.data          7160                       # Number of bytes written to this memory
    system.mem_ctrl.bytes_written::total             7160                       # Number of bytes written to this memory
    system.mem_ctrl.num_reads::cpu.inst              7283                       # Number of read requests responded to by this memory
    system.mem_ctrl.num_reads::cpu.data              1084                       # Number of read requests responded to by this memory
    system.mem_ctrl.num_reads::total                 8367                       # Number of read requests responded to by this memory
    system.mem_ctrl.num_writes::cpu.data              941                       # Number of write requests responded to by this memory
    system.mem_ctrl.num_writes::total                 941                       # Number of write requests responded to by this memory
    system.mem_ctrl.bw_read::cpu.inst           114728823                       # Total read bandwidth from this memory (bytes/s)
    system.mem_ctrl.bw_read::cpu.data            14112685                       # Total read bandwidth from this memory (bytes/s)
    system.mem_ctrl.bw_read::total              128841507                       # Total read bandwidth from this memory (bytes/s)
    system.mem_ctrl.bw_inst_read::cpu.inst      114728823                       # Instruction read bandwidth from this memory (bytes/s)
    system.mem_ctrl.bw_inst_read::total         114728823                       # Instruction read bandwidth from this memory (bytes/s)
    system.mem_ctrl.bw_write::cpu.data           14098901                       # Write bandwidth from this memory (bytes/s)
    system.mem_ctrl.bw_write::total              14098901                       # Write bandwidth from this memory (bytes/s)
    system.mem_ctrl.bw_total::cpu.inst          114728823                       # Total bandwidth to/from this memory (bytes/s)
    system.mem_ctrl.bw_total::cpu.data           28211586                       # Total bandwidth to/from this memory (bytes/s)
    system.mem_ctrl.bw_total::total             142940409                       # Total bandwidth to/from this memory (bytes/s)

Later in the file is the CPU statistics, which contains information on
the number of syscalls, the number of branches, total committed
instructions, etc.

    system.cpu.dtb.walker.pwrStateResidencyTicks::UNDEFINED    507841000                       # Cumulative time (in ticks) in various power states
    system.cpu.dtb.rdAccesses                        1084                       # TLB accesses on read requests
    system.cpu.dtb.wrAccesses                         941                       # TLB accesses on write requests
    system.cpu.dtb.rdMisses                             9                       # TLB misses on read requests
    system.cpu.dtb.wrMisses                             7                       # TLB misses on write requests
    system.cpu.apic_clk_domain.clock                16000                       # Clock period in ticks
    system.cpu.interrupts.pwrStateResidencyTicks::UNDEFINED    507841000                       # Cumulative time (in ticks) in various power states
    system.cpu.itb.walker.pwrStateResidencyTicks::UNDEFINED    507841000                       # Cumulative time (in ticks) in various power states
    system.cpu.itb.rdAccesses                           0                       # TLB accesses on read requests
    system.cpu.itb.wrAccesses                        7284                       # TLB accesses on write requests
    system.cpu.itb.rdMisses                             0                       # TLB misses on read requests
    system.cpu.itb.wrMisses                            31                       # TLB misses on write requests
    system.cpu.workload.numSyscalls                    11                       # Number of system calls
    system.cpu.pwrStateResidencyTicks::ON       507841000                       # Cumulative time (in ticks) in various power states
    system.cpu.numCycles                           507841                       # number of cpu cycles simulated
    system.cpu.numWorkItemsStarted                      0                       # number of work items this cpu started
    system.cpu.numWorkItemsCompleted                    0                       # number of work items this cpu completed
    system.cpu.committedInsts                        5712                       # Number of instructions committed
    system.cpu.committedOps                         10313                       # Number of ops (including micro ops) committed
    system.cpu.num_int_alu_accesses                 10204                       # Number of integer alu accesses
    system.cpu.num_fp_alu_accesses                      0                       # Number of float alu accesses
    system.cpu.num_vec_alu_accesses                     0                       # Number of vector alu accesses
    system.cpu.num_func_calls                         221                       # number of times a function call or return occured
    system.cpu.num_conditional_control_insts          986                       # number of instructions that are conditional controls
    system.cpu.num_int_insts                        10204                       # number of integer instructions
    system.cpu.num_fp_insts                             0                       # number of float instructions
    system.cpu.num_vec_insts                            0                       # number of vector instructions
    system.cpu.num_int_register_reads               19293                       # number of times the integer registers were read
    system.cpu.num_int_register_writes               7976                       # number of times the integer registers were written
    system.cpu.num_fp_register_reads                    0                       # number of times the floating registers were read
    system.cpu.num_fp_register_writes                   0                       # number of times the floating registers were written
    system.cpu.num_vec_register_reads                   0                       # number of times the vector registers were read
    system.cpu.num_vec_register_writes                  0                       # number of times the vector registers were written
    system.cpu.num_cc_register_reads                 7020                       # number of times the CC registers were read
    system.cpu.num_cc_register_writes                3825                       # number of times the CC registers were written
    system.cpu.num_mem_refs                          2025                       # number of memory refs
    system.cpu.num_load_insts                        1084                       # Number of load instructions
    system.cpu.num_store_insts                        941                       # Number of store instructions
    system.cpu.num_idle_cycles                          0                       # Number of idle cycles
    system.cpu.num_busy_cycles                     507841                       # Number of busy cycles
    system.cpu.not_idle_fraction                        1                       # Percentage of non-idle cycles
    system.cpu.idle_fraction                            0                       # Percentage of idle cycles
    system.cpu.Branches                              1306                       # Number of branches fetched
