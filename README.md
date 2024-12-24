# full-duplex-data-transmission-

This project implements a USART (Universal Synchronous and Asynchronous Receiver-Transmitter) communication protocol using Arduino. It simulates data transmission (TX) and reception (RX) between devices with parity checking and error handling.


# How It Works
TX Process:

Sends start, data, parity, and stop bits sequentially.
Includes bit timing using millis() for precise transmission intervals.
RX Process:

Reads incoming bits and validates them.
Checks start bit, data bits, parity, and stop bit for correctness.
Error States:

Handles errors in bit reading or timing, ensuring robustness in communication.
