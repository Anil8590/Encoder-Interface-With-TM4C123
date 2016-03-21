# Encoder-Interface-With-TM4C123
//There are two Quadrature Encoder Interface(QEI) Modules in TM4C123. In this project i am using QEI0 module for getting displacement //from Actuator.
//This Project uses QEIO PD6/PD7.
//To Use PD7 as QEI peripherals, you must include following code:
//HWREG(GPIO_PORTD_BASE + GPIO_O_LOCK) = GPIO_LOCK_KEY;
//HWREG(GPIO_PORTD_BASE + GPIO_O_CR) |= 0x80;
//HWREG(GPIO_PORTD_BASE + GPIO_O_LOCK) = 0;
//*****************************************************************************

#include <stdint.h>
#include <stdbool.h>
#include "inc/hw_gpio.h"
#include "inc/hw_types.h"
#include "inc/hw_ints.h"
#include "inc/hw_memmap.h"
#include "driverlib/sysctl.h"
#include "driverlib/pin_map.h"
#include "driverlib/gpio.h"
#include "driverlib/qei.h"
#include "driverlib/interrupt.h"
#include "driverlib/timer.h"
#include <time.h>
void encoder0_init(void);
uint32_t qei0Position;
int main(void)
{
    Hardware_init();
    encoder0_init();
    QEIEnable(QEI0_BASE);//Enable QEI0 Module.
	  QEIPositionSet(QEI0_BASE,000000);//This Function will set Current position to zero.
	` qei0Position = QEIPositionGet(QEI0_BASE);//QEIPositionGet() Function is used to get Current Encoder Position.

    while(1);

}
void encoder0_init(void)
{
	  //Set the clock frequency to 40Mhz.
		SysCtlClockSet(SYSCTL_SYSDIV_4|SYSCTL_USE_PLL|SYSCTL_XTAL_16MHZ|SYSCTL_OSC_MAIN);

		// Enable QEI Peripherals
		SysCtlPeripheralEnable(SYSCTL_PERIPH_GPIOD);
		SysCtlPeripheralEnable(SYSCTL_PERIPH_QEI0);

		//Unlock GPIOD7 - Like PF0 its used for NMI - Without this step it doesn't work
		HWREG(GPIO_PORTD_BASE + GPIO_O_LOCK) = GPIO_LOCK_KEY;
		HWREG(GPIO_PORTD_BASE + GPIO_O_CR) |= 0x80;
		HWREG(GPIO_PORTD_BASE + GPIO_O_LOCK) = 0;


		//Set Pins to be PHA0 and PHB0
		GPIOPinConfigure(GPIO_PD6_PHA0);
		GPIOPinConfigure(GPIO_PD7_PHB0);

		//Set GPIO pins for QEI. PhA0 -> PD6, PhB0 ->PD7.
		GPIOPinTypeQEI(GPIO_PORTD_BASE, GPIO_PIN_6 |  GPIO_PIN_7);

		//DISable peripheral and int before configuration
		QEIDisable(QEI0_BASE);
		QEIIntDisable(QEI0_BASE,QEI_INTERROR | QEI_INTDIR | QEI_INTTIMER | QEI_INTINDEX);

		// Configure quadrature encoder, use an arbitrary top limit of 65535
		QEIConfigure(QEI0_BASE, (QEI_CONFIG_CAPTURE_A_B  | QEI_CONFIG_NO_RESET 	| QEI_CONFIG_QUADRATURE | QEI_CONFIG_NO_SWAP),8000000);

}
