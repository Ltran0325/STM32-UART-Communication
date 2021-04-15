# STM32-UART-Communication

Communicate between microcontroller and PC using UART. With and without HAL driver library.

## Universal Asynchronous Reciever-Transmitter (UART):

Unlike SPI which is a communication protocol, the UART is a physical circuit inside the STM32 microcontroller. UART allows for asynchronous communication between two devices using two wires. In this project, we cover UART via polling, interrupt, DMA, and DMA RTOS.

![image](https://user-images.githubusercontent.com/62213019/114249000-214b1680-994e-11eb-86c1-71296ad0ecb9.png)

Source: Scott Campbell -- https://www.circuitbasics.com/basics-uart-communication/

Data from the UART is sent and recieved as a packet.

## STM32CubeMX (Initialization Code Generator GUI):


<img src="https://user-images.githubusercontent.com/62213019/114250042-3a08fb80-9951-11eb-89cf-6784db620426.png" width="624" height="351">

In STM32CubeMX, enable USART2. Set buad rate to 9600 bit/s, 8 data bits, no parity bit, and 1 stop bit.

## UART Interrupt Method Without HAL UART Module Driver:

### 1) Recieve and return message via UART interrupt.


```c
#define MAX 6

uint8_t message[MAX];
uint8_t Tx_count = 0;
uint8_t Rx_count = 0;

int main(void){

	HAL_Init();
	SystemClock_Config();
	MX_GPIO_Init();
	MX_USART2_UART_Init();

	while(1){

	}

}
```

### 2) USART2 initialization. Enable USART2 recieve interrupt.

```c
static void MX_USART2_UART_Init(void)
{

  /* USER CODE BEGIN USART2_Init 0 */

  /* USER CODE END USART2_Init 0 */

  /* USER CODE BEGIN USART2_Init 1 */

  /* USER CODE END USART2_Init 1 */
  huart2.Instance = USART2;
  huart2.Init.BaudRate = 9600;
  huart2.Init.WordLength = UART_WORDLENGTH_8B;
  huart2.Init.StopBits = UART_STOPBITS_1;
  huart2.Init.Parity = UART_PARITY_NONE;
  huart2.Init.Mode = UART_MODE_TX_RX;
  huart2.Init.HwFlowCtl = UART_HWCONTROL_NONE;
  huart2.Init.OverSampling = UART_OVERSAMPLING_16;
  huart2.Init.OneBitSampling = UART_ONE_BIT_SAMPLE_DISABLE;
  huart2.AdvancedInit.AdvFeatureInit = UART_ADVFEATURE_NO_INIT;
  if (HAL_UART_Init(&huart2) != HAL_OK)
  {
    Error_Handler();
  }
  /* USER CODE BEGIN USART2_Init 2 */
  USART2->CR1 |= USART_CR1_RXNEIE; // enable receive interrupt
  /* USER CODE END USART2_Init 2 */

}
```

### 3) Program USART2 interrupt request handler. Disable USART2_IRQHandler when finished. 

```c
void USART2_IRQHandler(void)
{
	// if receive buffer full
	if( USART2->ISR & USART_ISR_RXNE ){

		// move byte from buffer to message
		if(Rx_count < MAX){
		receive[Rx_count] = USART2->RDR & 0x0FF; // reading RDR clears RXNE flag
		Rx_count++;
		}

	}

	// if byte ready to transfer
	if(USART2->ISR & USART_ISR_TC){

		// send  byte to TDR
		if(Tx_count < MAX){
			USART2->TDR = transmit[Tx_count]; // write to TDR (clears TC bit)
			Tx_count++;
		}

	}

}
```

### 4) Run the program. 

Board transmits "Hello!" and recieves "Return".

![image](https://user-images.githubusercontent.com/62213019/114915969-1f170b00-9dd9-11eb-9923-ca81e076e8c4.png)




## UART With HAL UART Module Driver:

### Polling Method:

The simplest but least efficient method for UART. Polling blocks the CPU until the UART is finished recieving or transmitting data.

Polling method code:
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

<img src="https://user-images.githubusercontent.com/62213019/114441498-53dc5580-9b80-11eb-8b8b-1e4032788eed.png" width="468" height="263">

Interrupt method code:
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
Completion of data transmit is handled in similar fashion.

### DMA Method:

The DMA method is the most efficient method for saving CPU cycles. This method offloads the data transfer process to the device's DMA controller instead of the CPU. The DMA is especially useful when data is transferred in quick bursts which would otherwise be handled by rapid interrupt calls.

 ![image](https://user-images.githubusercontent.com/62213019/114440221-baf90a80-9b7e-11eb-8a0e-417cfbf72be0.png)

Source: Yasen Stoyanov -- https://open4tech.com/direct-memory-access-dma-in-embedded-systems/

To configure the DMA, enable USART2_RX and USART2_TX in STMCubeMX

<img src="https://user-images.githubusercontent.com/62213019/114440955-a49f7e80-9b7f-11eb-99c6-1ac076f23d09.png" width="468" height="263">

DMA method code:

```c
  HAL_UART_Receive_DMA(&huart2, RX_Buffer, sizeof(RX_Buffer));   // Recieve data from PC
  HAL_UART_Transmit_DMA(&huart2, TX_Buffer, sizeof(TX_Buffer));  // Transmit data to PC
 ```

