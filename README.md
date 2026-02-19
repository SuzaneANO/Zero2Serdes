# Zero2Serdes
Legendary Quest


#### Origin Story 

https://www.linkedin.com/feed/update/urn:li:activity:7428935929311965184/?dashCommentUrn=urn%3Ali%3Afsd_comment%3A(7429200218283479040%2Curn%3Ali%3Aactivity%3A7428935929311965184)&dashReplyUrn=urn%3Ali%3Afsd_comment%3A(7429422102346178561%2Curn%3Ali%3Aactivity%3A7428935929311965184)

#### Presentation of the Tools :
https://openlane2.readthedocs.io/en/latest/getting_started/newcomers/index.html


#### Serdes Definition :
https://en.wikipedia.org/wiki/SerDes
##### Questions : 
1/ How can you describe the recovery of a clock in a Serdes parallel interface ? 

2/ What's a clock [jitter](https://en.wikipedia.org/wiki/Jitter "Jitter") , Framing ,DC- balence, what do the numbers mean in 8b 10b 64b 66b ... ? What is a transition ? 

##### Answers :
###### 1. Clock Recovery in a SerDes

In high-speed serial links, we don’t send a separate "clock wire" because at Gigabit speeds, the clock and data would arrive at different times (a problem called **Skew**).

Instead, we use **Clock and Data Recovery (CDR)**.

- **The Concept:** The receiver looks at the incoming data and "extracts" the clock from the **transitions** (the jumps from 0 to 1 or 1 to 0).
    
- **How it works:** A local oscillator (VCO) at the receiver tries to guess the speed. A **Phase Detector** compares the timing of the incoming data edges to the local clock. If the data arrives "early," the clock speed is increased; if "late," it's slowed down.
    
    +1
    
- **The Result:** The receiver’s clock eventually "locks" onto the transmitter’s frequency, allowing it to sample the data exactly in the middle of the "eye."
    

---

###### 2. The Jargon: Jitter, DC-Balance, and Framing

 **Clock Jitter**

Think of jitter as the "shaky hands" of a clock.

- **Definition:** It is the short-term variation of a signal's edges from their ideal positions in time.
    
- **The Danger:** If jitter is too high, the "shaky" clock might sample the data at the exact moment it's switching between 0 and 1, leading to bit errors.

 **DC-Balance & Running Disparity**

Digital signals are electrical. If you send too many `1`s in a row (e.g., `11111111`), the voltage builds up on the line like a battery charging.

- **The Problem:** This "DC offset" can distort the signal and make it impossible for some receivers (like those using transformers or capacitors) to read it.
    
- **The Solution:** We use encoding to ensure that over time, the number of `0`s and `1`s is roughly equal (50/50). This is **DC-Balance**.
    

 **Framing**

In a serial stream, it's just a giant "snake" of bits. How do you know where Byte A ends and Byte B begins?

- **Framing** uses special bit patterns (often called **Comma characters**) that never appear in normal data. When the receiver sees a Comma, it says "Aha! This is the start of a new frame," and aligns its parallel output accordingly.
    

---

######  3. Decoding the Numbers: 8b/10b vs. 64b/66b

These numbers describe the **Overhead** of the encoding scheme.

|**Scheme**|**Meaning**|**Efficiency**|**Why use it?**|
|---|---|---|---|
|**8b/10b**|Takes **8 bits** of data and turns them into **10 bits**.|80%|**Very high transition density.** Great for slower links (like 1G Ethernet) because it makes clock recovery very easy.|
|**64b/66b**|Takes **64 bits** of data and adds **2 bits** of preamble.|97%|**Low overhead.** Used in 10G+ Ethernet. It uses "scrambling" to get DC balance instead of strict mapping.|

- **The Trade-off:** 8b/10b is "expensive" (you waste 20% of your bandwidth), but it is robust. 64b/66b is "cheap" but requires more complex math to keep the clock locked.
    

---

###### 4. What is a "Transition"?

A **Transition** is simply the event where a signal changes state (0 → 1 or 1 → 0).

- **Why it matters for you:** No transitions = No clock recovery. If you send `00000000...` forever, the receiver’s CDR "loses its way" because there are no edges to look at.
    
- **The Goal:** Encoders like 8b/10b guarantee that you will see a transition at least once every 5 bits, keeping the system "in sync."

#### 8b/10b Logic Design exploration
https://github.com/ghaithalhafi/openlane-eth-phy/tree/main/runs/RUN_2026-02-12_18-41-11

##### Questions :  
1/ How OpenLane is able to synthesize all of this ?

2/ What are the improvement margins the OpenLane code allows ? How to implement modifications 

##### Answers :  

###### 1. How OpenLane Synthesizes this Code

OpenLane follows a modular "RTL-to-GDSII" flow. For the files you uploaded (`phy_top_1g.v` and `pcs_tx_8b10b.v`), the process looks like this:

- **Logic Synthesis (Yosys):** OpenLane uses **Yosys** to translate your high-level Verilog logic into a "gate-level netlist". It maps your `case` statements and state machines in `pcs_tx_8b10b.v` to specific "standard cells" (AND gates, Flip-Flops) from the **Sky130 PDK**.
    
- **Technology Mapping (abc):** A sub-tool called `abc` optimizes this netlist for either the smallest area or the fastest timing based on your configuration.
    
- **Floorplanning & Placement (OpenROAD):** OpenLane determines where on the silicon chip these gates should sit. It creates a "Power Distribution Network" (PDN) to make sure every gate gets electricity.
    
- **Clock Tree Synthesis (TritonCTS):** Since Ethernet requires precise timing, this step is critical. It builds a network of buffers to ensure the `clk` signal reaches every flip-flop at exactly the same time.
    
- **Routing (TritonRoute):** Finally, it draws the tiny metal wires that connect all the gates together.
    

###### 2. Improvement Margins & Modifications

The current code is a "minimalist" implementation, which leaves significant "improvement margins" in three areas: **PPA** (Power, Performance, Area), **Standard Compliance**, and **Physical Reliability**.

Improvement Margins

- **Timing Slack (Performance):** If you run the flow and see a positive "Slack" in the reports, you have a margin to increase the clock speed. You can push for a 125MHz clock (standard for Gigabit Ethernet) by changing `SYNTH_STRATEGY` to focus on `DELAY` instead of `AREA`.
    
- **Core Utilization (Area):** If the design is sparse, you can increase `FP_CORE_UTIL` (e.g., from 5% to 40%) to make the chip smaller and cheaper to produce.
    
- **Logic Completeness:** The current RTL is very "light." A professional PHY would need:
    
    - **Auto-Negotiation:** Logic to detect if the other end of the cable is 10, 100, or 1000 Mbps.
        
    - **Error Reporting:** CRC checks and link-status registers.
        

How to Implement Modifications

To modify the project, you don't just change the Verilog; you change the **OpenLane Configuration**:

1. **Modify the RTL:** Open `pcs_tx_8b10b.v` and add your logic (e.g., a "K-symbol" detector for special Ethernet control characters).
    
2. **Edit `config.json` / `config.tcl`:** This is where you control the "margins."
    
    - To improve timing: Set `"SYNTH_STRATEGY": "DELAY 1"`.
        
    - To fix hold violations: Adjust `PL_RESIZER_HOLD_SLACK_MARGIN`.
        
3. **Re-run the Flow:** Use the command `./flow.tcl -design <design_name> -overwrite` to see how your changes affected the physical chip.
    
4. **Check Reports:** Look in `runs/<tag>/reports/synthesis/` for the `area.rpt` and `sta.rpt` files to see if your modifications met the targets.
    

[Practical guide to the OpenLane digital ASIC flow](https://www.youtube.com/watch?v=_AXknRBk4QI)


### Custom Design using Open Lane 

