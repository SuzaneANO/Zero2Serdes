- **Press `Win + R`** on your keyboard.
    
- Type `cmd` and press Enter.
    
- In the black command prompt window, type:
```
    taskkill /F /IM process.exe
```

- In order to get this ollama to work (Never forget to be in the wsl shell):
```
nix-shell -p ollama 
ollama serve
```
![[Pasted image 20260223235157.png|200]]

You can open two terminal one hosting nix serve and one running command to the ollama server api.


![[Pasted image 20260302045203.png|200]]

```
python ../panamax/netlist/layout/asy2sym.py --testbench "../panamax/netlist/layout/sky130_fd_io__top_gpio_ovtv2.asy"
```

```
xschem ../panamax/netlist/layout/sky130_fd_io__top_gpio_ovtv2.sym
```


```
/home/adamsbane/.volare/volare/sky130/versions/af3485525297d5cbe93c129ea853da2d588fac41/sky130A/libs.tech/xschem/sky130_fd_pr
```



```
unit n
voltage 1.2
slope 1
format hspice

beginfile stimuli.cir

* Power Supplies
VVDDIO VDDIO 0 3.3
VVDDIO_Q VDDIO_Q 0 3.3
VVDDA VDDA 0 3.3
VVCCHIB VCCHIB 0 1.2
VVCCD VCCD 0 1.2
VVSSA VSSA 0 0
VVSSD VSSD 0 0
VVSSIO VSSIO 0 0
VVSSIO_Q VSSIO_Q 0 0

* Mode Controls (Setting to Output Mode)
V_DM0 DM[0] 0 1.2
V_DM1 DM[1] 0 1.2
V_DM2 DM[2] 0 1.2
V_OE OE_N 0 0

* Input Signal (Toggling)
V_IN_SIG IN 0 PULSE(0 1.2 1n 1n 1n 10n 20n)

* Digital Tie-offs
V_SLOW SLOW 0 0
V_HLD_OVR HLD_OVR 0 0
V_ENABLE_H ENABLE_H 0 3.3
V_HLD_H_N HLD_H_N 0 3.3
V_ENABLE_VDDA_H ENABLE_VDDA_H 0 3.3
V_ANALOG_EN ANALOG_EN 0 0
V_ENABLE_INP_H ENABLE_INP_H 0 3.3
V_IN_H IN_H 0 0
V_VINREF VINREF 0 0
V_ANALOG_POL ANALOG_POL 0 0
V_ANALOG_SEL ANALOG_SEL 0 0
V_TIE_HI_ESD TIE_HI_ESD 0 1.2
V_TIE_LO_ESD TIE_LO_ESD 0 0
V_PAD_A_ESD_0_H PAD_A_ESD_0_H 0 0
V_PAD_A_ESD_1_H PAD_A_ESD_1_H 0 0
V_PAD_A_NOESD_H PAD_A_NOESD_H 0 0
V_ENABLE_VSWITCH_H ENABLE_VSWITCH_H 0 0
V_ENABLE_VDDIO ENABLE_VDDIO 0 3.3
V_INP_DIS INP_DIS 0 0
V_VTRIP_SEL VTRIP_SEL 0 0
V_HYS_TRIM HYS_TRIM 0 0

* Slew & Bias Controls
V_IB_SEL0 IB_MODE_SEL[0] 0 0
V_IB_SEL1 IB_MODE_SEL[1] 0 0
V_SLEW0 SLEW_CTL[0] 0 0
V_SLEW1 SLEW_CTL[1] 0 0

* Analog Muxes
V_AMUXA AMUXBUS_A 0 0
V_AMUXB AMUXBUS_B 0 0
V_VSWITCH VSWITCH 0 0

endfile
```


```
unit n
voltage 1.8
slope 1
format hspice

beginfile tb_gpio_ovtv2_stimuli.cir

.option SCALE=1e-6 
.option method=gear 
.option wnflag=1 

* --- Global Parameters & Variations ---
.param VCCGAUSS = agauss (1.8, 0.05, 1) 
.param VCC = 1.8 
.param TEMPGAUSS = agauss(40, 30, 1) 
.option temp = 25 

* --- Power Supplies ---
VVDDIO VDDIO 0 3.3
VVDDIO_Q VDDIO_Q 0 3.3
VVDDA VDDA 0 3.3
VVCCHIB VCCHIB 0 {VCC}
VVCCD VCCD 0 {VCC}
VVSSA VSSA 0 0
VVSSD VSSD 0 0
VVSSIO VSSIO 0 0
VVSSIO_Q VSSIO_Q 0 0

* --- Functional Controls (Output Mode) ---
V_DM0 DM[0] 0 {VCC}
V_DM1 DM[1] 0 {VCC}
V_DM2 DM[2] 0 {VCC}
V_OE OE_N 0 0
V_IN_SIG IN 0 PULSE(0 {VCC} 1000 1 1 10000 20000)

* --- Tie-Offs for 3.3V Domain Signals ---
V_ENABLE_H ENABLE_H 0 3.3
V_HLD_H_N HLD_H_N 0 3.3
V_ENABLE_VDDA_H ENABLE_VDDA_H 0 3.3
V_ENABLE_INP_H ENABLE_INP_H 0 3.3
V_ENABLE_VDDIO ENABLE_VDDIO 0 3.3

* --- Tie-Offs for Core Logic / Ground Domain ---
V_SLOW SLOW 0 0
V_HLD_OVR HLD_OVR 0 0
V_ANALOG_EN ANALOG_EN 0 0
V_IN_H IN_H 0 0
V_VINREF VINREF 0 0
V_ANALOG_POL ANALOG_POL 0 0
V_ANALOG_SEL ANALOG_SEL 0 0
V_TIE_HI_ESD TIE_HI_ESD 0 {VCC}
V_TIE_LO_ESD TIE_LO_ESD 0 0
V_PAD_A_ESD_0_H PAD_A_ESD_0_H 0 0
V_PAD_A_ESD_1_H PAD_A_ESD_1_H 0 0
V_PAD_A_NOESD_H PAD_A_NOESD_H 0 0
V_ENABLE_VSWITCH_H ENABLE_VSWITCH_H 0 0
V_INP_DIS INP_DIS 0 0
V_VTRIP_SEL VTRIP_SEL 0 0
V_HYS_TRIM HYS_TRIM 0 0
V_IB_SEL0 IB_MODE_SEL[0] 0 0
V_IB_SEL1 IB_MODE_SEL[1] 0 0
V_SLEW0 SLEW_CTL[0] 0 0
V_SLEW1 SLEW_CTL[1] 0 0
V_AMUXA AMUXBUS_A 0 0
V_AMUXB AMUXBUS_B 0 0
V_VSWITCH VSWITCH 0 0

* --- Simulation Control Loop ---
.control
  option seed=9 
  let run=0 
  dowhile run <= 100 
    save all 
    tran 1n 4000n uic 
    remzerovec 
    write tb_gpio_ovtv2.raw 
    set appendwrite 
    reset 
    let run = run + 1 
  end 
.endc

endfile
```



```
nix develop .#dev

```

```
V[Name] [Node+] [Node-] [DC_Value] SIN(Voffset Vamplitude Freq Tdelay Theta Phase)
```

```
Bash # Inside your .envrc 
use flake 

Then run: Bash 

direnv allow 
How to verify it’s working Open a brand new terminal and just cd into your project work directory: Bash 
cd /mnt/c/Users/AdamSbane/Downloads/OpenLane/openlane2/rply_ex0/work xschem --version
```


```
grep -rnw . -e "sky130_fd_pr__res_generic_po" --include=\*.{v,spice,lib,mag,lef}
```

