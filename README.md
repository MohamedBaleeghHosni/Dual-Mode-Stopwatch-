# ⏱️ Dual Mode Stopwatch — ATmega32 & Seven-Segment Display

> A digital stopwatch built on an ATmega32 microcontroller featuring two operational modes — count-up and countdown — displayed across six multiplexed seven-segment displays with full button control, LED indicators, and buzzer alarm.

![Status](https://img.shields.io/badge/Status-Completed-brightgreen)
![Platform](https://img.shields.io/badge/Platform-ATmega32-blue)
![Language](https://img.shields.io/badge/Language-C%20%2F%20Embedded-orange)
![Timer](https://img.shields.io/badge/Timer-CTC%20Mode-purple)
![Frequency](https://img.shields.io/badge/Frequency-16%20MHz-red)

---

## 📸 Demo

> _Photos of the physical build coming soon._

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Features](#-features)
- [Hardware Components](#-hardware-components)
- [Pin Configuration](#-pin-configuration)
- [System Architecture](#-system-architecture)
- [How It Works](#-how-it-works)
- [Source Code](#-source-code)
- [Getting Started](#-getting-started)
- [How to Use](#-how-to-use)
- [Team & Contributions](#-team--contributions)

---

## 🔭 Overview

This project implements a **dual-mode digital stopwatch** using the ATmega32 microcontroller running at **16 MHz**. The stopwatch displays time in **HH:MM:SS** format across six multiplexed seven-segment displays (common anode), driven by a **7447 BCD decoder** using the multiplexing technique.

The system supports two operational modes:

| Mode | Description |
|------|-------------|
| **Increment Mode** (default) | Counts upward from 00:00:00 — classic stopwatch |
| **Countdown Mode** | Counts down from a user-set time, triggers buzzer alarm at zero |

Built as a mini project for the **Standard Embedded Diploma** under Edges for Training.

---

## ✨ Features

- ⏫ **Increment mode** — counts up from zero automatically on power-on
- ⏬ **Countdown mode** — user sets target time, buzzer triggers at zero
- ⏸️ **Pause / ▶️ Resume** — hold current time and continue from it
- 🔄 **Reset** — returns time to 00:00:00 instantly
- 🔀 **Mode toggle** — switch between increment and countdown with one button
- 🕐 **Time adjustment buttons** — increment/decrement hours, minutes, seconds independently
- 💡 **LED indicators** — Red LED for count-up, Yellow LED for countdown
- 🔔 **Buzzer alarm** — activates when countdown reaches zero
- 🖥️ **Six multiplexed 7-segment displays** — persistence-of-vision multiplexing for clean HH:MM:SS display
- ⚙️ **Timer1 CTC mode** — precise 1-second tick using hardware timer

---

## 🔧 Hardware Components

| Component | Details |
|-----------|---------|
| Microcontroller | ATmega32 @ 16 MHz |
| Display | 6× Seven-Segment Display (Common Anode) |
| BCD Decoder | 7447 (BCD to 7-segment) |
| Transistors | NPN BJT × 6 (multiplexing enable/disable per display) |
| Push Buttons | 10 total (reset, pause, resume, mode, H+, H−, M+, M−, S+, S−) |
| LED Indicators | Red (PD4) — count-up, Yellow (PD5) — countdown |
| Buzzer | Connected to PD0 |

---

## 📌 Pin Configuration

### PORTA — Display Enable (Multiplexing)

| Pin | Function |
|-----|----------|
| PA0 | Enable 7-segment 1 (seconds units) |
| PA1 | Enable 7-segment 2 (seconds tens) |
| PA2 | Enable 7-segment 3 (minutes units) |
| PA3 | Enable 7-segment 4 (minutes tens) |
| PA4 | Enable 7-segment 5 (hours units) |
| PA5 | Enable 7-segment 6 (hours tens) |

### PORTC — BCD Data (to 7447 decoder)

| Pin | Function |
|-----|----------|
| PC0–PC3 | BCD output to 7447 decoder |

### PORTD — Interrupts, Buzzer, LEDs

| Pin | Function | Trigger |
|-----|----------|---------|
| PD0 | Buzzer | Activated at countdown = 0 |
| PD2 (INT0) | Reset button | Falling edge, internal pull-up |
| PD3 (INT1) | Pause button | Rising edge, external pull-down |
| PD4 | Red LED | ON in increment mode |
| PD5 | Yellow LED | ON in countdown mode |

### PORTB — Resume, Mode Toggle, Time Adjustment

| Pin | Function | Pull |
|-----|----------|------|
| PB2 (INT2) | Resume button | Falling edge, internal pull-up |
| PB7 | Mode toggle | Internal pull-up |
| PB0 | Decrement hours | Internal pull-up |
| PB1 | Increment hours | Internal pull-up |
| PB3 | Decrement minutes | Internal pull-up |
| PB4 | Increment minutes | Internal pull-up |
| PB5 | Decrement seconds | Internal pull-up |
| PB6 | Increment seconds | Internal pull-up |

---

## 🏗 System Architecture

```
                        ATmega32 @ 16MHz
                       ┌──────────────────────────────┐
  [Reset  - INT0/PD2] ─┤ External Interrupt 0          │
  [Pause  - INT1/PD3] ─┤ External Interrupt 1          │
  [Resume - INT2/PB2] ─┤ External Interrupt 2          │
                        │                              │
  [Mode Toggle - PB7] ─┤ PORTB                        ├─► [Red LED   - PD4]
  [H+/H-  - PB1/PB0] ─┤                              ├─► [Yellow LED - PD5]
  [M+/M-  - PB4/PB3] ─┤                              ├─► [Buzzer    - PD0]
  [S+/S-  - PB6/PB5] ─┤                              │
                        │  Timer1 (CTC Mode)           ├─► [PC0-PC3] → [7447 BCD Decoder]
                        │  1-second tick               │                      │
                        │                              ├─► [PA0-PA5]          ▼
                        └──────────────────────────────┘   (Multiplexing) [6× 7-Segment]
                                                            via NPN BJTs
```

**Multiplexing logic:**

```
Cycle through PA0→PA5 rapidly (each display ON for ~2ms)
Send BCD digit for that position to PC0-PC3
Due to persistence of vision → appears as steady HH:MM:SS display
```

---

## ⚙️ How It Works

### Timer1 CTC Mode

Timer1 is configured in **CTC (Clear Timer on Compare Match)** mode. At 16 MHz with a suitable prescaler and compare value, it generates an interrupt **exactly every 1 second** to increment or decrement the time counter.

### Interrupt Summary

| Interrupt | Edge | Resistor | Action |
|-----------|------|----------|--------|
| INT0 (PD2) | Falling | Internal pull-up | Reset time to 00:00:00 |
| INT1 (PD3) | Rising | External pull-down | Pause counting |
| INT2 (PB2) | Falling | Internal pull-up | Resume counting |

### Mode Switching Flow

```
Power ON → Increment Mode (Red LED ON)
                │
         [Mode button PB7]
                │
                ▼
         Countdown Mode (Yellow LED ON)
         → Pause timer first
         → Set time using adjustment buttons
         → Press Resume to start countdown
         → Buzzer triggers when time reaches 00:00:00
```

---

## 💻 Source Code

```c
/*
 * StopWatch_max.c
 *
 *  Created on: Apr 12, 2026
 *      Author: MoTheBaleegh
 */
#include <avr/io.h>
#include <avr/interrupt.h>
#include "TIMER.h"
#include "std_types.h"
#include "keypad.h"
#include "gpio.h"
#include "common_macros.h"
#include "util/delay.h"
#include "ALARM.h"
#include "LED.h"

/*******************************************************************************
 *                              Pin Definitions
 *******************************************************************************/
#define hour_increment_button_port PORTB_ID
#define hour_increment_button_pin PIN1_ID
#define hour_decrement_button_port PORTB_ID
#define hour_decrement_button_pin PIN0_ID
#define min_increment_button_port PORTB_ID
#define min_increment_button_pin PIN4_ID
#define min_decrement_button_port PORTB_ID
#define min_decrement_button_pin PIN3_ID
#define sec_increment_button_port PORTB_ID
#define sec_increment_button_pin PIN6_ID
#define sec_decrement_button_port PORTB_ID
#define sec_decrement_button_pin PIN5_ID
#define reset_button_port         PORTD_ID
#define reset_button_pin          PIN2_ID
#define resume_button_port        PORTB_ID
#define resume_button_pin         PIN2_ID
#define pause_button_port         PORTD_ID
#define pause_button_pin          PIN3_ID
#define switch_mode_button_port   PORTB_ID
#define switch_mode_button_pin    PIN7_ID
/*******************************************************************************
 *                         Global Variables                                *
 *******************************************************************************/
enum Timer_Mode{ COUNT_UP , COUNT_DOWN};

unsigned char mode = COUNT_UP;
unsigned char Sec_unit = 0 , Sec_tenth = 0 ,
			  Min_unit = 0 , Min_tenth = 0 ,
			  Hours_unit = 0 , Hours_tenth = 0 ;
unsigned char flag = 0 /*mode flag*/, flag_S_inc = 0 , flag_S_dec ,
				  flag_M_inc = 0, flag_M_dec = 0 , flag_H_inc = 0 ,
				  flag_H_dec = 0;



/*******************************************************************************
 *                         Types Declaration                                 *
 *******************************************************************************/
Timer_ConfigType Timer1_Configuration = {
			0,
			15625, /* 1 second per tick*/
			TIMER1_ID,
			TIMER_FCPU_1024,
			TIMER_MODE_COMPARE
	};


/*******************************************************************************
 *                         Functions                                   *
 *******************************************************************************/




/*
 * interrupt 0 which is for the reset function connected into PD2 in the MCU.
 * The button is internal pull up resistor type
 * */
void INT0_Init(void)
{
	GICR |= (1 << INT0);
	MCUCR |= (1 << ISC01);
	GPIO_setupPinDirection(PORTD_ID,PIN2_ID,PIN_INPUT);
	GPIO_writePin(PORTD_ID,PIN2,1);// INTERNAL PULL UP RESISTOR
}



/*
 * interrupt 1 which is for the pause function connected into PD3 in the MCU.
 * The button is internal pull down resistor type
 * */
void INT1_Init(void)
{
	GICR |= (1 << INT1);
	MCUCR |= (1 << ISC10) | (1 << ISC11);
	DDRD &= ~(1 << 3);

}



/*
 * interrupt 2 which is for the resume function connected into PB2 in the MCU.
 * The button is internal pull up resistor type
 * */
void INT2_Init(void)
{
	GICR |= (1 << INT2);
	MCUCSR |= (1 << ISC2);
	GPIO_setupPinDirection(PORTB_ID, PIN2_ID, PIN_INPUT);
	GPIO_writePin(PORTB_ID, PIN2, 1); // INTERNAL PULL UP RESISTOR
}

// Reset the Timer
ISR(INT0_vect)
{
	TCNT1 = 0;
	Sec_unit = 0 , Sec_tenth = 0 ,
	Min_unit = 0 , Min_tenth = 0 ,
	Hours_unit = 0 , Hours_tenth = 0 ;
}
// Pause the timer
ISR(INT1_vect)
{
	TCCR1B &= ~((1<<CS10) | (1<<CS11) | (1<<CS12));
}
// Resume the timer
ISR(INT2_vect)
{
	TCCR1B |= (1 << CS10) | (1 << CS12);
}

void Timer1_AppTick(void)
{
	if(mode == COUNT_UP)/* Automatic Count up mode */
	{
		if(Sec_unit < 9)
		{
			Sec_unit++;
		}
		else
		{
			Sec_unit = 0;
			if(Sec_tenth < 5)
			{
				Sec_tenth++;
			}
			else
			{
				Sec_tenth = 0;
				if(Min_unit < 9)
				{
					Min_unit++;
				}
				else
				{
					Min_unit = 0;
					if(Min_tenth < 5)
					{
						Min_tenth++;
					}
					else
					{
						Min_tenth = 0;
						if(Hours_unit < 9)
						{
							Hours_unit++;
						}
						else
						{
							Hours_unit = 0;
							Hours_tenth++;
						}
					}
				}
			}
		}
	}
	else if (mode == COUNT_DOWN)/* Automatic Count down mode */
	{
		 // If already at 00:00:00 wait
		if (Hours_tenth == 0 && Hours_unit == 0 &&
		        Min_tenth == 0 && Min_unit == 0 &&
		        Sec_tenth == 0 && Sec_unit == 0)
		    {
		       return;
		    }
		else
		{
			if(Sec_unit > 0)
			{
				Sec_unit--;
			}
			else
			{
				Sec_unit = 9;
				if(Sec_tenth > 0)
				{
					Sec_tenth--;
				}
				else
				{
					Sec_tenth = 5;
					if(Min_unit > 0)
					{
						Min_unit--;
					}
					else
					{
						Min_unit = 9;
						if(Min_tenth > 0)
						{
							Min_tenth--;
						}
						else
						{
							Min_tenth = 5;
							if(Hours_unit > 0)
							{
								Hours_unit--;
							}
							else
							{
								Hours_unit = 9;
								Hours_tenth--;
							}
						}
					}
				}
			}
		}
	}
}



/*******************************************************************************
 *                        MAIN FUNCTION                                  *
 *******************************************************************************/
int main (void)
{
	SREG |= (1 << 7);/* Enable the  global interrupts*/
	/* Function Initializations */
	Timer_init(&Timer1_Configuration);
	Timer_setCallBack(Timer1_AppTick, TIMER1_ID);
	INT0_Init();
	INT1_Init();
	INT2_Init();
	LEDS_init();
	Alarm_init();
	DDRC |= 0x0F;
	PORTC &= 0xF0;
	DDRA |= 0x3F;
	PORTA &= 0xC0;
	// Button for mode switch , hours , minutes and second increment and decrement
	DDRB &= 0x04;
	PORTB |= 0xFB; // INTERNAL PULL UP RESISTOR
	while(1)
	{

		// check if timer is paused
		if (!(TCCR1B & ((1 << CS10) | (1 << CS11) | (1 << CS12))))
		{
		    // ================= MODE TOGGLE =================
			if (GPIO_readPin(PORTB_ID, PIN7_ID) == LOGIC_LOW)
			{
			    _delay_ms(30);

			    if (GPIO_readPin(PORTB_ID, PIN7_ID) == LOGIC_LOW)
			    {
			        if (flag == 0)
			        {
			            mode ^= 1;
			            LED_toggle(RED_LED);
			            LED_toggle(YELLOW_LED);

			            flag = 1;
			        }
			    }
			}
			else
			{
			    flag = 0;

			}

		    // ================= SECONDS INC =================
		    if (GPIO_readPin(PORTB_ID, PIN6_ID) == LOGIC_LOW)
		    {
		        _delay_ms(30);
		        if (GPIO_readPin(PORTB_ID, PIN6_ID) == LOGIC_LOW)
		        {
		            if (flag_S_inc == 0)
		            {
		                flag_S_inc = 1;

		                if (Sec_unit < 9)
		                {
		                    Sec_unit++;
		                }
		                else if (Sec_tenth < 5)
		                {
		                    Sec_unit = 0;
		                    Sec_tenth++;
		                }
		            }
		        }
		    }
		    else
		    {
		        flag_S_inc = 0;
		    }

		    // ================= SECONDS DEC =================
		    if (GPIO_readPin(PORTB_ID, PIN5_ID) == LOGIC_LOW)
		    {
		        _delay_ms(30);
		        if (GPIO_readPin(PORTB_ID, PIN5_ID) == LOGIC_LOW)
		        {
		            if (flag_S_dec == 0)
		            {
		                flag_S_dec = 1;

		                if (Sec_unit > 0)
		                {
		                    Sec_unit--;
		                }
		                else if (Sec_tenth > 0)
		                {
		                    Sec_unit = 9;
		                    Sec_tenth--;
		                }
		            }
		        }
		    }
		    else
		    {
		        flag_S_dec = 0;
		    }

		    // ================= MINUTES INC =================
		    if (GPIO_readPin(PORTB_ID, PIN4_ID) == LOGIC_LOW)
		    {
		        _delay_ms(30);
		        if (GPIO_readPin(PORTB_ID, PIN4_ID) == LOGIC_LOW)
		        {
		            if (flag_M_inc == 0)
		            {
		                flag_M_inc = 1;

		                if (Min_unit < 9)
		                {
		                    Min_unit++;
		                }
		                else if (Min_tenth < 5)
		                {
		                    Min_unit = 0;
		                    Min_tenth++;
		                }
		            }
		        }
		    }
		    else
		    {
		        flag_M_inc = 0;
		    }

		    // ================= MINUTES DEC =================
		    if (GPIO_readPin(PORTB_ID, PIN3_ID) == LOGIC_LOW)
		    {
		        _delay_ms(30);
		        if (GPIO_readPin(PORTB_ID, PIN3_ID) == LOGIC_LOW)
		        {
		            if (flag_M_dec == 0)
		            {
		                flag_M_dec = 1;

		                if (Min_unit > 0)
		                {
		                    Min_unit--;
		                }
		                else if (Min_tenth > 0)
		                {
		                    Min_unit = 9;
		                    Min_tenth--;
		                }
		            }
		        }
		    }
		    else
		    {
		        flag_M_dec = 0;
		    }

		    // ================= HOURS INC =================
		    if (GPIO_readPin(PORTB_ID, PIN1_ID) == LOGIC_LOW)
		    {
		        _delay_ms(30);
		        if (GPIO_readPin(PORTB_ID, PIN1_ID) == LOGIC_LOW)
		        {
		            if (flag_H_inc == 0)
		            {
		                flag_H_inc = 1;

		                if (Hours_unit < 9)
		                {
		                    Hours_unit++;
		                }
		                else if (Hours_tenth < 9)
		                {
		                    Hours_unit = 0;
		                    Hours_tenth++;
		                }
		            }
		        }
		    }
		    else
		    {
		        flag_H_inc = 0;
		    }

		    // ================= HOURS DEC =================
		    if (GPIO_readPin(PORTB_ID, PIN0_ID) == LOGIC_LOW)
		    {
		        _delay_ms(30);
		        if (GPIO_readPin(PORTB_ID, PIN0_ID) == LOGIC_LOW)
		        {
		            if (flag_H_dec == 0)
		            {
		                flag_H_dec = 1;

		                if (Hours_unit > 0)
		                {
		                    Hours_unit--;
		                }
		                else if (Hours_tenth > 0)
		                {
		                    Hours_unit = 9;
		                    Hours_tenth--;
		                }
		            }
		        }
		    }
		    else
		    {
		        flag_H_dec = 0;
		    }
		}
		if (mode == COUNT_DOWN && Hours_tenth == 0 && Hours_unit == 0
				&& Min_tenth == 0 && Min_unit == 0 && Sec_tenth == 0
				&& Sec_unit == 0)
		{
			Alarm_on();
		}
		else
		{
			Alarm_off();
		}

		/* Displaying on the seven segment */
		PORTA |= (1 << 0);
		PORTC = (PORTC & 0xF0) | (Sec_unit & 0x0F);
		_delay_ms(2);
		PORTA &= ~(1 << 0);

		PORTA |= (1 << 1);
		PORTC = (PORTC & 0xF0) | (Sec_tenth & 0x0F);
		_delay_ms(2);
		PORTA &= ~(1 << 1);

		PORTA |= (1 << 2);
		PORTC = (PORTC & 0xF0) | (Min_unit & 0x0F);
		_delay_ms(2);
		PORTA &= ~(1 << 2);

		PORTA |= (1 << 3);
		PORTC = (PORTC & 0xF0) | (Min_tenth & 0x0F);
		_delay_ms(2);
		PORTA &= ~(1 << 3);

		PORTA |= (1 << 4);
		PORTC = (PORTC & 0xF0) | (Hours_unit & 0x0F);
		_delay_ms(2);
		PORTA &= ~(1 << 4);

		PORTA |= (1 << 5);
		PORTC = (PORTC & 0xF0) | (Hours_tenth & 0x0F);
		_delay_ms(2);
		PORTA &= ~(1 << 5);
	}
}



```

---

## 🚀 Getting Started

### Prerequisites

- **AVR-GCC** toolchain installed, or
- **Atmel Studio / Microchip Studio**, or
- **Eclipse IDE** with AVR plugin
- **AVRDUDE** for flashing
- **USBasp** or similar AVR programmer

### Compile & Flash

Using AVR-GCC from terminal:

```bash
# Compile
avr-gcc -mmcu=atmega32 -DF_CPU=16000000UL -Os -o stopwatch.elf main.c

# Convert to hex
avr-objcopy -O ihex stopwatch.elf stopwatch.hex

# Flash to ATmega32
avrdude -c usbasp -p m32 -U flash:w:stopwatch.hex
```

### Fuse Bits

Make sure the ATmega32 is configured for **external 16 MHz crystal**:

```
Low fuse:  0xBF
High fuse: 0x89
```

---

## 📱 How to Use

**Increment Mode (default on power-on):**
1. Power on → stopwatch starts counting up automatically
2. Red LED (PD4) confirms increment mode
3. Press **Pause (PD3)** to freeze the display
4. Press **Resume (PB2)** to continue
5. Press **Reset (PD2)** to return to 00:00:00

**Countdown Mode:**
1. Press **Pause (PD3)** first to stop the timer
2. Press **Mode Toggle (PB7)** — Yellow LED (PD5) turns ON
3. Use adjustment buttons to set your target time:
   - `PB1 / PB0` → Hours up / down
   - `PB4 / PB3` → Minutes up / down
   - `PB6 / PB5` → Seconds up / down
4. Press **Resume (PB2)** to start the countdown
5. Buzzer (PD0) sounds when countdown reaches 00:00:00

---



> This was an individual mini project completed as part of the **Standard Embedded Systems Diploma** under **Edges for Training**, supervised by Eng. Mohamed Tarek.

---

<p align="center">Built as Mini Project 2 — Standard Embedded Systems Diploma | Edges for Training</p>
