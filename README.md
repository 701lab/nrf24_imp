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

An example of the initialization of the struct. CE is on PB0, CSN is on PB1, SPI1 is used for communication (STM32):

```C
// @file main.c 
#include "nrf24l01p.h"

uint8_t spi1_write_single_byte(uint8_t byte_to_be_sent);
void gpiob1_high(void); // PB1 connected to CSN
void gpiob1_low(void);
void gpiob0_high(void); // PB0 connected to CE
void gpiob0_low(void);

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

## Workflows

In all following examples it is considered, that struct **nrf24l01p.h** file is included and **example_nrf24** struct is initialized. To understand more about functions and parameters look into nrf24l01p.c file.

### Basic TX mode

```C
// @file main.c 
#include "nrf24l01p.h"

// Some initialization code

uint32_t array_to_send[3] = {1, 2, 3};

uint8_t tx_address[5] = {0x11, 0x22, 0x33, 0x44, 0x55}; // Must be 5 bytes long

int main()
{
// ... - example_nrf24 struct and other initalizations

// Must be called for every nrf24l01+ instance before other initializations.
nrf24_basic_init(&example_nrf24); 	

// This function is called to set new tx addres. Address must be the same as one of RX addresses on receiver.
nrf24_set_tx_address(&example_nrf24, new_addr_for_nrf_tx);  

// Must be called for any nrf24l01+ instance to enable tx mode.
nrf24_tx_mode(&example_nrf24); 		

// Other project code

	while(1)
	{
		// ... some code 

		// When it is time to send data
		nrf24_send_message(&example_nrf24, array_to_send, sizeof(array_to_send), 1); 
	}
}

```

### Basic RX mode

```C
// @file main.c 
#include "nrf24l01p.h"

// Some initialization code

uint32_t array_to_receive[3] = {0, 0, 0};

uint8_t pipe1_rx_address[5] = {0x11, 0x22, 0x33, 0x44, 0x55}; // Must be 5 bytes long
uint8_t pip3_address = 0x77;
uin8_t pip5_address = 0x99;

int main()
{
// ... - example_nrf24 struct and other initalizations

// Must be called for every nrf24l01+ instance before other initializations.
nrf24_basic_init(&example_nrf24); 	

// This function should be called even if not pipe 1 is used to receive data because 4 first bytes of addresses of all pipes
// are the same as pip1 and must set up through pipe1. Only last byte of every pipe rx address must be different.
nrf24_enable_pipe1(&example_nrf24, pipe1_rx_address);	

// Can be called to enable pipe 2-5 if needed. Only last byte of the address is used as input. Pther 4 are the same as of pipe 1.
nrf24_enable_pipe2_5(&example_nrf24, 3, pip3_address);
nrf24_enable_pipe2_5(&example_nrf24, 5, pip5_address);
							 
// Must be called for any nrf24l01+ instance to enable rx mode.						 
nrf24_rx_mode(&example_nrf24); 		

// Other project code

	while(1)
	{
	// ... some code 
	
		// To receive data we should check if anything new is available.
		if(nrf24_is_new_data_availiable(&example_nrf24))
		{
			// Read new data
			nrf24_read_message(&example_nrf24, array_to_receive, sizeof(array_to_receive));
			// ... Do something with new data
		}
		
	}
}

```

### Using both RX and TX on a single device

```C
// @file main.c 
#include "nrf24l01p.h"

// Some initialization code

uint32_t array_to_receive[3] = {0, 0, 0};
uint32_t array_to_send[3] = {1, 2, 3};

uint8_t tx_address[5] = {0x55, 0x44, 0x33, 0x22, 0x11}; // Must be 5 bytes long
uint8_t pipe1_rx_address[5] = {0x11, 0x22, 0x33, 0x44, 0x55}; // Must be 5 bytes long. 
uint8_t pip3_address = 0x77;

int main()
{
// ... - example_nrf24 struct and other initalizations

// Must be called for every nrf24l01+ instance before other initializations.
nrf24_basic_init(&example_nrf24); 	

// Set RX and TX addresses
nrf24_enable_pipe1(&example_nrf24, pipe1_rx_address);	
nrf24_enable_pipe2_5(&example_nrf24, 3, pip3_address);
nrf24_set_tx_address(&example_nrf24, new_addr_for_nrf_tx);  
			
// In two-sided mode it is better to keep nrf24l01 + in receive mode all the time when it should not transmit data.
nrf24_rx_mode(&example_nrf24); 		

// Other project code

	while(1)
	{
	// ... some code 
	
		// Receive data if availiable
		if(nrf24_is_new_data_availiable(&example_nrf24))
		{
			// Read new data
			nrf24_read_message(&example_nrf24, array_to_receive, sizeof(array_to_receive));
			// ... Do something with new data
		}
		
		// When it is time to transmit message
		nrf24_tx_mode(&example_nrf24); // Switch to TX mode.
		nrf24_send_message(&example_nrf24, array_to_send, sizeof(array_to_send), 1); 
		
		// Probably it is better to make a small delay cause device needs some time to get into TX mode and send data.
		
		nrf24_rx_mode(&example_nrf24); // Get back to RX mode.
	}
}

```

## A little bit more useful information

### Interrupt handling

NRF24l01+ has 3 interrupt sources:
- Rt interrupt (called when new message received);
- TX interrupt (called when the message is successfully transmitted);
- MAX_RT interrupt (called when the message was retransmitted maximum amount of times without acknowledgment).

By default library disables all interrupts on the IRQ pin. To enable interrupts you should call:

```C
//... some code

nrf24_enable_interrupts(&example_nrf24, 1, 1, 0);

//... also some code 
```

Where each parameter describes the corresponding interrupts state with respect to the list above. "1" means that particular interrupt will be enabled, "0" means that particular interrupt will be disabled.

The programmer should implement an external interrupt handler on the pin connected to IRQ of nrf24l01+. The interrupt is signalized only by the pin high to low transition. So low to high transition on the IRQ pin should not be interpreted as an interrupt.

To understand what type of interrupt occurred in interrupt handler nrf24_get_interrupts_status function should be called:

```C
void my_interrupt_nadler()
{
	uint8_t interrupt_state = nrf24_get_interrupts_status(&example_nrf24);

	// Some type of handling (reading from, or writing to the device atc).
}

```
This function returns STATUS register with only interrupts-related bits masked. This means that 1 in bits 4-6 will mean that either MAX_RT, TRX_DR or RX_DR interrupt occurred respectively an all other bits will always be 0. So return value can be - 0x00 if no interrupts occurred and from 0x10 to 0x70 if any interrupts occurred. After reading interrupts states this functions clears register. So immediate second call will for sure return 0x00 as no interrupts occurred since the last call of the same function.

## Error handling

Most of the functions in the library return error codes if any error occured or 0 otherwise. All return codes are stored in **nrf24l01p_mistakes.h file**. Also programmer can change mistakes offset to the desired bye defining NRF24L01P_MISTAAKES_OFFSET before #include "nrf24l01p.h.

The two only function that return something other than mistakes codes are: nrf24_is_new_data_availiable (returns number of pipe from which last recieved message was received or 0 if no messages are in RX FIFO), nrf24_get_interrupts_status (returns interrupts statuses).

With all other functions, it is recommended to check for mistakes. For example, if nrf24_basic_init returns mistake device probably isn't set up and won't work. In general, you can have some type of mistakes array that contains all mistakes codes, that occurred during runtime and a function to write to such an array, for example, **void add_to_mistakes_log(uint32_t mistake-code)**. Then your code can look something like that:

```C
int main()
{
	//... Some setups
	add_to_mistakes_log(nrf24_basic_init(&example_nrf24));
	add_to_mistakes_log(nrf24_enable_pipe1(&example_nrf24, new_addr_for_nrf));
	add_to_mistakes_log(nrf24_rx_mode(&example_nrf24));
	//... Some setups
	
	while(1)
	{
		//... Some code 
		add_to_mistakes_log(nrf24_send_message(&example_nrf24, nrf_data, 12, yes));
	}
}
```

After that, you will be able to debug your device in a more controlled way.

## What is not implemented
- received power detection (RPD) reading and handling;
- only addresses with a length of 5 bytes are allowed. Addresses with 3 and 4 bytes are not allowed;
- library is not compatible with nrf24l01;
- payload reusing in RX mode;
- all devices are configured to use 2 byte CRC;
- dynamic payload size and sending data with ack are not implemented.

**I hope now you know how to use this library. One more time. If you have any problems with the library feel free to use issues and pull requests to ask questions suggest features and make fixes**
