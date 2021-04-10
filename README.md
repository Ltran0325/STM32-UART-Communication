# STM32-UART-Communication

Communicate between microcontroller and PC using UART.

## Table of Contents

- [Universal Asynchronous Reciever-Trasnmitter (UART)](#UART)
- [STM32CubeMX (Initialization Code Generator GUI)](#STM32CubeMX)
- [Polling Method](#Polling Method)
- [Interrupt Method](#Interrupt Method)
- [DMA Method](#DMA Method)

### Universal Asynchronous Reciever-Trasnmitter (UART)

Unlike, SPI which is a communication protocol, the UART is a physical circuit inside the STM32 microcontroller. UART allows for asynchronous communication between two devices using two wires. In this project, we cover three methods of UART: polling, interrupt, and DMA. 

![image](https://user-images.githubusercontent.com/62213019/114249000-214b1680-994e-11eb-86c1-71296ad0ecb9.png)

Source: Scott Campbell -- https://www.circuitbasics.com/basics-uart-communication/

Data from the UART is sent and recieved as a packet.

Example:

![image](https://user-images.githubusercontent.com/62213019/114283937-6aa86e00-9a01-11eb-8843-d9c3c9f23a1a.png)

Data "Hello World" is transmitted from Nucleo-board to PC using UART interrupt method. PC returns "Interrupt!" to Nucleo-board which is captured inside the RX_Buffer. Hercules SETUP is used to handle UART on the PC side.

### STM32CubeMX (Initialization Code Generator GUI)


<img src="https://user-images.githubusercontent.com/62213019/114250042-3a08fb80-9951-11eb-89cf-6784db620426.png" width="624" height="351">

In STM32CubeMX, enable USART2. Set buad rate to 9600 bit/s, 8 data bits, no parity bit, and 1 stop bit.

### Polling Method:

The simplest but least efficient method for UART. Polling blocks the CPU until the UART is finished recieving or transmitting data.

Polling Code:
```c
  uint8_t TX_Buffer[] = "Hello World!\r\n";
  
  uint8_t RX_Buffer[8];

  HAL_UART_Transmit(&huart2,TX_Buffer,sizeof(TX_Buffer),1000); // "Hello World!"
  
  HAL_UART_Receive(&huart2, RX_Buffer, 8, 5000);               // "Goodbye!"
```
  
Purpose:

MCU sends "Hello World!" to PC. PC returns "Goodbye!" to MCU. Hercules SETUP is used to handle PC UART.
 
Both HAL polling transmit and recieve functions use similar arguments. First, address variable &huart2 is the UART handle for our enabled USART2. Then, the buffer, size of the buffer in bytes, and the allowed blocking time of the function in ms. 

To further understand these two functions read the STM32F4 HAL User Manual: https://www.st.com/resource/en/user_manual/dm00105879-description-of-stm32f4-hal-and-ll-drivers-stmicroelectronics.pdf. Or, highlight the function inside STM32Cube IDE and right-click to open-declaration. which will bring you to the UART HAL module driver.



### Interrupt Method:

The interrupt method is non-blocking, meaning that recieve/transmit completion will be indicated through interrupt. 
Since we are using interrupts, the NVIC must be configured.

<img src="https://user-images.githubusercontent.com/62213019/114283354-2bc4e900-99fe-11eb-8480-441aa23e4fa2.png" width="468" height="263">

Interrupt Code:
```c
  HAL_UART_Receive_IT(&huart2, RX_Buffer, sizeof(RX_Buffer));   // Recieve data from PC
  HAL_UART_Transmit_IT(&huart2, TX_Buffer, sizeof(TX_Buffer));  // Transmit data to PC
 ```
When data the size of RX_Buffer is recieved via UART, the HAL_UART_RxCpltCallback interrupt is called by the HAL_UART_IRQ_Handler.
```c
  void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
  {
	  __NOP(); // for debugging
  }
```
Completion of data transfer is handled in similar fashion.

### DMA Method:

-In Progress-
