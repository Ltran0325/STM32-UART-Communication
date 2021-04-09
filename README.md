# STM32-UART-Communication

Communicate between microcontroller and PC using UART.

### Universal Asynchronous Reciever-Trasnmitter (UART)

Unlike, SPI which is a communication protocol, the UART is a physical circuit inside the STM32 microcontroller. UART allows for asynchronous communication between two devices using two wires. In this project, we cover three methods of UART: polling, interrupt, and DMA. 

![image](https://user-images.githubusercontent.com/62213019/114249000-214b1680-994e-11eb-86c1-71296ad0ecb9.png)

Source: Scott Campbell -- https://www.circuitbasics.com/basics-uart-communication/

Data from the UART is sent and recieved as a packet.

### Polling:

The simplest but least efficient method for UART. Polling blocks the CPU until the UART is finished recieving or transmitting data.
