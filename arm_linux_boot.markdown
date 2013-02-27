# linux kernel prerequisites and assumptions #

Before goto linux kernel entry, bootloader should prepare the environment for the Linux kernel. This part is done by the bootloader, such as UBoot.

* DRAM is initialized
* Hardware-related initialization tasks are done
* MMU/Cache is disabled
* Specific values should be saved in registers
  * r1:Machine ID
  * r2:pointer of ATAG list(optional)
