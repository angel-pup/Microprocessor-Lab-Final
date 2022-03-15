//RESOURCES
/* Design specific to Nexys 4 Artix 7 FPGA
https://digilent.com/reference/programmable-logic/nexys-4/start
https://www.xilinx.com/products/design-tools/microblaze.html
*/

#include <stdio.h>
#include "platform.h"
#include "xparameters.h"
#include "mb_interface.h"

#define LED_DATA	   		*(volatile int *) (XPAR_LEDS_8BITS_BASEADDR)
#define SWITCH_DATA	       	*(volatile int *) (XPAR_DIP_SWITCHES_8BITS_BASEADDR)

#define	PORT0_DATA			*(volatile int *) (XPAR_XPS_GPIO_0_BASEADDR)
#define	PORT0_DIRECTION		*(volatile int *) (XPAR_XPS_GPIO_0_BASEADDR + 0x04)

#define	PORT1_DATA			*(volatile int *) (XPAR_XPS_GPIO_1_BASEADDR)
#define	PORT1_DIRECTION		*(volatile int *) (XPAR_XPS_GPIO_1_BASEADDR + 0x04)

#define PORT2_DATA			*(volatile int *) (XPAR_XPS_GPIO_2_BASEADDR)
#define PORT2_DIRECTION		*(volatile int *) (XPAR_XPS_GPIO_2_BASEADDR + 0x04)

#define PORT2_GIER			*(volatile int *) (XPAR_XPS_GPIO_2_BASEADDR + 0x11C)
#define PORT2_IPISR			*(volatile int *) (XPAR_XPS_GPIO_2_BASEADDR + 0x120)
#define PORT2_IPIER			*(volatile int *) (XPAR_XPS_GPIO_2_BASEADDR + 0x128)

#define SERIAL2_RX_FIFO		*(volatile int *) (XPAR_XPS_UARTLITE_0_BASEADDR)
#define SERIAL2_TX_FIFO		*(volatile int *) (XPAR_XPS_UARTLITE_0_BASEADDR + 0x04)
#define SERIAL2_STATUS		*(volatile int *) (XPAR_XPS_UARTLITE_0_BASEADDR + 0x08)

#define	SPICR				*(volatile int *) (XPAR_XPS_SPI_0_BASEADDR+0x60)
#define	SPISR				*(volatile int *) (XPAR_XPS_SPI_0_BASEADDR+0x64)
#define	SPIDTR				*(volatile int *) (XPAR_XPS_SPI_0_BASEADDR+0x68)
#define	SPIDRR				*(volatile int *) (XPAR_XPS_SPI_0_BASEADDR+0x6C)
#define	SPISSR				*(volatile int *) (XPAR_XPS_SPI_0_BASEADDR+0x70)

#define	A_TO_D_TRIGGER		*(volatile int *) (XPAR_PMOD_AD1_0_BASEADDR)
#define A_TO_D_DONE			*(volatile int *) (XPAR_PMOD_AD1_0_BASEADDR + 0x04)
#define A_TO_D_DATA0		*(volatile int *) (XPAR_PMOD_AD1_0_BASEADDR + 0x08)
#define A_TO_D_DATA1		*(volatile int *) (XPAR_PMOD_AD1_0_BASEADDR + 0x0C)

#define SEVEN_SEG_DATA		*(volatile int *) (XPAR_DIGILENT_SEVSEG_DISP_BASEADDR)

#define TIMER00_CSR			*(volatile int *) (XPAR_XPS_TIMER_0_BASEADDR)
#define TIMER00_LOAD		*(volatile int *) (XPAR_XPS_TIMER_0_BASEADDR + 0x04)
#define TIMER01_CSR			*(volatile int *) (XPAR_XPS_TIMER_0_BASEADDR + 0x10)
#define TIMER01_LOAD		*(volatile int *) (XPAR_XPS_TIMER_0_BASEADDR + 0x14)

#define TIMER10_CSR			*(volatile int *) (XPAR_XPS_TIMER_1_BASEADDR)
#define TIMER10_LOAD		*(volatile int *) (XPAR_XPS_TIMER_1_BASEADDR + 0x04)
#define TIMER11_CSR			*(volatile int *) (XPAR_XPS_TIMER_1_BASEADDR + 0x10)
#define TIMER11_LOAD		*(volatile int *) (XPAR_XPS_TIMER_1_BASEADDR + 0x14)

#define INT_ISR				*(volatile int *) (XPAR_XPS_INTC_0_BASEADDR)
#define INT_IPR				*(volatile int *) (XPAR_XPS_INTC_0_BASEADDR + 0x4)
#define INT_IER				*(volatile int *) (XPAR_XPS_INTC_0_BASEADDR + 0x8)
#define INT_IAR				*(volatile int *) (XPAR_XPS_INTC_0_BASEADDR + 0xC)
#define INT_MER				*(volatile int *) (XPAR_XPS_INTC_0_BASEADDR + 0x1C)

/* Serial 2 Status Register bits */
#define TX_FIFO_FULL		0x08	// Transmit FIFO is full
#define RX_FIFO_VALID_DATA	0x01	// Receive FIFO has valid data in it

/* SPI Control Register bits */
#define	LSB_FIRST			0x200	// Send data LSB first (1) or MSB first (0)
#define MASTER_INHIBIT		0x100	// Master transactions disabled (1) or enabled (0)
#define MANUAL_SLAVE_SELECT	0x080	// SS follows slave select register (1) or logic (0)
#define RX_FIFO_RESET		0x040	// Reset RX FIFO (1) or normal operation (0)
#define TX_FIFO_RESET		0x020	// Reset TX FIFO (1) or normal operation (0)
#define CLK_PHASE_ADJUST	0x010	// Select one of two different formats (use default = 0)
#define CLK_POLARITY		0x008	// Select SCK to idle high (1) or low (0)
#define MASTER_MODE			0x004	// Select module to be Master (1) or Slave (0)
#define SPI_ENABLED			0x002	// Enable SPI module (1) or disable SPI module (1)
#define LOOP_MODE			0x001	// Select loopback mode (1) or normal mode (0)

/* SPI Status Register bits */
#define TX_FIFO_EMPTY		0x04	// Transmit FIFO is empty, and receive FIFO now has data

/* Timer Control and Status Register bits */
#define	ENABLE_ALL			0x400	// Enable all timers
#define	PWM_MODE			0x200	// Enable PWM output
#define ENABLE_TIMER		0x080	// Enable this timer
#define	ENABLE_INTS			0x040	// Enable interrupts for this timer
#define LOAD_TIMER			0x020	// Load timer with load value
#define AUTO_RELOAD			0x010	// Auto Reload timer when expired
#define ENABLE_EXT_CAPTURE	0x008	// Enable external capture trigger
#define ENABLE_GEN_SIGNAL	0x004	// Enable external generate signal
#define DOWN_COUNTER		0x002	// Configure as down counter (1) or up counter (0)
#define CAPTURE_MODE		0x001	// Configure to capture signal (1) or generate signal (0)

/* Interrupt Control Register bits */
#define GPIO2_INTR			0x0001	// Enable Port 2 interrupt
#define TIMER1_INTR			0x0002	// Enable Timer1 interrupt
#define TIMER0_INTR			0x0004	// Enable Timer0 interrupt
#define PUSHBUTTON_INTR		0x0008	// Enable Pushbutton interrupt

#define HARDWARE_INT_ENABLE	0x0002	// Enable hardware interrupts
#define MASTER_INT_ENABLE	0x0001	// Enable master interrupts

#define P_Read_Offset 		0x800	//Offset for Potentiometer

//Special characters and 'Quick' list
#define Char_Esc			0x1b

//LCD dimensions
//#define LCD_ROWS			2
//#define LCD_COLUMNS			16

//Function prototypes
//unsigned char getCharacter();
void putCharacter(unsigned char);
void put_String(char *string);
void LCD_setup();
//void Delay_ms(unsigned short delay_time);

void microblaze_0_Interrupt(void) __attribute__ ((interrupt_handler));

/* Global Variables */
unsigned short timer_flag;
unsigned short counter;

/* Interrupt Service Routine */
void microblaze_0_Interrupt(void){

	int tmp;

	//Read IPR
	tmp = INT_IPR;

	//Is this a PORT 2 interrupt?
	if((tmp & GPIO2_INTR)!=0x00)
	{
		//increment counter
		counter++;
		//Clear interrupt in PORT 2
		PORT2_IPISR = PORT2_IPISR;
	}
	//Is this an Timer1 interrupt?
	if((tmp & TIMER1_INTR)!=0x00)
	{
		//Set the timer flag
		timer_flag=0x01;
		//Clear the interrupt in Timer 1
		TIMER10_CSR = TIMER10_CSR;
	}
	//Clear interrupts in Int controller
	INT_IAR = tmp;
}

int main()
{
	//Enable Caches
	init_platform();

	//Declare Variables
	unsigned short P_Read;
	unsigned short mod_value_for_P_Read;
	unsigned char current_direction;
	unsigned char seven_segment_counter;
	unsigned char LCD_counter;
	unsigned char RPM_counter;
	unsigned char LED_counter;
	unsigned short old_count;
	unsigned short delta_count;
	unsigned short cpm;
	unsigned char whole;
	unsigned char tenths;
	unsigned char One_Hertz_Counter;
	unsigned char led_bits;

	char fstring [16];

	//Initialize variables to 0
	timer_flag = 0;
	current_direction = 0;
	seven_segment_counter = 0;
	old_count = 0;
	delta_count = 0;
	LED_counter = 0;
	RPM_counter = 0;
	LCD_counter = 0;
	cpm = 0;
	One_Hertz_Counter = 0;

	//Configure PORT 0 to be an output port
	PORT0_DIRECTION = 0;

	//Set MOTOR_ROTATION output equal to current_direction
	PORT0_DATA = current_direction;

	//Configure PORT 2 to be an input
	PORT2_DIRECTION = 0b11;

	//Configure PORT 2 to generate interrupts
	PORT2_GIER = 0x80000000;
	PORT2_IPIER = 0x01;

	//Initialize_LCD and set cursor to home position
	LCD_setup();

	//Print static text on line 0 of LCD
	putCharacter(Char_Esc);
	putCharacter('[');
	putCharacter('0');
	putCharacter(';');
	putCharacter('0');
	putCharacter('H');
	put_String("SE:");

	//Send string to move cursor to start of line 1
	/*Not Needed*/

	//Print static text on line 1 of LCD
	/*Not Needed*/

	//Set Timer00 load value to generate PWM signal at 9.155 kHz with clock speed at 75 MHz
	TIMER00_LOAD = 0x2000;

	//Set Timer01 load value to 0 so no PWM signal will be generated yet
	TIMER01_LOAD = 0;

	//Configure TIMER00 and TIMER01 to count down and generate a PWM signal
	TIMER00_CSR 	= ENABLE_ALL	+	PWM_MODE	+	ENABLE_TIMER	+	ENABLE_GEN_SIGNAL	+	DOWN_COUNTER;
	TIMER01_CSR	= ENABLE_ALL	+	PWM_MODE	+	ENABLE_TIMER	+	ENABLE_GEN_SIGNAL	+	DOWN_COUNTER;

	//Set Timer10 load value to generate an interrupt every 20ms (50 Hz) with clock speed at 75 MHz
	TIMER10_LOAD = 1500000;

	//Configure Timer10 to counter down, auto reload, and generate an interrupt
	TIMER10_CSR	= ENABLE_ALL	+	ENABLE_TIMER	+	ENABLE_INTS	+	LOAD_TIMER		+	AUTO_RELOAD	+	ENABLE_GEN_SIGNAL	+	DOWN_COUNTER;

	//Start Timer10
	TIMER10_CSR	= ENABLE_ALL	+	ENABLE_TIMER	+	ENABLE_INTS					+	AUTO_RELOAD	+	ENABLE_GEN_SIGNAL	+	DOWN_COUNTER;

	//Start the first A/D conversion by setting A_TO_D_TRIGGER to 1
	A_TO_D_TRIGGER = 1;

	//Set TIMER10 and PORT 2 interrupts in Interrupt Controller
    	INT_IER = TIMER1_INTR + GPIO2_INTR;
    	INT_MER = HARDWARE_INT_ENABLE + MASTER_INT_ENABLE;

	//Enable interrupts in Microblaze
	microblaze_enable_interrupts();

	while(1)
	{
		if((timer_flag = 1))
		{
			//Re-arm the A/D converter by setting A_TO_D_TRIGGER to 0
			A_TO_D_TRIGGER = 0;

			//Get latest info from potentiometer
			P_Read = A_TO_D_DATA0;

			//Is the Potentiometer turned to the right?
			if(P_Read>=P_Read_Offset)
			{
				//Subtract center value from current potentiometer value
				mod_value_for_P_Read = P_Read - P_Read_Offset;

				//Set current_direction to 0
				current_direction = 0;
			}
			else{
				//Subtract current potentiometer value from center value
				mod_value_for_P_Read = P_Read_Offset - P_Read - 0x01;

				//Set current_direction to 1
				current_direction = 1;
			}
			//Multiply value by 0x04 to generate a value from 0x0000 - 0x2000
			mod_value_for_P_Read = mod_value_for_P_Read*0x04;

			//Write value to TIMER01_LOAD register to generate PWM signal
			TIMER01_LOAD = mod_value_for_P_Read;

			//Set MOTOR_ROTATION equal to current direction (This is PORT 0, bit 0)
			PORT0_DATA = current_direction;

			//Start the next A/D conversion by setting A_TO_D_TRIGGER to 1
			A_TO_D_TRIGGER = 1;

			//Set timer_flag back to 0
			timer_flag = 0;

			//Increment seven_segment counter
			seven_segment_counter++;

			//Increment LCD counter
			LCD_counter++;

			//Increment RPM counter
			RPM_counter++;

			//Increment LED counter
			LED_counter++;
		}

			if((seven_segment_counter >= 2))
			{
				//Read current Timer01 load register value and write it to the 7-Segment display
				SEVEN_SEG_DATA = TIMER01_LOAD;

				//Clear the 7_segment counter
				seven_segment_counter = 0x00;
			}
			if((LCD_counter >= 5))
			{
				//Send command to LCD to move the cursor to the correct location
				putCharacter(Char_Esc);
				putCharacter('[');
				putCharacter('0');
				putCharacter(';');
				putCharacter('4');
				putCharacter('H');

				//Build a formatted string with the shift encoder count value
				sprintf(fstring, "%d", counter);

				//Send this string to the LCD
				put_String(fstring);

				//Clear the LCD counter
				LCD_counter = 0x00;
			}
			if((RPM_counter>=10))
			{
				//Subtract old_count from current count to get delta_count
				delta_count = counter - old_count;

				//Capture the current count by setting old_count = current count
				old_count = counter;

				//counts per minute = delta_count*5*60
				cpm = delta_count*300;

				//Compute whole part of RPM by dividing counts per minute by 350
				whole = cpm/350;

				//Compute the remainder of counts per minute over 350
				tenths = cpm%350;

				//Compute tenthes part of rpm by dividing rem*10 by 350
				tenths = (tenths*10)/350;

				//Send command to LCD to move the cursor to the correct location
				putCharacter(Char_Esc);
				putCharacter('[');
				putCharacter('1');
				putCharacter(';');
				putCharacter('0');
				putCharacter('H');

				//Build a formatted string with the RPM value
				sprintf(fstring, "RPM:%02d.%d", whole, tenths);

				//Send this string to the LCD
				put_String(fstring);
				//Go to an exact location on LCD screen
				putCharacter(Char_Esc);
				putCharacter('[');
				putCharacter('1');
				putCharacter(';');
				putCharacter('1');
				putCharacter('0');
				putCharacter('H');

				if((current_direction == 0))
				{
					//Send string for "cw"
					put_String("cw ");
				} else { put_String("ccw"); } //Send string for "ccw"

				//Clear the RPM counter
				RPM_counter = 0x00;
			}
			if((LED_counter>=50))
			{
				//Increment another counter that counts at 1 Hz
				One_Hertz_Counter++;

				//Clear the LED counter
				LED_counter = 0x00;
			}
			//Update LED_DATA using the 1 Hz counter for bit 7 and current_direction for bit 0
			led_bits = One_Hertz_Counter*0b10000000;
			LED_DATA = (led_bits & 0b10000000)|(current_direction);
	}
}

void putCharacter(unsigned char c)
{
	//Declare variables
	unsigned char status;

	//Check the status register
	do{
		//Read the status register
		status = SERIAL2_STATUS;

		//Mask the bits unrelated to transmit valid data
		status = status & TX_FIFO_FULL;
	}while(!(status==0));

	//Transmit the character parameter given
	SERIAL2_TX_FIFO = c;
}
/*
unsigned char getCharacter()
{
	//Declare variables
	unsigned char status;
	unsigned char c;

	//Check the status register
	do{
		//Read the status register
		status = SERIAL2_STATUS;

		//Mask the bits unrelated to receive valid data
		status = status & RX_FIFO_VALID_DATA;
	}while(status==0);

	//Read the RX FIFO
	c = SERIAL2_RX_FIFO;

	//Return the received character
	return c;
}*/

void put_String(char string[])
{
	//Declare and initialize variables
	int i = 0;

		do {
			putCharacter(string[i]);
			i = i + 1;
		} while (string[i]);
}

void LCD_setup()
{
	//Enable LCD
	putCharacter(Char_Esc);
	putCharacter('[');
	putCharacter('1');
	putCharacter('e');

	//Turn on the cursor
	putCharacter(Char_Esc);
	putCharacter('[');
	putCharacter('1');
	putCharacter('c');

	//Clear the display and place cursor in top-left corner
	putCharacter(Char_Esc);
	putCharacter('[');
	putCharacter('j');
}
/*
//No Data display yet
int main()
{
	//Declare Variables
	unsigned short P_Read; 		//variable storage for potentiometer read
	unsigned short modvalue; 	//variable storage for modified potentiometer read
	// Enable both data and instruction caches
    init_platform();

    // Set up system here as needed

    //Load Timer00 load value to generate PWM signal
    TIMER00_LOAD = 0x2000;

    //Set Timer01 load value to 0 so no PWM signal will be generated yet
    TIMER01_LOAD = 0;

    //Configure Timer00 and 01 to generate a PWM signal
    TIMER00_CSR = ENABLE_ALL	+	PWM_MODE	+	ENABLE_TIMER	+	ENABLE_GEN_SIGNAL	+	DOWN_COUNTER;
    TIMER01_CSR	= ENABLE_ALL	+	PWM_MODE	+	ENABLE_TIMER	+	ENABLE_GEN_SIGNAL	+	DOWN_COUNTER;

    //Set Timer10 load register with count value
    TIMER10_LOAD = 750000;

    //Configure Timer10 to counter down, auto reload, and generate an interrupt
    TIMER10_CSR	= ENABLE_ALL	+	ENABLE_TIMER	+	ENABLE_INTS	+	LOAD_TIMER	+	AUTO_RELOAD	+	ENABLE_GEN_SIGNAL	+	DOWN_COUNTER;

    //Start Timer
    TIMER10_CSR	= ENABLE_ALL	+	ENABLE_TIMER	+	ENABLE_INTS				+	AUTO_RELOAD	+	ENABLE_GEN_SIGNAL	+	DOWN_COUNTER;

    //Configure Timer Interrupt in Controller
    INT_IER = TIMER1_INTR;
    INT_MER = HARDWARE_INT_ENABLE + MASTER_INT_ENABLE;

    //Enable Interrupts in Microblaze (macro)
    microblaze_enable_interrupts();

    //Start first A/D conversion
    A_TO_D_TRIGGER = 1;
    // Enter loop
    while(1) {

    	//Is timer1flag set?
    	if(timer1flag==0x1)
    	{
    		//Re-arm A/D converter
    		A_TO_D_TRIGGER = 0x00;
    		//Get info from A/D converter
    		P_Read = A_TO_D_DATA0;
    		//Display raw A/D value on 7-segment display
    		if(P_Read>=P_Read_Offset)
    		    	{
    		    		LED_DATA = 0x00;							//Positive Direction Indicator
    		    		SEVEN_SEG_DATA = P_Read - P_Read_Offset;	//Display values for positive reading on 7Seg
    		    	}
    		    	else{
    		    		LED_DATA = 0x01;							//Negative Direction Indicator
    		    		SEVEN_SEG_DATA = P_Read_Offset - P_Read;	//Display values for negative reading on 7Seg
    		    	}
    		//Modify A/D value to match range of PWM values
    		if(P_Read>=P_Read_Offset)
    				{
    					//Positive Direction (Positive Read)
    					modvalue = (P_Read - P_Read_Offset)*4;		//Range adjustment from Potentiometer Range to Motor PWM range
    				}
    				else{
    					//Negative Direction (Negative Read)
    					modvalue = (P_Read_Offset - P_Read)*4;		//Range adjustment from Potentiometer Range to Motor PWM range
    				}
    		//Write modified A/D value to TIMER01_LOAD
    		TIMER01_LOAD = modvalue;
    		//Start next A/D conversion
    		A_TO_D_TRIGGER = 1;
    		//Set timer1flag to 0
    		timer1flag = 0;
    	}
    }
}*/
