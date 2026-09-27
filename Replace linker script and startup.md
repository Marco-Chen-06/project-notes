## Resources
- [Zero to main()](https://interrupt.memfault.com/tag/zero-to-main/)
- [Everything You Never Wanted To Know About Linker Script](https://mcyoung.xyz/2021/06/01/linker-script/)
- [RM0351 Reference Manual](https://www.st.com/resource/en/reference_manual/rm0351-stm32l47xxx-stm32l48xxx-stm32l49xxx-and-stm32l4axxx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf)
- [DS10198 STM32L476RG Datasheet](https://www.st.com/resource/en/reference_manual/rm0351-stm32l47xxx-stm32l48xxx-stm32l49xxx-and-stm32l4axxx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf)
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

### Linker
- `.` - Location Counter. It is a pointer to the current memory address in the linker script. 
- `ALIGN(N)` - It is just a one-time rounding command that forces the location counter to be rounded up to the next multiple of N bytes. So if the counter is at 0x08000001 then ALIGN(4) will shift counter to 0x08000004.
- `>` - It tells the linker to assign the memory region to the VMA. If an LMA is not explicitly declared using `AT>`, the linker defaults the LMA to equal the VMA. 
- `KEEP()` - Forces the linker to preserve a section in the final binary even if it appears unused by the software. So if I don't want something optimized out by the `--gc-sections` flag that I will inevitably use, I better wrap it in `KEEP()`.


```
// step by step example
MEMORY
{
	FLASH (rx) : ORIGIN = 0x08000000, LENGTH = 1024K
	SRAM1 (rwx) : ORIGIN = 0x2000 0000, LENGTH = 96K
}

SECTIONS
{
    // first example bullet point starts here
	.isr_vector :
	{
		. = ALIGN(4);
	    KEEP(*(.isr_vector)) /* Startup code */
	    . = ALIGN(4);
	} >ROM
	  
	.text :
	{
		...
	} >ROM
}
```
- Before looking at any code inside {...}, the linker sees `>ROM`. Since `.isr_vector` is the very first section targeting ROM, the linker says, ok initialize `.` to the ORIGIN of ROM (0x0800 0000).
	- Also note the `.` in `.isr_vector` is just GNU convention, it is not the location counter...
- Now the linker goes inside the {...} and sees `. = ALIGN(4)` so the linker says, well `.` is at 0x0800 0000 which is already a multiple of 4 bytes so I don't gotta do anything.
- Then the linker sees `KEEP(*(.isr_vector))`. So the linker searches through all `.o` files passed into the gcc link command for any section tagged with `.isr_vector` (It will probably be find something in the startup assembly code). Then the linker copies those raw bytes into 0x0800 0000 and increments `.` by the size of the vector table.
- Then the linker sees `. = ALIGN(4)` and checks if `.` is aligned to a 4-byte boundary and if it isn't, then it rounds `.` up.
- Now the linker closes `.isr_vector` and goes to the `.text :` block where it reads ` >ROM`, so the linker moves `.` to whever it ended off in `.isr_vector`

### LMA vs VMA
Load Memory Address (LMA) - Address where the data or code physically lives when the device is powered off.
Virtual Memory Address (VMA) - Address where the section resides and executes during runtime

```
// LMA vs VMA example
.data :
{
    _sdata = .;
    *(.data*)
    _edata = .;
} > SRAM AT> FLASH
```
- The code above says `> SRAM AT> FLASH`. This means the LMA is SRAM and the VMA is flash. 

Note that .data holds initialized global and static variables whose values are known at compile time. Suppose you declare a global variable below.

```
int delay_ms = 500; // will live in .data
```
- When the board is unplugged, the value of 500 is stored permanently in FLASH. (LMA)
- When power is applied, the startup code should physically copy raw values from FLASH to SRAM.
- Now, when you decide to modify the variable (`blink_delay = 1000;`), there is no problem because it lives in SRAM which is writeable.
