# Name: Makefile
# Project: hid-custom-rq example
# Author: Christian Starkjohann
# Creation Date: 2008-04-07
# Tabsize: 4
# Copyright: (c) 2008 by OBJECTIVE DEVELOPMENT Software GmbH
# License: GNU GPL v2 (see License.txt), GNU GPL v3 or proprietary (CommercialLicense.txt)
# This Revision: $Id: Makefile 692 2008-11-07 15:07:40Z cs $

SHELL=C:/Windows/System32/cmd.exe

DEVICE  = atmega8
F_CPU   = 16000000
FUSE_L  = 0xBE
FUSE_H  = 0xD1
AVRDUDE = avrdude -c usbtiny -P usb -p $(DEVICE) # edit this line for your programmer

CFLAGS  = -Iusbdrv -Iws2812 -I. -DDEBUG_LEVEL=0 -funsigned-char -funsigned-bitfields
OBJECTS = usbdrv/usbdrv.o usbdrv/usbdrvasm.o usbdrv/oddebug.o ws2812/light_ws2812.o main.o

COMPILE = avr-gcc -Wall -Os -DF_CPU=$(F_CPU) $(CFLAGS) -mmcu=$(DEVICE)
COMPILEPP = avr-g++ -Wall -Os -DF_CPU=$(F_CPU) $(CFLAGS) -mmcu=$(DEVICE)

ifeq ($(OS), Windows_NT)
REMOVE = del /f
PYTHON = python
else
REMOVE = rm -f
PYTHON = python3
endif

# symbolic targets:
help:
	@echo "This Makefile has no default rule. Use one of the following:"
	@echo "make hex ....... to build main.hex"
	@echo "make program ... to flash fuses and firmware"
	@echo "make fuse ...... to flash the fuses"
	@echo "make flash ..... to flash the firmware (use this on metaboard)"
	@echo "make dump ...... dump eeprom"
	@echo "make dumpflash . dump flash"
	@echo "make defaults .. write default eeprom"
	@echo "make clean ..... to delete objects and hex file"
	@echo "make deploy .... program, increment serial, defaults"

hex: main.hex

program: flash fuse

# rule for programming fuse bits:
fuse:
	@[ "$(FUSE_H)" != "" -a "$(FUSE_L)" != "" ] || \
		{ echo "*** Edit Makefile and choose values for FUSE_L and FUSE_H!"; exit 1; }
	$(AVRDUDE) -U hfuse:w:$(FUSE_H):m -U lfuse:w:$(FUSE_L):m

# rule for uploading firmware:
flash: main.hex
	$(AVRDUDE) -U flash:w:main.hex:i

dump: 
	$(AVRDUDE) -U eeprom:r:blinkstick-eeprom.hex:i

dumpflash: 
	$(AVRDUDE) -U flash:r:blinkstick-flash.hex:i

defaults: 
	$(AVRDUDE) -B 3 -U eeprom:w:eeprom.hex:i

# rule for deleting dependent files (those which can be built by Make):
clean:
	$(REMOVE) -f main.hex main.lst main.obj main.cof main.list main.map main.eep.hex main.elf 
	$(REMOVE) -f main.o usbdrv/oddebug.o usbdrv/usbdrv.o usbdrv/usbdrvasm.o main.s usbdrv/oddebug.s usbdrv/usbdrv.s ws2812/light_ws2812.o

.cpp.o:
	$(COMPILEPP) -c $< -o $@


# Generic rule for compiling C files:
.c.o:
	$(COMPILE) -c $< -o $@

# Generic rule for assembling Assembler source files:
.S.o:
	$(COMPILE) -x assembler-with-cpp -c $< -o $@
# "-x assembler-with-cpp" should not be necessary since this is the default
# file type for the .S (with capital S) extension. However, upper case
# characters are not always preserved on Windows. To ensure WinAVR
# compatibility define the file type manually.

# Generic rule for compiling C to assembler, used for debugging only.
.c.s:
	$(COMPILE) -S $< -o $@

# file targets:

main.elf: $(OBJECTS)
	$(COMPILE) -o main.elf $(OBJECTS)

main.hex: main.elf
	$(REMOVE) main.hex main.eep.hex
	
	avr-objcopy -j .text -j .data -O ihex main.elf main.hex
	avr-size main.hex

# debugging targets:

disasm:	main.elf
	avr-objdump -d main.elf

cpp:
	$(COMPILE) -E main.cpp

increment:
	@ $(PYTHON) -m pip install intelhex
	$(PYTHON) increment.py
	

deploy: program increment defaults
