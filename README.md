# Low-Power-Scan-DFT-Architecture
Pre-synthesis RTL implementation of a dual-tier low-power scan-based DFT architecture combining combinational vector gating and segmented clock deactivation (TSMC 65nm, Synopsys DC &amp; QuestaSim).

##  Overview
During structural scan testing, pseudo-random test patterns induce simultaneous circuit switching activities of 80%–100% compared to 10%–20% in normal functional mode. Automated post-synthesis DFT flows (`insert_dft`) typically bypass functional clock gating during shift mode, causing severe dynamic IR-drop and Vmin timing degradation. 

This repository implements a synchronized, dual-tier low-power scan architecture directly in synthesizable Verilog RTL prior to logic synthesis:
* **Tier 1 (Cell-Level Vector Gating):** Custom `lp_scan_dff` cells clamp functional outputs (`Q_out = ~SE & Q_core`) during shift mode (`SE = 1`) without incurring the sequential register inflation of shadow latches.
* **Tier 2 (System-Level Clock Deactivation):** Scan chains are partitioned into independent segments gated by glitch-free Integrated Clock Gating (ICG) cells (`CKLNQD1HPBWP`), disabling clock distribution to idle scan segments.

##  EDA Toolflow (Two-Pass SAIF Sign-Off)
1. **Pass 1 – Logic Synthesis:** Mapped to **TSMC 65 nm** standard-cell library (`50 MHz`, T = 20.0ns) using **Synopsys Design Compiler**.
2. **Gate-Level Simulation:** Shift-in, capture, and shift-out test vectors executed in **Siemens QuestaSim** to capture exact switching activity (`.saif`).
3. **Pass 2 – Power Sign-Off:** SAIF toggle back-annotation (`read_saif`) in Design Compiler for exact dynamic and leakage power profiling.

## Experimental Results (TSMC 65 nm @ 50 MHz)

### ISCAS'89 `s386` (Pilot Benchmark)
| Architecture | Dynamic Power (uW) | Net Switching Power (uW) | Total Area (um^2) | Dynamic Power Reduction |
| :--- | :---: | :---: | :---: | :---: |
| Automated Tool Baseline (`insert_dft`)| 12.11 | 2.49 | — | Baseline |
| Early Dual-Tier (Shadow Latches) | 13.67 | 1.89 | 357.60 | -12.88% *(Cell inflation)* |
| **Proposed Dual-Tier (Zero-Clamping)** | **7.31** | **1.57** | **290.00** | **39.64% Reduction** |

### ISCAS'89 `s1423` (Scaled Benchmark)
| Architecture | Dynamic Power ($\mu\text{W}$) | Total Cell Area ($\mu\text{m}^2$) | Area Delta | Dynamic Power Reduction |
| :--- | :---: | :---: | :---: | :---: |
| Automated Tool Baseline (`insert_dft`) | 107.63 | 1451.20 | Baseline | Baseline |
| **Proposed 2-Segment Architecture** | **35.14**| **1440.40** | **-0.74%** | **67.35% Reduction** |
| **Proposed 3-Segment Architecture** | **32.70** | **1450.00** | **-0.08%** | **69.62% Reduction** |
| **Proposed 4-Segment Architecture**| **31.76** | **1460.80** | **+0.66%** | **70.49% Reduction** |

> **Key Finding:** Eliminating downstream combinational toggling during shift mode allows the synthesis compiler to downsize sequential flip-flop drive strengths, achieving **silicon area neutrality** (+0.66% on 4-segment) alongside a **70.49% dynamic power reduction**.

## Tech stack / tools

| Category | Details |
|---|---|
| HDL | Verilog |
| Technology node | TSMC 65nm |
| Synthesis | Synopsys Design Compiler |
| Simulation | QuestaSim |

## Repository structure
