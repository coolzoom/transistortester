Author: Karl-Heinz Kübbeler   (kh_kuebbeler@web.de) , Mai 2016

This is a assembly language version of optiboot 6.2 !
The required space could be reduced with the same functionality,
so that a support for EEprom read/write could be integrated in 510 Bytes.
Therefore the EEprom support is integrated by default, but you can
also deselect the feature with a option SUPPORT_EEPROM=0 .

The clock rate is preselected for many configurations in the Makefile,
but you can select a different value by setting the option AVR_FREQ=xxxxxx ,
where xxxxxx is the new frequency in cycles per second (Hz).

Example:
***********************************************
make atmega328p AVR_FREQ=8000000 BAUD_RATE=9600
***********************************************
This will generate a bootloader code for a ATmega328P with 8 MHz clock frequency
and a serial communication at 9600 baud.


For a Arduino UNO only the simple call is needed for generating the program data:
***************
make atmega328p
***************
ATmega328P with 16MHz and 115200 Baud.

With the simular call:
*******************
make atmega328p_isp 
*******************
the program data will be transfered to the ISP-programmer and additionaly all fuses are set.
You should know, that the Flash memory content is completely erased before
programming the new bootloader code. Any application program must be programmed
again later with the serial interface.
The ISP-programmer should be set in the Makefile.isp file correctly before you
use any "make  xxx_isp" type make call.
Typically the program avrdude is used to transfer the data to the programmer.
A list of sample configurations you can find in the makeall batch file.
The Makefile make use of utilities, which determine the size of the bootloader program
automatically and set the start address and the required fuses right
according to the actual size.
This kind of automatic is tested at a Linux system and should run at any Linux system.
But it is completely untested for any Windows system.
This optiboot bootloader is only tested with a Arduino-UNO and -Nano board and with 
ATmega8, ATmega32, ATmega1284p and a ATtiny84.

The optional implementation of a software UART requires a little more space, but should
also match to a 512 Byte boot partition.
Lower baud rates can often better supported with the SOFT_UART option and you are free to
select any present port pin for the serial input and output (assembly version only!).
With a 20 MHz CPU you can select baud rates of 300 Baud and higher, with lower CPU clock rates
you can even select lower baud rates.
With the hardware solution and a 20 MHz CPU the lowest selectable standard serial rate is 600 baud,
with a 16 MHz CPU you can also select 300 baud.
If your CPU clock rate is sufficient, the standard serial rate of 115200 baud
is a good selection to get a fast transmission. 
For a good CPU clock rate match is also 1 Mbaud possible with the hardware UART.

Only the ATtiny84 example requires the VIRTUAL_BOOT_PARTITION option.
Normally the start vector is switched with the fuses (BOOTRST bit) to the
bootloader start addresss, but this feature is missing at the ATtiny84.
If the VIRTUAL_BOOT_PARTITION is selected, the AVR reset vector
can remain at the application start address (0x0000).
Instead of switching the Reset start address the Reset interrupt vector of the
application program should be changed to the bootloader start address.
The original start address of the application program is moved to another
interrupt vector and will be used later by a bootloader timeout.
For that feature the bootloader try to change the interrupt vectors
during programming the application program and simulates the correct
back readed data for a following verify.
The host, which send the data to the bootloader, did not notify the modification.
But the implemented trick will not work for a second separate verify of
the Flash content by the host.
I believe, that it will be better to implement this feature by the application 
program itself. Another way is to use a patch program to modify the program data,
which are typically available at the host system in hex format.
A patch program can analyse the data more in detail as it is possible within
the space limited boot loader.
One of the difficulties with the patch is, that for Flash memory space above 8K
the interrupt vector addresses can hold a JMP or also a RJMP instruction!
Currently the bootloader assumes, that only JMP intructions are used in the 
interrupt vector table, if the processor has more than 8 kByte flash memory.

=======================================================================================
Available options
=======================================================================================
AVR_FREQ		specifies the frequency (Hz), at which the AVR mikrocontroller should run.

BAUD_RATE		specifies the baud rate for the serial interface.

SOFT_UART		Whith this option you select a software UART solution instead of
			a hardware UART. With the software solutions you are free to select
			any available port pins.

UART_RX			You can specify any input pin available at your chip for serial
			data input with the SOFT_UART option. You must only specify the
			input pin with two characters. The first uppercase alphabetical character
			specifies the Port and the following numeric character specifies
			the bit number of the port (0-7).

UART_TX			You can also select the serial output pin for the SOFT_UART Option.
			The format for selecting the port pin is the same as for the input (RX).
			You can also select different ports for input and output.

UART			specifies the hardware UART-interface to use.
			Possible values are 0 to 3, if the selected AVR support several
			hardware UART interfaces. The default number is 0.

LED_START_FLASHES	specifies, how often the LED should flicker, before the serial 
			communication is started. If you specify a 0, no LED output is done.
			If you specify a 1, the LED Flash only onces without the program loop.

LED_DATA_FLASH		let the LED light during wait for a serial character input. 

LED			specifies with two characters the port and the bit number (0-7) of
			the LED signal output. 
			The first character is the port code letter (A,B,C,D,E,F,G,H,J,K,L).

C_SOURCE		with this option set to 1 you can select the C-source instead of
			the standard assembler source (optiboot.S).
			The with the C-source generated program requires more Flash memory.

SUPPORT_EEPROM		If you set this value to a nonzero value, the bootloader is created 
			with the EEprom write and read support.
			The assembly version of the bootloader support the EEprom use 
			without the option setting. But you can deselect this feature
			by setting the option to zero (SUPPORT_EEPROM).
			You should know, that the Flash memory content can be destroyed
			by using a EEprom write without the EEprom support!

BIGBOOT			This option was earlier required to match the setting of start address
			and AVR fuses to a program size above 512 Bytes.
			The adaption of the correct starting address and the correct setting
			of the AVR fuses is done now automatically.
			But the option is not removed. You can use it now to test the
			automatical adaption or to set the bootloader start address to
			a specific address. The bootloader code will grow for as many
			bytes, as you specify. 500 additional bytes let the size of
			the bootloader grow, so that the bootloader start address
			will change from 0x7e00 to 0x7c00 for example. 

VIRTUAL_BOOT_PARTITION 	this option changes two interrupt vector jmp addresses,
			so that the bootloader is started with the reset vector.
			Another interrupt vector is used to save the original
			start address of the application program.
			The JMP instruction or the target address of the JMP 
			is saved in the last two bytes of the EEprom memory.
			So a verify of the loaded flash data is possible
			as long as this EEprom data are not destroyed by
			the application program.

TIMEOUT_MS		This option specifies a time limit in ms units for receiving boot data. 
			After this time without data the boot process is aborted and it is tried to
			start the user program.
			Possible values for TIMEOUT_MS are 500, 1000, 2000, 4000 and 8000.
			The effective value can be limited to 2 seconds because of processor limits for the watchdog.

OSCCAL_CORRECTION	This option specifies a correction value for the OSCCAL Byte of the AVR-Processor.
			The correction value is only effectual, if the internal RC-oscillator of the AVR processor
			is used, not with the crystal operation.
			Because the produced Baud-rate depends directly of the processor clock,
			a calibrated clock frequency is important for a successfull serial data transfer.

Recommendation:
===============
The optiboot loader transfer the original content of the MCUSR register to the application
program in the r2 register. The MCUSR register shows the reason for the reset.
I think, that it would be better to use a AVR hardware IO register like GPIOR0.
For a C application program ist it easier to read the IO-Register content as the content
of any working register (r0-r31).
The GPIORx register has no special function and can therefore used without any restriction.
A additional use of this register by a application program is not prohibited.
For AVR mikrocontrollers without this IO register group the counter register OCR2 is used.

List of the supported AVR-Prozessors:
==========================
ATmega8/16/32/64
ATmega162/163
ATmega164/324/644/1284 (P)
ATmega88/168/328  (P)
ATmega165/325/3250/6450
ATmega169/329/649/6490
ATmega640/1280
AT90CAN32/64/128
AT90PWM2/3
ATtiny84/85/88/861/1634
For the most processors the finish test is missing!

I look forward for any replies to the "assembly language optiboot" subject.
