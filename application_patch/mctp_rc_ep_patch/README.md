# Execute MCTP RC/EP at the same time
## Board design:
- Please contact ASPEED’s contact window for the AP-note.
## Software patch
- Pleas apply the patch to your repo.
  - kernel 5.4:
  ```
  git am common/*.patch
  git am linux-5.4/*.patch
  ```
  - kernel 5.15:
  ```
  git am common/*.patch
  git am linux-5.15/*.patch
  ``` 
## Modify the dts to match the GPIO selected by the board
```
// file path: arch/arm/boot/dts/aspeed-ast2600-evb.dts
&pcie1 {
      // The GPIO used for monitoring PERST# from the host
      perst-ep-in-gpios = <&gpio0 ASPEED_GPIO(B, 0) GPIO_ACTIVE_HIGH>;
      // The GPIO used output the PERST# from the BMC PCIe RC
      perst-rc-out-gpios = <&gpio0 ASPEED_GPIO(B, 1) GPIO_ACTIVE_HIGH>;
 	status = "okay";
};
```
## The Side-effect of this software patch
- When Host sends the PERST# low to the dedicated GPIOx, AST26x0 MCTP controller of Root Complex Port will be reset due to internal circuit design. The software path will execute below procedure to fix the MCTP over PCIe EP/RC issue and make PCIe RC will disconnect for a while:
  1. Remove the devices on the RC bus
  2. Toggle the RC-PERST (SSPRST# pin) to reset the PCIe EP and PCIe RC
  3. Rescan the devices on the RC bus
