## Resources
- [Zero to main()](https://interrupt.memfault.com/tag/zero-to-main/)
- [Everything You Never Wanted To Know About Linker Script](https://mcyoung.xyz/2021/06/01/linker-script/)
- [RM0351 Reference Manual](https://www.st.com/resource/en/reference_manual/rm0351-stm32l47xxx-stm32l48xxx-stm32l49xxx-and-stm32l4axxx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf)
- [DS10198 STM32L476RG Datasheet](https://www.st.com/resource/en/reference_manual/rm0351-stm32l47xxx-stm32l48xxx-stm32l49xxx-and-stm32l4axxx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf)
1111 0100 0000 0000
plus 1011 1111 1111= 1+2+4+8+16+32+64+128+256+512+1024
1111 1111 1111 1111
## Notes

### Flash
- Flash Memory Region starts at 0x0800 0000 and ends at 0x0810 0000 (RM0351, 77) 
- The G in STM32L476RG means 1 MB flash (DS10198, 261) so the below info is assuming 1 MB dual bank flash organization. This sort of information is usually in the "Ordering Information" section
- Flash memory is divided into 2 banks (RM0351, 98). Each bank has the following:
	- A main memory block containing 256 pages of 2KB size (each page is 8 rows of 256 bytes -> 2KB). This starts at 0x0800 0000.
	- Information block containing system memory (28 KB), OTP area (1KB, only for bank 1), and option bytes (16 bytes)

### SRAM
- SRAM1 is 96 KB (RM0351, 88) and starts at 0x2000 0000 (RM0351, 77)
### Assembly
- STM32L476RG startup code uses ARM Cortex-M4 assembly
- There's a strategy where you can loop by jumping to loop\<label\> and then if you want to loop then you bcc (branch if carry clear) to \<label\> 