#### Origin Story 

https://www.linkedin.com/feed/update/urn:li:activity:7428935929311965184/?dashCommentUrn=urn%3Ali%3Afsd_comment%3A(7429200218283479040%2Curn%3Ali%3Aactivity%3A7428935929311965184)&dashReplyUrn=urn%3Ali%3Afsd_comment%3A(7429422102346178561%2Curn%3Ali%3Aactivity%3A7428935929311965184)

- Question
	Suppose I can create a Serdes 1GHZ now ,is he going to produce it and then connect it to his PCB ?
- Answer :
	To answer your question directly: **Yes, he would absolutely produce it and connect it to a PCB, but not in the way you might expect.** Andrew Zonenberg is a pioneer in the "Open Source Silicon" movement. His strategy isn't just to make one chip for one board; he is building a **reusable ecosystem**. If you were to give him a verified 1GHz SerDes in SKY130 today, here is the "Andrew Zonenberg Playbook" for how he would use it: 1. The Tapeout (Production) He wouldn't just print one chip. He uses the **Efabless MPW (Multi-Project Wafer)** program. This allows him to "tape out" (manufacture) his design on a silicon wafer alongside dozens of other designers. **The Result:** He gets back about 50 to 100 tiny silicon "dies" (the actual chips without a plastic case). The Packaging (The "Chip" Form) Because he is a high-end hobbyist/engineer, he often does **"reballing"** or uses specialized **QFN/BGA packaging** services. He takes that tiny piece of SKY130 silicon and puts it into a package that can actually be soldered onto a board.3. The "Flyover" PCB Strategy This is the most likely way he would connect your SerDes to his projects. As seen in his **LatentRed** and **Ethernet Switch** projects: **The Motherboard:** He builds a large, 10-layer PCB (usually manufactured at OSHPark or Multech) using standard FR-4 material. **The High-Speed Path:** Since FR-4 is bad for 1GHz+ signals, he uses **Samtec ARF6 "Flyover" cables**. He would place your SerDes ASIC on a small "daughter card" right next to the FPGA. **The Connection:** He would use those high-speed cables to "fly" the signal directly from your ASIC to his FPGA or to an SFP+ port, bypassing the noisy PCB entirely.4. Why your 1GHz SerDes is the "Missing Piece" Currently, Andrew uses FPGAs (like the Kintex-7) because they have built-in "hard" SerDes blocks that can do 10Gbps. However, FPGAs are **expensive and power-hungry** (his switches pull ~150W).**His Goal:** He wants to replace those power-hungry FPGAs with custom SKY130 ASICs.**Your SerDes:** If you provide a 1GHz SerDes, you are giving him the "doorway" that allows his custom chips to talk to the rest of the world at high speed. He would use it to build a **low-power, open-source Ethernet switch** that pulls 50W instead of 150W.
- End
#### Presentation of the Tools :
https://openlane2.readthedocs.io/en/latest/getting_started/newcomers/index.html

https://web.stanford.edu/class/ee183/handouts_win2003/VerilogQuickRef.pdf

https://openlane2.readthedocs.io/en/latest/reference/index.html
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

3/what's VERILATOR doing there ?

4/ Is is compliant with another architecture we know of withouth additional verification ? Can I use a UVM methodology to assert if it convineienvt ? How can I run the iverilog runs on the final netlist using Openlane ?

5/i JUST wanan see a 1GHz signal in a simulation using the verilog he provides , can I ?

6/ I need to check equivalence or isomorphism on the gate level design with another gate level design ? Are there equivalence on the logic verification level : I mean a standard .sv file I can try on this verilog to test it ? I need to find another verilog in an another node that has been used to know whether that is isomorphic if I can check that : I will validate cause I'll have a source to rely on. Or do I need to verify the specs on which he designed the Serdes agains ? https://ioc.exchange/@azonenberg Here find the conventional SERDES here.

7/ Block Diagram
![[Pasted image 20260222051034.png]]
Ok now I need to be able to synthesize the using Openlane ? There is no way I can extract the needed frequency out of these sv files ? What are the specs I can get from his blog and the limits of SKY130 node.
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



###### 3. Verification
Questions: 
1/ 

Answer : 
Verification of Verilog designs, often comprising 70% of the design cycle, ==involves testing Register Transfer Level (RTL) code to ensure it meets specifications using simulation, testbenches, and methodologies like [UVM](https://www.google.com/search?q=UVM&rlz=1C1ONGR_enUS1178US1179&oq=Verification+of+verilog+&gs_lcrp=EgZjaHJvbWUyBggAEEUYOTIICAEQABgWGB4yCAgCEAAYFhgeMgcIAxAAGO8FMgcIBBAAGO8FMgcIBRAAGO8FMgcIBhAAGO8FMgoIBxAAGIAEGKIE0gEINTg3MWowajeoAgCwAgA&sourceid=chrome&ie=UTF-8&ved=2ahUKEwiJxriowOiSAxWf2wIHHaQcMA4QgK4QegQIARAE) (Universal Verification Methodology)== -- According to search : Verification of verilog -- GOOGLE AI Search

- **CLI
		>nix-shell -p iverilog gtkwave
		>iverilog -g2012 -s tb_phy -o sim.out $(find . -name "pcs_tx_8b10b.v") $(find . -name "phy_top_1g.v") $(find . -name "tb_phy.sv") && vvp sim.out
		>gtkwave simulation.vcd


###### 4. UVM
What is MISSING?

The only thing missing from this list is **Post-Layout Functional Verification**.

- **What you did with `iverilog`:** You verified that the _logic_ was correct.
    
- **What OpenLane did:** It verified the _structure_ is correct.
    
- **The missing link:** Most high-end projects take the "final netlist" (the gates) and run your `tb_phy.sv` testbench on it one last time to be 100% sure the timing delays didn't break the logic.


###### 5. 1Ghz Tb 

Does it wait for the first `rd_pos`?

Not exactly. It waits for the **Reset signal to settle** and the first **valid data** to be clocked in.

- Look at the **5 ns** mark in your waveform: The `rst` signal goes low.
    
- Shortly after, `valid_in` goes high at roughly **6.5 ns**.
    
- At the very next rising clock edge (at **7 ns**), the internal logic finally "captures" its first real values, and the signals turn from **Red (`x`)** to **Green (Known values)**.
    
 3. The "SerDes" Connection

In a real SerDes or high-speed link, this "red" period is what we call the **Initialization/Link Training phase**.

- **Synchronization:** A SerDes cannot just start sending data immediately. It must wait for the Clock and Data Recovery (CDR) to lock and for the disparity to be initialized.
    
- **Comma Characters:** Usually, the first thing a 1G PHY sends isn't "Data 00," but a special **K28.5 (Comma)** character. This character is used by the receiver to find the "start" of the 10-bit word and to establish the first `rd` state.


###### 6. Verification synthesis : 

i can't understand a single thing of connection with the pdk and the .v generated of the gate list generated after the Openlane finished. 

What does the Makefile do ? 

By using this Makefile to run an exhaustive Python test against your `./runs/.../eth_phy_1g.v` file, you are performing **Post-Synthesis Functional Verification**. If the Python test (which knows the "perfect" 8b10b standard) passes when talking to your synthesized gates, you have confirmed that your silicon netlist is a perfect logical match for the Ethernet standard.


###### 7. Specs 
| **Metric**    | **Target Specification**                  |
| ------------- | ----------------------------------------- |
| **Speed**     | 1.25 Gbps (Min) / 2.5 Gbps (Target)       |
| **Protocol**  | 8b/10b (1000BASE-X)                       |
| **Integrity** | BER < $10^{-12}$                          |
| **Tooling**   | 100% Open Source (OpenLane/Sky130)        |
| **I/O**       | Differential Signaling (CML or LVDS-like) |
In Sky130, a **1 ns cycle** is extremely tight. A typical rule of thumb for 130nm is that you can fit roughly **15–20 logic levels** (standard gates like AND/OR) in 1 ns.

Some Research papers on the architecture I found and the operating frequencies : 
http://semanticscholar.org/paper/A-1.1-GHz-8B-10B-encoder-and-decoder-design-Wang-Hua/9c8b583cf73a34a3462e9a2566b1806976a68812


###### 8. End 


#### Exploration of the automated flow

##### Questions  :  
1/ What's the difference between a SV file and .V file for Openlane. Can they interchangeably work to synthesis via Yosys ? 

2/ Ok So part of learning to use Openlane is to master the config.json right ?

3/ Ok let's synthesize this one, I just want to probe the synthesized one not the post layout one. Let's go step by step push it to its hardest and see what I can do with it. But I don't know what testbench it should go through ? I need to look at the other directories in the serdes folder I copied I guess. What should I grep for to know the logic tests he's running so that I can sweep oveer different frequencies around the needed one and see how far it can goes 

4/But I need to check which modules are using the serdes part right, Is there a tool to querry all the files in the repo that uses the serdes ? 

5/I would need to recurse on this result to see at which point he stops and define real specs. Can you write a code that greps what has been grepped to see a top of the top of the top module that can I can read to look for specs ?

6/ I got some files in the spanning tree, yet some files seems to be missing I need to invistigate the errors. However a question : Do I need to grep the sources nodes of the graph and check if those are used in other directories afterwards ? Q.Difficulty=2 .
##### Answers :  
1/

| **Feature**           | **Verilog (.v)**                                     | **SystemVerilog (.sv)**                                                                                              |
| --------------------- | ---------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| **Logic Intent**      | Uses `always @(*)`. Can accidentally create latches. | Uses `always_comb`. Yosys will warn you if it can't be pure combinational logic.                                     |
| **Data Types**        | Distinguishes between `wire` and `reg`.              | Uses `logic`, which is much more flexible for ASIC synthesis.                                                        |
| **Synthesis Support** | 100% supported by almost every tool in existence.    | Yosys supports the **synthesizable subset**. It will ignore "testbench" features like `initial` blocks or `program`. |
2/


3/Found something : 
	//SERDES interface: 125 MHz, 32 bit data plus control flags
	//(8B/10B coding handled by transceiver)
	I decided I DON'T want to see the whole context at once I grepped once :
	grep -r "Encode8b10b" . --include="*.sv" --include="*.v" -> Gave 2 other files it's enough. 
	I want then to go to those files and look up if there is a test on them other than large specs, real logic tests. I need to see the critical path ?  And go from there to define my Pareto Front. That's a good way I thing to do things. A good way to identify the module is to build a spanning tree.I need to build a spanning tree of the verilog files of Directed graph in which an edge is a being a module that instantiates another module.# Read all your design files read_verilog -sv antikernel-ipcores/serdes/linecode/*.sv read_verilog -sv antikernel-ipcores/interface/ethernet/*.sv # Establish the hierarchy hierarchy -check -auto-top # Generate the graph # -prefix specifies the output filename show -format dot -prefix design_hierarchy
	



- 4/1. Identifying the "Top Level"

	In your case, **`SGMIIToGMIIBridge.sv`** is likely your "target" top-level module for 1Gb Ethernet.
	- **SGMII (Serial Gigabit MII):** This is the high-speed serial side (where the 8b/10b logic is used to prepare data for the wire).
	- **GMII (Gigabit MII):** This is the parallel side (where the MAC lives).
	I would need to recurse on this result to see at which point he stops and define real specs. 

	[[commands_history]] <- Here you can find command to get the graph to work out in the nix-shell after setting up Yosys the right way in the openlane2 directory and getting sourcing the right python enviroement for Yosys python command.
	
	nix develop .#default
	# 1. Gather all SerDes files into a variable
	SERDES_FILES=$(echo ../antikernel-ipcores/serdes/linecode/*.sv)
	# 2. Run Yosys to build the hierarchy
	yosys -p "
    read_verilog -sv $SERDES_FILES;
    hierarchy -check -auto-top;
    show -format dot -prefix serdes_hierarchy

5/

6/ I think that I need to figure out how to remove the error , especially to get to the Ethernet folder connection somehow.
ok I added : touch ../antikernel-ipcores/interface/ethernet/GmiiBus.svh

https://docs.google.com/drawings/d/1GEjCoLwY57bsuZoj5ymyXoToIEOC0H4j2SEYsqQupM8/edit

Ok apparently the error : TOK error is a common error and I should get around it using the s2v2s code.



##### End  :  
#### Next step


### Custom Design using Open Lane 

