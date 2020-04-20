# STK500-ATMEGA-MP3-

//Header file for Main.h
/****************************************************************************************************************************************
** Programmer: Cameron Burford
** Email: cburford@purdue.edu
** ECET 279, MP3 project
** Lab Section : Wednesday 7:30 am
** Program Description: This program is code for the mp3 and LCD. This code is suppose to make a mp3 with the STK600.
**
**	~Hardware~
**		~PORTB for LCD data~
**			B.0  LCD D0
**			B.1  LCD D1
**			B.2  LCD D2
**			B.3  LCD D3
**			B.4  LCD D4
**			B.5  LCD D5
**			B.6  LCD D6
**			B.7  LCD D7
**		~PORTA for LCD control~
**			A.7	LCD RS
**			A.6	LCD E
**			A.5	LCD R/!W
**		~PORTE for Tx/Rx to the MP3/TTYL~
**			E.0 Tx
**			E.1 Rx
**		~PORTL for LEDs
**			L.0	LED0
**			L.1	LED1
**			L.2	LED2
**			L.3 LED3
**		~PORTC for push buttons
**			C.0 SW0
**			C.1 SW1
**			C.2 SW2
**			C.3 SW3
*********************************************************************************************************************************************/


#ifndef MAIN_H_		//#include macro guard
#define MAIN_H_		//to ensure idempotence

//#includes
#include <avr/io.h>
#include <util/delay.h>

#include "Ports.h"
#include "LCD.h"
#include "UART.h"



#endif /* MAIN_H_ */

//Source code for Main.c
/****************************************************************************************************************************************
** Programmer: Cameron Burford
** Program Description: This program is code for the mp3 and LCD. This code is suppose to make a mp3 with the STK500.
**
**	~Hardware~
**		~PORTB for LCD data~
**			B.0  LCD D0
**			B.1  LCD D1
**			B.2  LCD D2
**			B.3  LCD D3
**			B.4  LCD D4
**			B.5  LCD D5
**			B.6  LCD D6
**			B.7  LCD D7
**		~PORTA for LCD control~
**			A.7	LCD RS
**			A.6	LCD E
**			A.5	LCD R/!W
**		~PORTE for Tx/Rx to the MP3/TTYL~
**			E.0 Tx
**			E.1 Rx
**		~PORTL for LEDs
**			L.0	LED0
**			L.1	LED1
**			L.2	LED2
**			L.3 LED3
**		~PORTC for push buttons
**			C.0 SW0
**			C.1 SW1
**			C.2 SW2
**			C.3 SW3
*********************************************************************************************************************************************/


#include "main.h"

void mp3string(volatile char *str);

int main(void)
{
	char play_pause[2][10] = {"|>","||"};
	char stop[] = {"[]"};
	char next[] = {">>"};
	char prev[] = {"<<"};
	char userprmpt[] = {"s0:|>/||s1:[]s2:>>s3:<<"};
	char mp3_play [10][30] = {"Preach.mp3","Slow_Motion.mp3","Must_Be_Nice.mp3","My_Boo.mp3","B.I.T.E.mp3","Charlene.mp3","Piece_Of_My_Love.mp3","Thim_Slick.mp3","Dont_Change.mp3","Slow_Jamz.mp3"};
	uint8_t pause1 = 1;
	uint8_t songlist = 0;
	uint8_t play_pause_command = 0;
	
	
	init_Ports();						//Initializing Ports
	init_LCD();						  //Initializing LCD
	init_UART();						//initializing UART									
	
	uart_out(0x0D);				          		//This is ASCII for carriage return
	_delay_ms(205);	
	
for(;;)						                  	//super loop, this equivalent to the while(1)
{
	LCD_write(INSTR_WR,0xC0);				   	//Clears the LCD
	LCD_string (userprmpt);		             //Prompt the use the pushbutton to play/pause/stop/next/previous
	
	LCD_write(INSTR_WR, 0x02);		      	//Display on the top row of the LCD
	LCD_string (mp3_play[songlist]);			//Display songs on the array


	LCD_write(INSTR_WR, 0xC0);				  	//Clears the LCD
	_delay_us(50);						          	//Delay
	
	//Play/Pause
	if ((PINC & 0b00000001) == 0)				//Push Button 1 is pressed
	{
		if (pause1 == 1)			          	//if pause1 is 1 then it goes the loop
		{
			LCD_write(INSTR_WR,0x01);	                	   //Display on the bottom of the LCD
			LCD_write(INSTR_WR, 0xC0);		            	   //Clears the LCD
			LCD_string (play_pause[play_pause_command]);   //Display whatever song in the array
			mp3string("PC F/");				                     //Play command to play the music
			mp3string (mp3_play[songlist]);		             //play that song in the array
			uart_out(0x0D);		                            	//This is ASCII for carriage return
			pause1 = 0;					                           //pause1 is equal to 0
			play_pause_command++;		                    	//play_pause_command is incremented
		}
		else if (pause1 == 0)		            	//pause1 is equal to 0 is goes in the loop
		{
			LCD_write(INSTR_WR,0x01);		        //Display on the bottom row of the LCD
			LCD_write(INSTR_WR, 0xC0);		     	//Clear the LCD
			mp3string("PC P/");	              	//Pause command to pause the music
			uart_out(0x0D);			              	//This is ASCII for carriage return
			
			if (play_pause_command == 1)	      //play_pause_command is - 1 it goes in this loop
			{
				
				LCD_write(INSTR_WR,0x01);	                        //Display on the bottom of the LCD
				LCD_write(INSTR_WR, 0xC0);        	               //Clears the LCD
				LCD_string (play_pause[play_pause_command]);       //play that song in the array
				play_pause_command--;	                          	//play_pause_command is decrement
			}
			else
			{
				LCD_write(INSTR_WR,0x01);	                        //displays on the bttom of the LCD
				LCD_write(INSTR_WR, 0xC0);		                    //Clears the LCD
				LCD_string (play_pause[play_pause_command]);     //play that song in the array
				play_pause_command++;		                         //play_pause_command is incremented
			}
		}
		
	}
	while ((PINC & 0b00000001) == 0);	  	//Wait until the push button is released
	
	//Stop
	if ((PINC & 0b00000010) == 0)		  		//Push Button 2 is pressed
	{
		play_pause_command = 0;			      	//play_pause_command is equal to 0
		pause1 = 1;					                //Pause1 is equal to 1
		LCD_write(INSTR_WR,0x01);			    	//Display on bottom of the LCD
		LCD_write(INSTR_WR, 0xC0);			  	//Clears The LCD
		LCD_string (stop);				          //Display stop on the LCD
		mp3string("PC S/");			          	//The stop command to stop the song
		mp3string (mp3_play[songlist]);			//Plays the songs that is in the array
		uart_out(0x0D);					            //This is ASCII for carriage return
	}
	while((PINC & 0b00000010) == 0);				//Wait until the push button is released
	
	
	//Next
	if ((PINC & 0b00000100) == 0)				//Push Button 3 is pressed
	{
		if (songlist == 9)			          //wont go to the past the 10th song
		{
			songlist = songlist;			    	//songlist = songlist
		} 
		else
		{
			songlist++;					//increment songlist
		}
		LCD_write(INSTR_WR,0x01);			         //Display on the bottom of the LCD
		LCD_write(INSTR_WR, 0xC0);		      		//Clear The screen
		LCD_string (next);					           //Display (next) >> on the LCD
		mp3string("PC F/");					           //Play command to play the music
		mp3string (mp3_play[songlist]);			   //Plays the songs that is in the array
		uart_out(0x0D);					               //This is ASCII for carriage return
		
	}
	while((PINC & 0b00000100) == 0);				//Wait until the push button is released
	
	//Previous
	if ((PINC & 0b00001000) == 0)				  	//Push Button 4 is pressed
	{
		if (songlist == 0)		              	//wont go to pass the first song in the array
		{
			songlist = songlist;					      //songlist = songlist
		}
		else
		{
			songlist--;					               //songlist is decrement
		}
		LCD_write(INSTR_WR,0x01);			       //Display on the bottom of the LCD
		LCD_write(INSTR_WR, 0xC0);			     //Clear the LCD
		LCD_string (prev);				           //Display previous on the LCD
		mp3string("PC F/");				           //Play command for the music
		mp3string (mp3_play[songlist]);			 //Plays the songs that is in the array
		uart_out(0x0D);					             //This is ASCII for carriage return
	}
	while((PINC & 0b00001000) == 0);			//Wait until the push button is released
	
}	//end of for loop
return(0);
}//end of main


void mp3string(volatile char *str)			//Mp3sring function to write a string
{
	while(*str!= 0)							//Wait until end of string
	{
		uart_out(*str);					//Output the character at that address
		str++;						    	//Goes to next character
	}
}

//Header file for Ports.h

#ifndef PORTS_H_		                 //#include macro guard
#define PORTS_H_		                 //to ensure idempotence

extern void init_Ports(void);      	//initializing Ports  

#endif /* PORTS_H_ */

//Source code for Ports.c

#include "main.h"

void init_Ports(void)				      //initializing the ports
{
	DDRA = 0b11100000;
	DDRB = 0b11111111;
	PORTB = 0b00000000;
	DDRC = 0b00000000;
	PORTC = 0b11111111;
	DDRL = 0b00001111;
	PORTL = 0b00000000;
}


//Header File for UART.h

#ifndef UART_H_			//#include macro guard
#define UART_H_			//to ensure idempotence

#define F_CPU 8000000UL			          //16mega Hz in the Frequency CPU
#define BAUD 9600				              //Baud Rate is 9600

#include <avr/interrupt.h>	

void init_UART(void);			           //initializing the UART function
void uart_out(uint8_t ch);		       //UART out function

extern volatile uint8_t rx_char;    //Define rx_char


#endif /* UART_H_ */

//Source file for UART.c

#include "main.h"
volatile uint8_t rx_char;

void init_UART(void)			 	//initializing the UART
{
	UCSR0A = 0;
	UCSR0B = 0b10011000;
	UCSR0C = 0b00000110;
	uint16_t myubr = ((F_CPU/(16UL*BAUD))-1);
	UBRR0L = myubr;
	UBRR0H = 0;
}

void uart_out(uint8_t ch)		      //UART out function
{
	while((UCSR0A&(1<<UDRE0))==0)
	{
		//wait for URE0 flag to indicate UDR0 ready for new data to transmit
	}
	UDR0 = ch;					      	//UDR0 equal to channel
}


ISR(USART0_RX_vect)					 //UART0 receive interrupt routine
{
	rx_char = UDR0;					   //rx_char is equal to UDR0
}

//Header file for LCD.h

#ifndef LCD_H_			//#include macro guard
#define LCD_H_			//to ensure idempotence

#define INSTR_WR 0
#define DATA_WR 1

#include <avr/io.h>

extern void init_LCD (void);		                                //initializing the LCD
extern void LCD_write (unsigned char RS, unsigned char data);		//Writing to the LCD function
void LCD_string(volatile char *str_ptr);		                    //LCD string function
extern void busy_check (void);			                          	//check busy function

#endif /* LCD_H_ */


//Source code for LCD.c

#include "main.h"

void init_LCD (void)			          	//initializing the LCD
{
	_delay_ms(35);					            // Wait for more than 30mS after VDD rises to 4.5V
	LCD_write(INSTR_WR,0x38);		        // Function set 8bits, 2line, display off
	_delay_us(50);				            	// Wait for more than 39microS
	LCD_write(INSTR_WR,0x0C);	        	// Display on, cursor off, blink off
	_delay_us(50);					            // Wait for more than 39microS
	LCD_write(INSTR_WR,0x01);		        // Display clear
	_delay_ms(2);				              	// Wait for more than 1.53mS
	LCD_write(INSTR_WR,0x06);		        // Entry mode set, increment mode
}


void LCD_write (unsigned char RS, unsigned char data)		//Writing to the LCD function
{
	busy_check();
	if(RS==DATA_WR) PORTA = 0b10000000;		// write data: RS = 1 E = 0,R/!W=0 (write) 
	else PORTA = 0b00000000;		        	// Write instruction: RS = 0 E = 0, R/!W=0 (write)
	PORTA = PORTA | 0x40;			          	// Take E HIGH (logic 1)
	PORTB = data;					                // PORTB is equal to data
	PORTA = PORTA & 0x80;			          	// Take E LOW (logic 0)
}

void LCD_string(volatile char *str_ptr)		//LCD string to function
{
	PORTA = 0b10000000;			      // write data: RS = 1 E = 0, R/!W=0 (write)

	while(*str_ptr != '\0')			 // It checks is if string id not equal to null
	{
	busy_check();				       // Busy_check
	PORTA = 0b10000000;			   // Write data: RS = 1 E = 0, R/!W=0 (write)
	PORTA = PORTA | 0x40;			 // Take E HIGH (logic 1)
	PORTB = *str_ptr++;				 // PORTB = string pointer
	PORTA = PORTA & 0x80;			 // Take E LOW (logic 0)
	}
}

void busy_check (void)				 //Check Busy function
{
	PORTA = 0b00100000;
	DDRB = DDRB & 0b01111111;
	do
	{
	PORTA = PORTA | 0b01000000;
	PORTA = PORTA & 0b00100000;
	} while (PINB & 0b10000000);
	DDRB = 0b11111111;
}

