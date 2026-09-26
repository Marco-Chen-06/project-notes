## Resources
- [Zero to main()](https://interrupt.memfault.com/tag/zero-to-main/)
- [Everything You Never Wanted To Know About Linker Script](https://mcyoung.xyz/2021/06/01/linker-script/)
- [RM0351 Reference Manual](https://www.st.com/resource/en/reference_manual/rm0351-stm32l47xxx-stm32l48xxx-stm32l49xxx-and-stm32l4axxx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf)
-
1111 0100 0000 0000
plus 1011 1111 1111= 1+2+4+8+16+32+64+128+256+512+1024
1111 1111 1111 1111
## Notes
- Flash memory starts at 0x0800 0000 and ends at 0x0810 0000 (RM0351, 77) 
- Flash memory divided into 2 banks. Each bank contains 256 pages, 2 KB each. (RM0351, 98)
	- Each page is 8 rows of 256 bytes. This corresponds to 2048 bytes = 2KB
	- Both banks collectively take up 1024 KB -> 0x0100 0000. That's why flash goes from 0x0800 0000 -> 0x0810 0000