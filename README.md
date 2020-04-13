# nrf24l01p_imp
NRF24L01+ basic implementation-independent library. Implementation-independent (impi for short) means that this library can be used on any device from any manufacturer: STM, TI, NXP, Atmel with whatever architecture MCU or MPU uses. To work like that impi-libraries require your project to some basic functions in a specific way. This is a pretty complicated library to understand at first, but bear with us because after reading this instruction we hope that you'll understand how powerful is this approach.

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

it is possimbe 

обязательно надо подключить библиотеку implementation в которой должен быть подключен файл, который определеяет Си типы данных: uint32_t, uin16_t, Uint8_t и так далее


### NRF24l01+ instance declaration

## Examples


## What is not implemented
- received power detection (RPD) reading and handling;
- only addresses with a length of 5 bytes are allowed. Addresses with 3 and 4 bytes are not allowed;
- library is not compatible with nrf24l01;
- payload reusing in RX mode;
- all devices are configured to use 2 byte CRC;
- dynamic payload size and sending data with ack are not implemented.
