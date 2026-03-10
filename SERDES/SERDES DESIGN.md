## Project Overview: Open-Source SerDes Development in Sky130

This document outlines the technical exploration, verification strategies, and architectural considerations for implementing a high-speed Serializer/Deserializer (SerDes) using the **Sky130** process node and **OpenLane** EDA flow.

---

## 1. SerDes Architectural Specifications

The following targets have been established for the physical and logical implementation of the SerDes PHY:

|**Metric**|**Target Specification**|
|---|---|
|**Data Rate**|1.25 Gbps (Minimum) / 2.5 Gbps (Target)|
|**Encoding**|8b/10b (Standard 1000BASE-X)|
|**Target BER**|$< 10^{-12}$|
|**Clocking**|Clock and Data Recovery (CDR) via Phase Detector and VCO|
|**I/O Strategy**|Differential Signaling (CML/LVDS-like) via GPIO OVT pads|

### Key Design Challenges

- **Clock Recovery:** Utilizing **CDR** to extract the clock from data transitions, avoiding skew issues inherent in separate clock/data wiring at Gigabit speeds.
    
- **DC-Balance:** Implementing 8b/10b encoding to ensure a 50/50 ratio of 1s and 0s, preventing voltage buildup (DC offset).
    
- **Framing:** Identifying byte boundaries using **Comma characters** (e.g., K28.5) that do not appear in standard data streams.
    

---

## 2. OpenLane Digital Flow & Optimization

The digital logic (PCS layer) is synthesized using the OpenLane "RTL-to-GDSII" flow.

### Synthesis & Timing

- **Toolchain:** Yosys for logic synthesis, OpenROAD for floorplanning, and TritonCTS for Clock Tree Synthesis.
    
- **Timing Optimization:** To achieve a **1.25 GHz** signal (1 ns cycle), the synthesis strategy must be tuned to `DELAY 1` in the `config.json` to prioritize speed over area.
    
- **Logic Depth:** In Sky130, a 1 ns cycle is restricted to approximately **15–20 logic levels**.
    

### Design Flattening & Compatibility

For complex SystemVerilog (SV) hierarchies, the following translation/flattening strategies are utilized:

- **sv2v:** Used to convert advanced SV features (interfaces, modports, packages) into standard Verilog that Yosys can reliably parse.
    
- **Include Management:** Dynamic generation of `-I` flags for `.svh` files is required to maintain single-compilation-unit integrity.
    

---

## 3. Analog & I/O Characterization

Standard Sky130 library cells typically support only tens of MHz. High-speed operation requires the characterization of specialized I/O cells.

### GPIO OVT Pad Testing (`sky130_fd_io__top_gpio_ovtv2`)

- **Transition Time ($t_R/t_F$):** Must be shorter than the SerDes bit-period to avoid signal degradation.
    
- **Panamax Integration:** Utilizing the Panamax wrapper for the GPIO OVT pads to manage power distribution and analog bus (amuxbusA/B) connections.
    
- **Verification:** Eye diagram simulations are required at the `PAD` node to validate signal integrity against Jitter and Slew rate constraints.
    

---

## 4. Verification Methodology

A multi-stage verification approach ensures silicon success:

1. **Functional RTL Simulation:** Using `iverilog` and `Verilator` for initial logic validation.
    
2. **Post-Synthesis Functional Verification:** Running Python-based testbenches against the synthesized gate-level netlist to ensure 8b/10b standard compliance.
    
3. **Post-Layout Verification:** Executing timing-annotated simulations on the final netlist to confirm that physical delays do not break logical functionality.