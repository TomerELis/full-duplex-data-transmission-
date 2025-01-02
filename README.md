# full-duplex-data-transmission-

This project implements a USART (Universal Synchronous and Asynchronous Receiver-Transmitter) communication protocol using Arduino. It simulates data transmission (TX) and reception (RX) between devices with parity checking and error handling.


# How It Works
The USART communication in this project is implemented through two main processes: transmitting (TX) and receiving (RX).

The transmitter (TX) starts by preparing the data to send. It begins with a start bit, followed by the data bits, a parity bit
and ends with a stop bit.
Each bit is sent sequentially, with precise timing controlled by millis().
The transmitter ensures that all bits are sent properly before transitioning back to idle.

The receiver (RX) continuously monitors the RX pin for a start bit. Once detected, it reads the data bits, validates each bit using majority voting logic, and verifies the parity bit to ensure data integrity. If the parity or any other bit does not match the expected pattern, the receiver transitions to an error state, resets necessary parameters, and waits for the next frame.

Both TX and RX processes handle timing using BIT_TIME to ensure synchronization between the transmitter and receiver. This guarantees that the data transmission and reception are both accurate and robust, even in the presence of timing variations.

# Link to a scheme
Part 1:
https://ibb.co/BLMhfGz
Part 2:
https://ibb.co/c8TCXDS
