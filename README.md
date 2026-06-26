<div align="center">

# 64:1 Multiplexer from RTL to GDSII 
### A Silicon Journey: Managing Data Convergence and Routing Congestion

[![OpenLane](https://img.shields.io/badge/OpenLane-1.0.2-blue.svg)](https://github.com/The-OpenROAD-Project/OpenLane)
[![PDK](https://img.shields.io/badge/PDK-Sky130-red.svg)](https://github.com/google/skywater-pdk)
[![Language](https://img.shields.io/badge/Language-Verilog-blueviolet.svg)](#)
[![Status](https://img.shields.io/badge/Status-DRC%20Clean-success.svg)](#)

*Tackling logic folding, select-line fanout, and routing congestion using open-source VLSI tools.*

<img src="mux_ss/tapeout_gds_1.png" alt="3D GDS Layout" width="800px">

---

**[Explore the Visual Journey](#-the-rtl-to-gdsii-visual-journey) • [Key Statistics](#-key-statistics) • [Repository Structure](#-repository-structure) • [How to Reproduce](#-how-to-reproduce)**

</div>

---

## 💡 The Physical Design Challenge

Unlike a decoder that explodes outward, a **64-to-1 Multiplexer** forces a massive amount of data to converge into a single focal point. This creates a unique set of physical design challenges:

> ⚠️ **The Congestion & Fanout Problem**
> 1. **Routing Congestion:** 64 data input wires and 6 select wires must all route toward a highly concentrated logic cone to produce a single output. This heavily congests the metal layers in the center of the standard cell placement.
> 2. **Select-Line Fanout:** The 6 select lines (`sel[5:0]`) must drive the gates of numerous internal 2:1 and 4:1 multiplexer cells. Without proper buffering, these select signals will suffer from high transition times (slew violations) and degrade the timing of the entire chip.

**Our Solutions:**
* **Logic Optimization (abc):** Utilizing combinations of `mux2_2` and `mux4_2` standard cells to build an efficient MUX tree, keeping the total cell count impressively low (61 cells).
* **Automated Buffer Insertion:** Allowing OpenROAD's resizer to step in and drive the high-fanout select lines.
* **Compact Floorplanning:** Shrinking the core area to minimize wire length, resulting in a highly optimized micro-footprint.

---

## 📊 Key Statistics

Based on our automated extraction and signoff reports, the `mux64` design achieves an incredibly efficient footprint and power profile on the Sky130 node.

| Metric | Value |
| :--- | :--- |
| **Total Cell Count** | `61` logic cells |
| **Total Chip Area** | `659.38 µm²` |
| **Total Power** | `28.1 µW` (2.81e-05 W) |
| **Magic DRC** | `0` Violations (Clean) |
| **Netgen LVS** | `Clean` (204 Nets) |
| **Antenna Violations** | `0` Violations |

---

## 🛠️ Tools & Technology Stack

| Stage | Open-Source Tool |
| :--- | :--- |
| **Process Node** | SkyWater 130nm (`sky130A`) |
| **Simulation** | Icarus Verilog (`iverilog`) & GTKWave |
| **Logic Synthesis** | Yosys & abc |
| **Floorplan & Placement** | OpenROAD |
| **Routing** | FastRoute (Global) & TritonRoute (Detailed) |
| **Physical Signoff (DRC/LVS)**| Magic & Netgen |

---

## 📖 The RTL-to-GDSII Visual Journey

### 1️⃣ RTL Design & Functional Verification
The multiplexer was simulated with GTKWave, driving the 64-bit input bus (`in[63:0]`) with known hex patterns and sweeping the `sel[5:0]` lines to ensure the single `out` wire correctly captures the targeted bit.

<p align="center">
  <img src="mux_ss/mux_1.png" width="48%" alt="Testbench Output 1">
  <img src="mux_ss/mux_2.png" width="48%" alt="Testbench Output 2">
</p>

### 2️⃣ Logic Synthesis & Area Profiling
Yosys mapped our behavioral Verilog into Sky130 standard cells. The synthesizer elegantly broke the 64:1 MUX down into a tree of smaller standard MUX cells.

<p align="center">
  <img src="mux_ss/area_report.png" width="70%" alt="Synthesis Area Report">
</p>

### 3️⃣ Floorplanning & Power
Establishing the core area and building the Power Delivery Network (PDN) to ensure the logic tree has a stable voltage supply.

<p align="center">
  <img src="mux_ss/floorplan.png" width="48%" alt="Floorplan">
  <img src="mux_ss/power.png" width="48%" alt="Power Analysis">
</p>

### 4️⃣ Placement & Clock Tree Synthesis (CTS)
Global and detailed placement of the standard cells, followed by buffer insertion to manage the high-fanout select nets.

<p align="center">
  <img src="mux_ss/placement_1.png" width="48%" alt="Placement Phase 1">
  <img src="mux_ss/placement_2.png" width="48%" alt="Placement Phase 2">
</p>
<p align="center">
  <img src="mux_ss/cts_1.png" width="48%" alt="CTS Phase 1">
  <img src="mux_ss/cts_2.png" width="48%" alt="CTS Phase 2">
</p>

### 5️⃣ Routing
FastRoute and TritonRoute connecting the dense logic cone while avoiding Design Rule Checking (DRC) spacing violations.

<p align="center">
  <img src="mux_ss/routing_1.png" width="48%" alt="Routing Phase 1">
  <img src="mux_ss/routing_2.png" width="48%" alt="Routing Phase 2">
</p>

### 6️⃣ Final Signoff & Tapeout Views
After strict geometric and electrical verification, the macro is ready. KLayout reveals the dense pin placement along the boundary.

<p align="center">
  <img src="mux_ss/klayout_gds.png" width="48%" alt="2D KLayout">
  <img src="mux_ss/manufacturability.png" width="48%" alt="Signoff Report">
</p>
<p align="center">
  <img src="mux_ss/tapeout_gds_2.png" width="70%" alt="Final Tapeout Zoom">
</p>

---

## 📂 Repository Structure

```text
├── cts/                 # Clock Tree Synthesis reports and logs
├── final/               # Final merged GDS, LEF, and DEF files
├── floorplan/           # Initial DEF and PDN generation files
├── mux_ss/              # Visuals, screenshots, and reports for documentation
├── placement/           # Global and detailed placement defs
├── routing/             # FastRoute and TritonRoute outputs
├── signoff/             # Magic DRC, Netgen LVS, and Antenna check reports
├── src/                 # Verilog source codes (.v) and testbenches
├── synthesis/           # Yosys synthesis netlists and stat reports
├── README.md            # Project documentation
├── config.json          # OpenLane physical design configuration parameters
└── mux64.gds            # Final extracted GDSII layout
```


# How to Reproduce

To build this mux64 macro on your own machine, ensure you have the OpenLane Docker environment and Sky130 PDK installed.
# Step 1: RTL Simulation

Test the logic before synthesizing:
Bash

Compile the design and testbench
```
iverilog -o tb_mux64 src/mux64.v src/tb_mux64.v

# Execute the simulation
vvp tb_mux64

# View Waveforms
gtkwave mux64.vcd
```
# Step 2: Physical Design Flow (RTL-to-GDSII)

Move this project directory into your OpenLane/designs/ folder.

1. Mount the OpenLane Docker environment
```
make mount
```
2. Run the automated physical design pipeline
```
./flow.tcl -design mux64
```
