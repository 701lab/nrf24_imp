# nrf24l01p_imp
**nrf24l01p_imp** is NRF24L01+ implementation-independent library. **Implementation-independent** (impi for short) means that this library can be used on any device from any manufacturer: STM, TI, NXP, Atmel with whatever architecture MCU or MPU uses. To work like that impi-libraries require your project to implement some basic functions in a specific way. This library can be library to understand at first, but bear with us because after reading this instruction we hope that you'll understand how powerful is this approach.

More information about impi-libraries can be found [here] - (add a link to explanation)

This particular library contains only basic nrf24l01+ functions but can be improved if needed. List of things, that are not implemented is at the end of this readme. 

## How to use 
  Your project should have **implementation.h** and **implementation.c** files with all required functions (list of such functions further in the text).
  implementation.h file should include file with safe types typedefs: uint32_t, uint16_t, uint8_t atc, or such typedefs should be in the library itself.
  
1. create implementation.h file with all desired functions in your project if not already existed;
2. add this library to your project;
3. include nrf24l01p_imp.h file into your project;
4. declare NRF24L01+ object (further in the text);
5. write your code using this library (see examples further in the text).

_If you found some problems in the code, or just don't like how stuff organized and explained, fill free to open issues and pull requests._

### Functions Required by the Library

The library allows you to have multiple nrf24l01+ devices connected to a single MCU. For that nrf24l01+ is described as a struct. Each struct object represents its own nrf24l01+ device.

**Every object requres 5 functions:**

Names of the functions and of the parameters could be different, but the amount of inputs and both input and return types should be the same.

1. **uint8_t spi_write_function(uin8_t byte_to_write)** - SPI write function. Should take one byte of data and send it through desired SPI (whatever hardware or software SPI you use to exchange data with nrf24l01+) in a single byte mode (this is important cause new devices support 2-byte SPI transmission). Should return byte of data that was received during send operation of var1; 
2. **void ce_high_function(void)** - Should set MCU/MPU pin connected to the **CE** pin of particular NRF24L01+ device logic high;
3. **void ce_low_function(void)** - Should set MCU/MPU pin connected to the **CE** pin of particular NRF24L01+ device logic LOW;
2. **void csn_high_function(void)** - Should set MCU/MPU pin connected to the **CSN** pin of particular NRF24L01+ device logic high;
2. **void csn_low_function(void)** - Should set MCU/MPU pin connected to the **CSN** pin of particular NRF24L01+ device logic LOW.

It is possible to control multiple nrf24l01+ devices with the same SPI (so the same spi_write function) if at least CSN pins of nrf24l01+ devices are connected to different MCU/MPU pins (so there should be different csn_low and csn_high functions). If devices, that are controlled with the same spi_write function should work in different modes (RX or TX), it is recommended to also connect CE pins of nrf24l01+ devices to different MCU/MPU pins.

### NRF24l01+ instance declaration

#### nrf24l01p struct fields
- _csn_high_ - pointer to the **void csn_high(void)** function;
- _csn_low_ - pointer to the **void csn_low(void)** function;
- _ce_high_ - pointer to the **void ce_high(void)** function;
- _ce_low_ - pointer to the **void ce_low(void)** function;
- _spi_write_byte_ - pointer to the **uint8_t spi_write_function(uin8_t byte_to_write)** function;
- _payload_size_in_bytes_ - payload size in bytes. Must be in range on 0 to 32. Must be the same at both transmitter and receiver;
- _frequency_channel_ - number of the frequency channel that nrf24l01+ will use to transmit data. The frequency step is 1 Mhz. Must be in range of 1 to 124 (carrier frequency will be 2.401- 2.524  GHz). Must be the same at both transmitter and receiver;
- _power_outout_ - power output of the device. Must be the same at both transmitter and receiver. Must have one of the following values:
  - nrf24_pa_min - 0 dbm RF output power, 11.3 mA dc current consumption;
  - nrf24_pa_low - -6 dbm RF output power, 9.0 mA dc current consumption;
  - nrf24_pa_high - -12 dbm RF output power, 7.5 mA dc current consumption;
  - nrf24_pa_max - -18 dbm RF output power, 7.0 mA dc current consumption;
- _data_rate_ - data rate of the device. Must be the same at both transmitter and receiver. Must have one of the following values:
  - nrf24_250_kbps - 250 KBps;
  - nrf24_1_mbps - 1 MBps;
  - nrf24_2_mbps - 2 MBps;
- _device_was_initialized_ - internal variable, used for error checking. Must be initialized with 0. Must not be changed by the user program.



#### Declaration in global variable space 

```C
// @file main.c 
#include "nrf24l01p.h"

// Some initialization code

nrf24l01p example_nrf24 = {.device_was_initialized = 0};

int main()
{

	example_nrf24.csn_high = gpiob1_high;
	example_nrf24.csn_low = gpiob1_low;
	example_nrf24.ce_high = gpiob0_high;
	example_nrf24.ce_low = gpiob0_low;
  example_nrf24.spi_write_byte = spi1_write_single_byte;
	example_nrf24.frequency_channel = 45;
	example_nrf24.payload_size_in_bytes = 12;
	example_nrf24.power_output = nrf24_pa_high;
	example_nrf24.data_rate = nrf24_1_mbps;

// Other project code
}

```

## Examples


## What is not implemented
- received power detection (RPD) reading and handling;
- only addresses with a length of 5 bytes are allowed. Addresses with 3 and 4 bytes are not allowed;
- library is not compatible with nrf24l01;
- payload reusing in RX mode;
- all devices are configured to use 2 byte CRC;
- dynamic payload size and sending data with ack are not implemented.
