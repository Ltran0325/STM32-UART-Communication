# STM32-UART-Communication

Communicate between microcontroller and PC using UART.

### Universal Asynchronous Reciever-Trasnmitter (UART)

Unlike, SPI which is a communication protocol, the UART is a physical circuit inside the STM32 microcontroller. UART allows for asynchronous communication between two devices using two wires. In this project, we cover three methods of UART: polling, interrupt, and DMA. 

![image](https://user-images.githubusercontent.com/62213019/114249000-214b1680-994e-11eb-86c1-71296ad0ecb9.png)

Source: Scott Campbell -- https://www.circuitbasics.com/basics-uart-communication/

Data from the UART is sent and recieved as a packet.

### STM32CubeMX


<img src="https://user-images.githubusercontent.com/62213019/114250042-3a08fb80-9951-11eb-89cf-6784db620426.png" width="624" height="351">

In STM32CubeMX, enable USART2. Set buad rate to 9600 bit/s, 8 data bits, no parity bit, and 1 stop bit.

### Polling:

The simplest but least efficient method for UART. Polling blocks the CPU until the UART is finished recieving or transmitting data.

Polling Code:
  
  uint8_t TX_Buffer[] = "Hello World!\r\n";
  
  uint8_t RX_Buffer[8];

  HAL_UART_Transmit(&huart2,TX_Buffer,sizeof(TX_Buffer),1000); // "Hello World!"
  
  HAL_UART_Receive(&huart2, RX_Buffer, 8, 5000);               // "Goodbye!"
 
Both HAL polling transmit and recieve functions use similar arguments. First, address variable &huart2 is the UART handle for our enabled USART2. Then, the buffer, size of the buffer in bytes, and the allowed blocking time of the function in ms. 

To further understand these two functions read the STM32F4 HAL User Manual: https://www.st.com/resource/en/user_manual/dm00105879-description-of-stm32f4-hal-and-ll-drivers-stmicroelectronics.pdf. Or, highlight the function inside STM32Cube IDE and right-click to open-declaration. which will bring you to the UART HAL module driver.


