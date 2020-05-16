2019/9/6
# PME Turn off message 梳理
- BIOS发送message **BIOSSMC_MSG_SleepEntry**
  - **void fSMC_MSG_SleepEntry(uint32_t MsgPort)**
  - **S5_EntrySequence()**
    - writeSmnReg(mmNB_PCI_ARB_alt_1, nb_pcie_arb.val) //set PMETurnoff = 1
    - **waiting for PMEToACKStatus = 1**
