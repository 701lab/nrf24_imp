# nrf24l01p_imp
NRF24L01+ implementation-independent library.

[What are implementation-independent libraries?] - (add a link to explanation)

## How to use 
1. your project should have implementation.h and implementation.c files with all required functions (list of such functions further in the text);
2. add this library to your project;
3. #include nrf24l01p_imp.h into your project;
4. declare NRF24L01+ object (further in the text);
5. write your code using this library.

If you found some problems int the code, or just don't like how stuff organized and explained, fill free to open issues and pull requests.

### Functions Required by the Library

### NRF24l01+ instance declaration

## Examples


## What is not implemented
- received power detection (RPD) reading and handling;
- only addresses with a length of 5 bytes are allowed. Addresses with 3 and 4 bytes are not allowed;
