# RPi OpenOcd Tips

## openocd.cfg
```
# SPDX-License-Identifier: GPL-2.0-or-later

#
# Config for using Raspberry Pi's expansion header
#
# This is best used with a fast enough buffer but also
# is suitable for direct connection if the target voltage
# matches RPi's 3.3V and the cable is short enough.
#
# Do not forget the GND connection, pin 6 of the expansion header.
#

adapter driver bcm2835gpio

# Raspi2 and Raspi3 peripheral_base address
# bcm2835gpio peripheral_base 0x3F000000
# Raspi4 peripheral_base address 
bcm2835gpio peripheral_base 0xFE000000

# Transition delay calculation: SPEED_COEFF/khz - SPEED_OFFSET
# These depend on system clock, calibrated for scaling_max_freq 900MHz
# bcm2835gpio speed SPEED_COEFF SPEED_OFFSET
# bcm2835gpio speed_coeffs 225000 36
# Raspi3 BCM2837 (1200Mhz): 
# bcm2835gpio speed_coeffs 194938 48
# Raspi4 BCM2711 (1500Mhz):
bcm2835gpio speed_coeffs 236181 60

# Each of the JTAG lines need a gpio number set: tck tms tdi tdo
# Header pin numbers: 23 22 19 21
# adapter gpio tck -chip 0 11
# adapter gpio tms -chip 0 25
# adapter gpio tdi -chip 0 10
# adapter gpio tdo -chip 0 9

# Each of the SWD lines need a gpio number set: swclk swdio
# Header pin numbers: 22 18
adapter gpio swclk -chip 0 25
adapter gpio swdio -chip 0 24

transport select swd

set CHIPNAME nrf52840
source [find target/nrf52.cfg]

# Uncomment & lower speed to address errors, defaults to 1000
# adapter speed 800

init
targets
reset halt
```

## commands
### backup firmware
* check banks
  ```
  > flash banks
  #0 : nrf52840.flash (nrf5) at 0x00000000, size 0x00000000, buswidth 1, chipwidth 1
  #1 : nrf52840.uicr (nrf5) at 0x10001000, size 0x00000000, buswidth 1, chipwidth 1
  ```
* dump flash memories
  ```
  > flash read_bank 0 bmp-bank0.bin
  nRF52840-xxAA(build code: D0) 1024kB Flash, 256kB RAM
  wrote 1048576 bytes to file bmp-bank0.bin from flash bank 0 at offset 0x00000000 in 8.368806s (122.359 KiB/s)
  
  > flash read_bank 1 bmp-bank1.bin
  wrote 4096 bytes to file bmp-bank1.bin from flash bank 1 at offset 0x00000000 in 0.068702s (58.222 KiB/s)
  ```

* verfy dumped memeories  
  ```
  > flash verify_bank 0 bmp-bank0.bin
  read 1048576 bytes from file bmp-bank0.bin and flash bank 0 at offset 0x00000000 in 8.366240s (122.397 KiB/s)
  contents match

  > flash verify_bank 1 bmp-bank1.bin
  read 4096 bytes from file bmp-bank1.bin and flash bank 1 at offset 0x00000000 in 0.076019s (52.618 KiB/s)
  contents match
  ```
  
### write firmware
* clear flash memories
  ```
  > nrf5 mass_erase                   
  Mass erase completed.
  ```

* write flash memories
  ```
  > flash write_bank 0 bmp-bank0.bin  
  wrote 1048576 bytes from file bmp-bank0.bin to flash bank 0 at offset 0x00000000 in 7.820391s (130.940 KiB/s)

  > flash write_bank 1 bmp-bank1.bin    
  wrote 4096 bytes from file bmp-bank1.bin to flash bank 1 at offset 0x00000000 in 0.096388s (41.499 KiB/s)
  ```
* verify flash memories 
  ```
  > flash verify_bank 0 bmp-bank0.bin 
  read 1048576 bytes from file bmp-bank0.bin and flash bank 0 at offset 0x00000000 in 8.369584s (122.348 KiB/s)
  contents match

  > flash verify_bank 1 bmp-bank1.bin 
  read 4096 bytes from file bmp-bank1.bin and flash bank 1 at offset 0x00000000 in 0.081877s (48.854 KiB/s)
  contents match
  ```
