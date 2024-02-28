# Introduction

First block of code shows UART using the abstracted C/C++ in Arduino Developement Environment.

```c
/*
* usart.c
* Created : 07/02/2024T15:13:00Z
* Author  : Sebastian Blair
*/

#define USART_BAUDRATE 9600 // Desired Baud Rate (USBRR)
char name[] = {'H','e','l','l','o',' ','w','o','r','l','d','!','\n'}; 

void setup() {
  Serial.begin(USART_BAUDRATE);
  // put your setup code here, to run once:

}

void loop() {
  // put your main code here, to run repeatedly:
    for(int i = 0; i < 13;i++){
      Serial.print(name[i]);
    }
		delay(1000);
}
```

-----


In the code block below is the embedded non-abstracted approach: 


```c
/*
* usart.c
* Created : 07/02/2024T15:13:00Z
* Author  : Sebastian Blair
*/

#define F_OSC 16000000UL // Defining the systems oscillator's frequency 16MHz

#include <avr/io.h>      // Contains all the I/O Register Macros
#include <util/delay.h>  // Generates a Blocking Delay

#define USART_BAUDRATE 9600 // Desired Baud Rate (USBRR)

// Provides the Baud Rate Register for (UBRRn) for 9600 USBRR needs to be 103, we use 16x oversampling for better accuracy
// refer to https://onlinedocs.microchip.com/pr/GUID-ED37252C-1496-4275-BAEF-5152050ED2C2-en-US-2/index.html?GUID-65A53416-8DE2-468E-99CF-09F480AEF734
#define BAUD_PRESCALER (((F_OSC / (USART_BAUDRATE * 16UL))) - 1) 

// Can have Synchronous or Asynchronous
#define ASYNCHRONOUS (0<<UMSEL00) // USART Mode Selection
#define SYNCHRONOUS (1<<UMSEL00) // USART Mode Selection

// USART parity mode (UPM)
#define DISABLED    (0<<UPM00) 
#define EVEN_PARITY (2<<UPM00)
#define ODD_PARITY  (3<<UPM00)
#define PARITY_MODE  DISABLED // USART Parity Bit Selection


// USART stop bit  (USB) 
#define ONE_BIT (0<<USBS0) 
#define TWO_BIT (1<<USBS0)
#define STOP_BIT ONE_BIT      // USART Stop Bit Selection

// USART character size (UCS) buts select the number of data bits in the frame
#define FIVE_BIT  (0<<UCSZ00) 
#define SIX_BIT   (1<<UCSZ00)
#define SEVEN_BIT (2<<UCSZ00)
#define EIGHT_BIT (3<<UCSZ00)
#define DATA_BIT   EIGHT_BIT  // USART Data Bit Selection

char name[] = {'H','e','l','l','o',' ','w','o','r','l','d','!','\n'}; 

void USART_Init()
{
	// Set Baud Rate register (H)igh and (L)ow
	UBRR0H = BAUD_PRESCALER >> 8; // shift right by 8 bits so 103 would be 0 
	UBRR0L = BAUD_PRESCALER; // here would 1101010
	
	// Set Frame Format ((U)SART (C)ontrol and (S)tatus (R)egister C) selects async or sync
	UCSR0C = ASYNCHRONOUS | PARITY_MODE | STOP_BIT | DATA_BIT;
	
	// Enable Receiver and Transmitter RX/TXEN Enable... (U)SART (C)ontrol and (S)tatus (R)egister B
	UCSR0B = (1<<RXEN0) | (1<<TXEN0);
}

void USART_TransmitPolling(uint8_t DataByte)
{
  // (U)SART (D)ata (R)egister (E)mpty if 0 empty, 1 has data
	while (( UCSR0A & (1<<UDRE0)) == 0) {}; // Do nothing until UDR is ready
  // Where your byte of data goes 
	UDR0 = DataByte;
}

int main()
{
	USART_Init();
	while (1)
	{
    for(int i = 0; i < 13;i++){
      USART_TransmitPolling(name[i]);
    }
		_delay_ms(1000);
	}
	return 0;
}
```
