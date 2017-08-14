# stm32f4_usart_interrupt

## プログラムの概要

STM32F4 DiscoveryのUSARTを割込みを使って動かす。文字列を送信しながら、文字列を受信する。
[ブログ記事](http://wp.me/p7Mkw3-Yz) ブログ記事と連動しています。


## 開発環境

Ubuntu 16.0.4 LTS
STM32CubeMX Version 4.20.0
System Workbench for STM32 (Eclipse Mars Release 4.5.2)
STM32F4 Discovery
FTDI USBシリアル変換アダプター

## 開発の手順

STM32CubeMXを使ってピンアウト、クロックコンフィグレーション、USARTパラメーター、USART NVICの設定を行い、System Workbench for STM32用のプロジェクトを生成する。
USART1のRXをPB7にTXをPB6にそれぞれアサインした。

生成されたプロジェクトをSystem Workbench for STM32で開き、main.cを以下の箇所を編集する。

 変数の宣言

	/* USER CODE BEGIN PV */
	/* Private variables ---------------------------------------------------------*/
	char * bufferTx = "Hello!\n\r";
	uint8_t bufferRx[5];
	/* USER CODE END PV */

main関数

	int main(void)
	{
	
	  /* USER CODE BEGIN 1 */
	
	  /* USER CODE END 1 */
	
	  /* MCU Configuration----------------------------------------------------------*/
	
	  /* Reset of all peripherals, Initializes the Flash interface and the Systick. */
	  HAL_Init();
	
	  /* Configure the system clock */
	  SystemClock_Config();
	
	  /* Initialize all configured peripherals */
	  MX_GPIO_Init();
	  MX_USART1_UART_Init();
	
	  /* USER CODE BEGIN 2 */
	  __HAL_UART_ENABLE_IT(&huart1, UART_IT_RXNE);
	  __HAL_UART_ENABLE_IT(&huart1, UART_IT_TC);
	  /* USER CODE END 2 */
	
	  /* Infinite loop */
	  /* USER CODE BEGIN WHILE */
	  while (1)
	  {
		  HAL_Delay(200);
	  /* USER CODE END WHILE */
	
	  /* USER CODE BEGIN 3 */
	
	  }
	  /* USER CODE END 3 */
	
	}
	
クロック設定（CubeMXが生成するコードがおかしいので修正する）

	void SystemClock_Config(void)
	{
	....
	....
	    /**Initializes the CPU, AHB and APB busses clocks 
	    */
	  RCC_OscInitStruct.OscillatorType = RCC_OSCILLATORTYPE_HSE;
	....
	....
	}
	

stm32f4xx_it.c （割込み関係の関数群が生成されている）を開きUSART1_IRQHandler()関数を編集。

	void USART1_IRQHandler(void)
	{
	  /* USER CODE BEGIN USART1_IRQn 0 */
	
	  /* USER CODE END USART1_IRQn 0 */
	  HAL_UART_IRQHandler(&huart1);
	  /* USER CODE BEGIN USART1_IRQn 1 */
	  HAL_UART_Receive_IT(&huart1, bufferRx, 5);
	  HAL_UART_Transmit_IT(&huart1, (uint8_t *)bufferTx, 8);
	
	  /* USER CODE END USART1_IRQn 1 */
	}
	
	
## I/Oモード
UARTの非同期通信および半二重通信には２つの転送モードがある。

### ブロッキングモード
転送が終了した時点で関数がステータスを返す

### ノンブロッキングモード
通信は割込みかDMAを使って行われる。データ転送の終了はUART IRQを通じて指示される。ユーザーコールバック関数が転送終了時に呼び出される。

### ブロッキングモードで使用するデータ転送関数
	HAL_UART_Transmit()
	HAL_UART_Receive()

###  割込みを使ったノンブロッキングモードで使用するデータ転送関数
	HAL_UART_Transmit_IT()
	HAL_UART_Receive_IT()
	HAL_UART_IRQHandler()

### DMAを使ったノンブロッキングモードで使用するデータ転送関数
	HAL_UART_Transmit_DMA()
	HAL_UART_Receive_DMA()

### ノンブロッキングモードで呼び出されるコールバック関数
	HAL_UART_TxCpltCallback()
	HAL_UART_RxCpltCallback()
	HAL_UART_ErrorCallback()

### UARTの割込みを有効化する
	__HAL_UART_ENABLE_IT()
引数は以下のいずれか。

	 UART_IT_CTS: CTS change interrupt (not available for UART4 and UART5)
	 UART_IT_LBD: LIN Break detection interrupt
	 UART_IT_TXE: Transmit Data Register empty interrupt
	 UART_IT_TC:  Transmission complete interrupt
	 UART_IT_RXNE: Receive Data register not empty interrupt
	 UART_IT_IDLE: Idle line detection interrupt
	 USART_IT_ERR: Error interrupt
	