420  yosys -p "
    read_verilog -sv antikernel-ipcores/serdes/linecode/*.sv;
    read_verilog -sv antikernel-ipcores/interface/ethernet/SGMIIToGMIIBridge.sv;
    hierarchy -check -top SGMIIToGMIIBridge;
    show -format dot -prefix design_hierarchy
"
  421  # 1. Store the file list in a variable (expanding the wildcard)    
  422  SERDES_FILES=$(ls ../antikernel-ipcores/serdes/linecode/*.sv)      
  423  BRIDGE_FILE="../antikernel-ipcores/interface/ethernet/SGMIIToGMIIBridge.sv"
  424  # 2. Run Yosys with the expanded list
  425  yosys -p "
    read_verilog -sv $SERDES_FILES;
    read_verilog -sv $BRIDGE_FILE;
    hierarchy -check -top SGMIIToGMIIBridge;
    show -format dot -prefix design_hierarchy
"
  426  # 1. Flatten the file list into a single line (space-separated)    
  427  ALL_FILES=$(echo ../antikernel-ipcores/serdes/linecode/*.sv ../antikernel-ipcores/interface/ethernet/SGMIIToGMIIBridge.sv)
  428  # 2. Run Yosys with the flattened list
  429  yosys -p "
    read_verilog -sv $ALL_FILES;
    hierarchy -check -top SGMIIToGMIIBridge;
    show -format dot -prefix design_hierarchy
"
  430  # 1. Define the include path and all source files
  431  INC_DIR="../antikernel-ipcores/interface/ethernet"
  432  SERDES_DIR="../antikernel-ipcores/serdes/linecode"
  433  # 2. Run Yosys with the -I flag
  434  yosys -p "
    read_verilog -sv -I$INC_DIR $SERDES_DIR/*.sv ../antikernel-ipcores/interface/ethernet/SGMIIToGMIIBridge.sv;
    hierarchy -check -top SGMIIToGMIIBridge;
    show -format dot -prefix design_hierarchy
"
  435  # Define your paths once to keep the command clean
  436  SERDES_DIR="../antikernel-ipcores/serdes/linecode"
  437  ETH_DIR="../antikernel-ipcores/interface/ethernet"
  438  yosys -p "
    read_verilog -sv -incdir $ETH_DIR $SERDES_DIR/*.sv $ETH_DIR/SGMIIToGMIIBridge.sv;
    hierarchy -check -top SGMIIToGMIIBridge;
    show -format dot -prefix design_hierarchy
"
  439  # 1. Grab all the files into a single line
  440  SERDES_FILES=$(echo ../antikernel-ipcores/serdes/linecode/*.sv)    
  441  BRIDGE_FILE="../antikernel-ipcores/interface/ethernet/SGMIIToGMIIBridge.sv"
  442  INC_DIR="../antikernel-ipcores/interface/ethernet"
  443  # 2. Run Yosys with -I (no space between -I and the path)
  444  yosys -p "
    read_verilog -sv -I$INC_DIR $SERDES_FILES $BRIDGE_FILE;
    hierarchy -check -top SGMIIToGMIIBridge;
    show -format dot -prefix design_hierarchy
"
  445  find ../antikernel-ipcores -name "GmiiBus.svh"
  446  # 1. Grab only the linecode files
  447  SERDES_FILES=$(echo ../antikernel-ipcores/serdes/linecode/*.sv)    
  448  # 2. Run Yosys. We use -auto-top so Yosys figures out the root node automatically
  449  yosys -p "
    read_verilog -sv $SERDES_FILES;
    hierarchy -check -auto-top;
    show -format dot -prefix serdes_hierarchy
"
  450  cat serdes_hierarchy.dot
  451  history | grep "yosys"
  452  tail -n 50 ~/.bash_history
  453  history