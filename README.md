<div align="center">

# AMBA APB UVM Verification Environment

**A reusable, class-based SystemVerilog UVM testbench for verifying AMBA APB protocol transactions, wait states, and error responses in QuestaSim.**

![SystemVerilog](https://img.shields.io/badge/Language-SystemVerilog-2563EB?style=flat-square)
![UVM](https://img.shields.io/badge/Methodology-UVM_1.2-16A34A?style=flat-square)
![Simulator](https://img.shields.io/badge/Simulator-QuestaSim-EA580C?style=flat-square)
![Pass Rate](https://img.shields.io/badge/Scoreboard-60%2F60_PASS-059669?style=flat-square)
![Coverage](https://img.shields.io/badge/Functional_Coverage-100%25-059669?style=flat-square)

</div>

---

## Overview

This Design Verification (DV) portfolio project implements a complete **Universal Verification Methodology (UVM)** environment to verify an **AMBA APB (Advanced Peripheral Bus)** interface against a behavioral **APB Slave BFM**. 

### Core Verification Capabilities
| Capability | Implementation Details |
| :--- | :--- |
| **Protocol Phases** | Full `IDLE` $\rightarrow$ `SETUP` $\rightarrow$ `ACCESS` handshaking for single and consecutive read/write transfers. |
| **Wait-State Handling** | Dynamic insertion of `0–2` wait cycles (`PREADY = 0`) by the Slave BFM, with `PREADY`-aware driver and monitor sampling. |
| **Error Injection (`PSLVERR`)** | Automated `PSLVERR` assertion and scoreboard verification on out-of-range address accesses (`> 0x3C`). |
| **Self-Checking Scoreboard** | Independent shadow memory model comparing expected vs. actual `PRDATA` and `PSLVERR` on every completed transfer. |
| **Concurrent Assertions (SVA)** | Protocol rules enforcing `PENABLE -> PSEL`, valid `SETUP` $\rightarrow$ `ACCESS` transitions, and signal stability while `PREADY == 0`. |
| **Functional & Cross Coverage** | Covergroups tracking command (`READ`/`WRITE`), valid/invalid address bins, error status, and `Command × Error` crosses. |

---

## Testbench Architecture

```text
+-------------------------------------------------------------------+
|                           APB UVM Test                            |
|  +-------------------------------------------------------------+  |
|  |                       UVM Environment                       |  |
|  |                                                             |  |
|  |   +-------------------------+      +--------------------+   |  |
|  |   |        APB Agent        |      |     Scoreboard     |   |  |
|  |   |  +-------------------+  |      | (Reference Memory) |   |  |
|  |   |  |     Sequencer     |  |      +---------+----------+   |  |
|  |   |  +---------+---------+  |                ^              |  |
|  |   |            |            |                |              |  |
|  |   |            v            |   Analysis     |              |  |
|  |   |  +-------------------+  |     Port       |              |  |
|  |   |  |      Driver       |  |    +-----------+              |  |
|  |   |  +---------+---------+  |    |           |              |  |
|  |   |            |            |    |           v              |  |
|  |   |            |   +--------+----+---+  +----------------+  |  |
|  |   |            |   |     Monitor     |  |   Functional   |  |  |
|  |   |            |   +--------+--------+  |    Coverage    |  |  |
|  |   +------------|------------|--------+  +----------------+  |  |
|  +----------------|------------|-------------------------------+  |
+-------------------|------------|----------------------------------+
                    |            |
                    v            ^ (Sampled on PSEL & PENABLE & PREADY)
              +------------------------+
              |     APB Interface      |  <-- Concurrent SVA Checks
              +-----------+------------+
                          |
                          v
              +------------------------+
              |     APB Slave BFM      |  <-- 0x00–0x3C Valid | >0x3C PSLVERR
              +------------------------+
