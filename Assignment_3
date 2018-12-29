; a3part4.asm
; CSC 230: Spring 2018
;
; Student name: Alwien Dippenaar
; Student ID: V00849850
; Date of completed work: March 25th, 2018
;
; *******************************
; Code provided for Assignment #3
;
; Author: Mike Zastre (2018-Mar-08)
; 
; This skeleton of an assembly-language program is provided to help you
; begin with the programming tasks for A#3. As with A#2, there are 
; "DO NOT TOUCH" sections. You are *not* to modify the lines
; within these sections. The only exceptions are for specific
; changes announced on conneX or in written permission from the course
; instructor. *** Unapproved changes could result in incorrect code
; execution during assignment evaluation, along with an assignment grade
; of zero. ****
;
; I have added for this assignment an additional kind of section
; called "TOUCH CAREFULLY". The intention here is that one or two
; constants can be changed in such a section -- this will be needed
; as you try to test your code on different messages.
;


; =============================================
; ==== BEGINNING OF "DO NOT TOUCH" SECTION ====
; =============================================
;
; In this "DO NOT TOUCH" section are:
;
; (1) assembler directives setting up the interrupt-vector table
;
; (2) "includes" for the LCD display
;
; (3) some definitions of constants we can use later in the
;     program
;
; (4) code for initial setup of the Analog Digital Converter (in the
;     same manner in which it was set up for Lab #4)
;     
; (5) code for setting up our three timers (timer1, timer3, timer4)
;
; After all this initial code, your own solution's code may start.
;

.cseg
.org 0
	jmp reset

; location in vector table for TIMER1 COMPA
;
.org 0x22
	jmp timer1

; location in vector table for TIMER4 COMPA
;
.org 0x54
	jmp timer4

.include "m2560def.inc"
.include "lcd_function_defs.inc"
.include "lcd_function_code.asm"

.cseg

; These two constants can help given what is required by the
; assignment.
;
#define MAX_PATTERN_LENGTH 10
#define BAR_LENGTH 6

; All of these delays are in seconds
;
#define DELAY1 0.5
#define DELAY3 0.1
#define DELAY4 0.01


; The following lines are executed at assembly time -- their
; whole purpose is to compute the counter values that will later
; be stored into the appropriate Output Compare registers during
; timer setup.
;

#define CLOCK 16.0e6 
.equ PRESCALE_DIV=1024  ; implies CS[2:0] is 0b101
.equ TOP1=int(0.5+(CLOCK/PRESCALE_DIV*DELAY1))

.if TOP1>65535
.error "TOP1 is out of range"
.endif

.equ TOP3=int(0.5+(CLOCK/PRESCALE_DIV*DELAY3))
.if TOP3>65535
.error "TOP3 is out of range"
.endif

.equ TOP4=int(0.5+(CLOCK/PRESCALE_DIV*DELAY4))
.if TOP4>65535
.error "TOP4 is out of range"
.endif


reset:
	; initialize the ADC converter (which is neeeded
	; to read buttons on shield). Note that we'll
	; use the interrupt handler for timer4 to
	; read the buttons (i.e., every 10 ms)
	;
	ldi temp, (1 << ADEN) | (1 << ADPS2) | (1 << ADPS1) | (1 << ADPS0)
	sts ADCSRA, temp
	ldi temp, (1 << REFS0)
	sts ADMUX, r16


	; timer1 is for the heartbeat -- i.e., part (1)
	;
    ldi r16, high(TOP1)
    sts OCR1AH, r16
    ldi r16, low(TOP1)
    sts OCR1AL, r16
    ldi r16, 0
    sts TCCR1A, r16
    ldi r16, (1 << WGM12) | (1 << CS12) | (1 << CS10)
    sts TCCR1B, temp
	ldi r16, (1 << OCIE1A)
	sts TIMSK1, r16

	; timer3 is for the LCD display updates -- needed for all parts
	;
    ldi r16, high(TOP3)
    sts OCR3AH, r16
    ldi r16, low(TOP3)
    sts OCR3AL, r16
    ldi r16, 0
    sts TCCR3A, r16
    ldi r16, (1 << WGM32) | (1 << CS32) | (1 << CS30)
    sts TCCR3B, temp

	; timer4 is for reading buttons at 10ms intervals -- i.e., part (2)
    ; and part (3)
	;
    ldi r16, high(TOP4)
    sts OCR4AH, r16
    ldi r16, low(TOP4)
    sts OCR4AL, r16
    ldi r16, 0
    sts TCCR4A, r16
    ldi r16, (1 << WGM42) | (1 << CS42) | (1 << CS40)
    sts TCCR4B, temp
	ldi r16, (1 << OCIE4A)
	sts TIMSK4, r16

    ; flip the switch -- i.e., enable the interrupts
    sei

; =======================================
; ==== END OF "DO NOT TOUCH" SECTION ====
; =======================================


; *********************************************
; **** BEGINNING OF "STUDENT CODE" SECTION **** 
; *********************************************

start:
	; Initialize the buttons
	ldi r16, (1 << ADEN) | (1 << ADPS2) | (1 << ADPS1) | (1 << ADPS0)
	sts ADCSRA, r16
	ldi r16, (1 << REFS0)
	sts ADMUX, r16

	rcall initialize_everything
	
	infinite_loop:
		nop
	rjmp infinite_loop

stop:
    rjmp stop

initialize_everything:
	; initalize lcd screen
	rcall lcd_init
	; load heartbeat chars
	rcall load_chars
	; put heartbeat chars on lcd screen
	rcall put_chars_on
	; initialize button count to zero
	ldi r16, 0x00
	ldi r17, 0x00
	sts high(BUTTON_COUNT), r16
	sts low(BUTTON_COUNT), r17
	; initialize button length
	sts BUTTON_LENGTH, r16
	; initialize the LCD display
	lds r17, high(BUTTON_COUNT)
 	lds r16, low(BUTTON_COUNT)
 	push r17
 	push r16
 	ldi r17, high(DISPLAY_TEXT)
 	ldi r16, low(DISPLAY_TEXT)
	push r17
	push r16
	; convert button count to decimal numbers
 	rcall to_decimal_text
	pop r16
	pop r17
	pop r16
	pop r17
	rcall load_counter
	ret

timer1:
	push YH
	push YL
	push r16
	push r17
	in YL, SREG
	in YH, SREG
	push YH
	push YL
	
	rcall put_chars_off
	rcall timer_3

	pop YL
	pop YH
	out SREG, YH
	out SREG, YL
	pop r17
	pop r16
	pop YL
	pop YH

	reti

timer4:
	push r16
	push r17
	push r18
	push r19
	in r16, SREG
	push r16
	
	rcall check_button
	cpi r18, 1
	brne skip_timer4
		rcall check_if_previous
		cpi r18, 1
		brne skip_timer4
			rcall reset_button_length	
			rcall inc_counter
			rcall timer_3
	skip_timer4:
	
	pop r16
	out SREG, r16
	pop r19
	pop r18
	pop r17
	pop r16
    reti


timer_3:
	cli
	in temp, TIFR3
	sbrs temp, OCF3A
	rjmp timer_3

	ldi temp, 1<< OCF3A
	out TIFR3, temp

	push r16
	push r17

	lds r17, high(BUTTON_COUNT)
 	lds r16, low(BUTTON_COUNT)
 	push r17
 	push r16
 	ldi r17, high(DISPLAY_TEXT)
 	ldi r16, low(DISPLAY_TEXT)
	push r17
	push r16
 	rcall to_decimal_text
	pop r16
	pop r17
	pop r16
	pop r17
	rcall load_counter
	rcall put_chars_on
	
	pop r17
	pop r16
	sei
	ret

; Checks to see if a button is being pressed or not		
check_button:
	cli
	push r16
	push r17
	lds r16, ADCSRA
	ori r16, (1 << ADSC)
	sts ADCSRA, r16

	check_button_wait:
		lds r16, ADCSRA
		sbrc r16, ADSC
		rjmp check_button_wait
		lds r16, ADCL
		lds r17, ADCH
		rcall change_button_state
		sts BUTTON_CURRENT, r17
		clr r18
		cpi r17, high(900)
		brge check_button_skip

		sts BUTTON_CURRENT, r17
		ldi r18, 1
		rcall put_on_asterisks ; puts on asterisks
		rcall inc_button_length ; increments the button length
		pop r17
		pop r16
		sei
		ret

		check_button_skip:
			rcall put_off_asterisks ; puts off asterisks
			rcall display_morse_symbol ; displays the morse sybmol
			pop r17
			pop r16
			sei
			ret

; Checks to see if the button is being held down or not					
check_if_previous:
	cli
	push r16
	push r17

	lds r16, BUTTON_PREVIOUS
	lds r17, BUTTON_CURRENT
	cp r16, r17
	brne check_if_previous_skip
	clr r18
	pop r17
	pop r16
	sei
	ret

	check_if_previous_skip:
	pop r17
	pop r16
	sei
	ret

; Puts the old current value of the current button into the previous
; value, to make room for the new current value			
change_button_state:
	push r16
	push r17

	lds r16, BUTTON_PREVIOUS
	lds r17, BUTTON_CURRENT
	sts BUTTON_CURRENT, r16
	sts BUTTON_PREVIOUS, r17

	pop r17
	pop r16
	ret
	
; Swap the heartbeat chars with spaces		
put_chars_off:
	push YH
	push YL
	push r17
	push r16

	lds YH, CHAR_ONE
	lds YL, CHAR_TWO
	lds r16, CHAR_THREE
	lds r17, CHAR_FOUR

	sts CHAR_THREE, YH
	sts CHAR_ONE, r16
	sts CHAR_FOUR, YL
	sts CHAR_TWO, r17

	pop r16
	pop r17
	pop YL
	pop YH

	ret

; Place the heartbeat chars on the lcd screen		
put_chars_on:
	push r16
	push r17
	ldi r16, 0
	ldi r17, 14
	push r16
	push r17
	rcall lcd_gotoxy
	pop r17
	pop r16

	lds r16,  CHAR_ONE
	sts PULSE, r16
	push r16
	rcall lcd_putchar
	pop r16

	lds r16, CHAR_TWO
	sts PULSE, r16
	push r16
	rcall lcd_putchar
	pop r16

	pop r17
	pop r16
	ret

; Load the chars into memory		
load_chars:
	ldi r16, '<'
	sts CHAR_ONE, r16
	ldi r16, '>'
	sts CHAR_TWO, r16
	ldi r16, ' '
	sts CHAR_THREE, r16
	ldi r16, ' '
	sts CHAR_FOUR, r16
	ret

; Loads the counter with the count that is from data memory	(DISPLAY_TEXT)	
load_counter:
	ldi r16, 1
	ldi r17, 11
	push r16
	push r17
	rcall lcd_gotoxy
	pop r17
	pop r16

	ldi r18, 5
	ldi r30, low(DISPLAY_TEXT)
	ldi r31, high(DISPLAY_TEXT)
	push r18
	push r31
	push r30
	load_counter_loop:
		ld r16, Z+
		rcall which_char
		push r17
		rcall lcd_putchar
		pop r17
		dec r18
	brne load_counter_loop
	pop r30
	pop r31 
	pop r18
	ret

; Increments the count when a button is pressed	
inc_counter:
	cli
	push r16
	push r17
	push r18
	push r19
	ldi r18, 1
	ldi r19, 0
	lds r16, low(BUTTON_COUNT)
	lds r17, high(BUTTON_COUNT)
	add r16, r18
	adc r17, r19
	sts high(BUTTON_COUNT), r17
	sts low(BUTTON_COUNT), r16
	pop r19
	pop r18
	pop r17
	pop r16
	sei
	ret

; Determines which number should appear on the count		
which_char:
	push r18
	push r16
	ldi r18, 9
	ldi r17, '9'
	cp r16, r18
	breq which_char_end
	ldi r17, '8'
	dec r18
	cp r16, r18
	breq which_char_end
	ldi r17, '7'
	dec r18
	cp r16, r18
	breq which_char_end
	ldi r17, '6'
	dec r18
	cp r16, r18
	breq which_char_end
	ldi r17, '5'
	dec r18
	cp r16, r18
	breq which_char_end
	ldi r17, '4'
	dec r18
	cp r16, r18
	breq which_char_end
	ldi r17, '3'
	dec r18
	cp r16, r18
	breq which_char_end
	ldi r17, '2'
	dec r18
	cp r16, r18
	breq which_char_end
	ldi r17, '1'
	dec r18
	cp r16, r18
	breq which_char_end
	ldi r17, '0'
	dec r18
	cp r16, r18
	breq which_char_end
	
	which_char_end:
	pop r16
	pop r18
	ret

; Places 6 asterisks in the bottom right corner
; when a button is being pressed	
put_on_asterisks:
	push r16
	push r17
	ldi r16, 1
	ldi r17, 0
	push r16
	push r17
	rcall lcd_gotoxy
	pop r17
	pop r16

	ldi r17, BAR_LENGTH
	ldi r16, '*'
	
	asterisks_on_loop:
		push r16
		rcall lcd_putchar
		pop r16
	dec r17
	brne asterisks_on_loop

	pop r17
	pop r16
	
	ret

; Places 6 spaces in the bottom right corner
; when a button is not being pushed		
put_off_asterisks:
	push r16
	push r17
	ldi r16, 1
	ldi r17, 0
	push r16
	push r17
	rcall lcd_gotoxy
	pop r17
	pop r16

	ldi r17, BAR_LENGTH
	ldi r16, ' '
	
	asterisks_off_loop:
		push r16
		rcall lcd_putchar
		pop r16
	dec r17
	brne asterisks_off_loop

	pop r17
	pop r16
	ret

; Increments the length of which the button is being held down	
inc_button_length:
	push r16
	lds r16, BUTTON_LENGTH
	inc r16
	sts BUTTON_LENGTH, r16
	pop r16
	ret

; resets the button length when the button is released
reset_button_length:
	cli
	push r16
	ldi r16, 0x00
	sts BUTTON_LENGTH, r16
	pop r16
	sei
	ret

; Displays the correct morse symbol dependent on the length of the button hold
; Will only show up ten symbols	
display_morse_symbol:
	push r16
	push r17

	ldi r16, 0
	lds r17, low(BUTTON_COUNT)
	cpi r17, 11
	brge no_more_symbols
;
	cpi r17, 0
	breq not_started_yet
;	
	dec r17
	push r16
	push r17
	rcall lcd_gotoxy
	pop r17
	pop r16
	
	lds r16, BUTTON_LENGTH
	cpi r16, 20
	brge is_dash
	ldi r17, '.'
	push r17
	rcall lcd_putchar
	pop r17
	pop r17
	pop r16
	ret

	is_dash:
		ldi r17, '-'
		push r17
		rcall lcd_putchar
		pop r17
		pop r17
		pop r16
		ret

	no_more_symbols:
		pop r17
		pop r16
		ret
	
	not_started_yet:
		ldi r17, ' '
		push r17
		rcall lcd_putchar
		pop r17
		pop r17
		pop r16
		ret

; ------------------------------------------------------
; The following "to_decimal_text" courtesy of Dr. Zastre
; ------------------------------------------------------
to_decimal_text:
	.def countL=r18
	.def countH=r19
	.def factorL=r20
	.def factorH=r21
	.def multiple=r22
	.def pos=r23
	.def zero=r2
	.def ascii_zero=r16
	push countH
 	push countL
 	push factorH
 	push factorL
 	push multiple
 	push pos
 	push zero
 	push ascii_zero
 	push YH
 	push YL
 	push ZH
 	push ZL
 	in YH, SPH
 	in YL, SPL
 ; fetch parameters from stack frame
 ;
 	.set PARAM_OFFSET = 16
 	ldd countH, Y+PARAM_OFFSET+3
 	ldd countL, Y+PARAM_OFFSET+2
 ; this is only designed for positive
 ; signed integers; we force a negative
 ; integer to be positive.
 ;
 	andi countH, 0b01111111
 	clr zero
	clr pos
	ldi ascii_zero, 0
 ; The idea here is to build the text representation
 ; digit by digit, starting from the left-most.
 ; Since we need only concern ourselves with final
 ; text strings having five characters (i.e., our
 ; text of the decimal will never be more than
 ; five characters in length), we begin we determining
 ; how many times 10000 fits into countH:countL, and
 ; use that to determine what character (from ’0’ to
 ; ’9’) should appear in the left-most position
 ; of the string.
 ;
 ; Then we do the same thing for 1000, then
 ; for 100, then for 10, and finally for 1.
 ;
 ; Note that for *all* of these cases countH:countL is
 ; modified. We never write these values back onto
 ; that stack. This means the caller of the function
 ; can assume call-by-value semantics for the argument
 ; passed into the function.
 ;
to_decimal_next:
 	clr multiple

to_decimal_10000:
 	cpi pos, 0
 	brne to_decimal_1000
 	ldi factorL, low(10000)
 	ldi factorH, high(10000)
 	rjmp to_decimal_loop

to_decimal_1000:
 	cpi pos, 1
 	brne to_decimal_100
 	ldi factorL, low(1000)
 	ldi factorH, high(1000)
 	rjmp to_decimal_loop

to_decimal_100:
 	cpi pos, 2
 	brne to_decimal_10
 	ldi factorL, low(100)
 	ldi factorH, high(100)
 	rjmp to_decimal_loop
to_decimal_10:
 	cpi pos, 3
 	brne to_decimal_1
 	ldi factorL, low(10)
 	ldi factorH, high(10)
 	rjmp to_decimal_loop

to_decimal_1:
 	mov multiple, countL
 	rjmp to_decimal_write

to_decimal_loop:
 	inc multiple
 	sub countL, factorL
 	sbc countH, factorH
 	brpl to_decimal_loop
 	dec multiple
 	add countL, factorL
 	adc countH, factorH

to_decimal_write:
 	ldd ZH, Y+PARAM_OFFSET+1
 	ldd ZL, Y+PARAM_OFFSET+0
 	add ZL, pos
 	adc ZH, zero
 	add multiple, ascii_zero
 	st Z, multiple
 	inc pos
 	cpi pos, 5
 	breq to_decimal_exit
 	rjmp to_decimal_next

to_decimal_exit:
 	pop ZL
 	pop ZH
 	pop YL
 	pop YH
 	pop ascii_zero
 	pop zero
 	pop pos
 	pop multiple
 	pop factorL
 	pop factorH
 	pop countL
 	pop countH
 	.undef countL
 	.undef countH
 	.undef factorL
 	.undef factorH
 	.undef multiple
 	.undef pos
 	.undef zero
 	.undef ascii_zero
 	ret
; ------------------------------------------------------
;          End of code provided by Dr. Zastre
; ------------------------------------------------------

; ***************************************************
; **** END OF FIRST "STUDENT CODE" SECTION ********** 
; ***************************************************


; ################################################
; #### BEGINNING OF "TOUCH CAREFULLY" SECTION ####
; ################################################

; The purpose of these locations in data memory are
; explained in the assignment description.
;

.dseg
CHAR_ONE: .byte 1
CHAR_TWO: .byte 1
CHAR_THREE: .byte 1
CHAR_FOUR: .byte 1

PULSE: .byte 1
COUNTER: .byte 2
DISPLAY_TEXT: .byte 16
BUTTON_CURRENT: .byte 1
BUTTON_PREVIOUS: .byte 1
BUTTON_COUNT: .byte 2
BUTTON_COUNT_2: .byte 1
BUTTON_LENGTH: .byte 1
DOTDASH_PATTERN: .byte MAX_PATTERN_LENGTH

; ##########################################
; #### END OF "TOUCH CAREFULLY" SECTION ####
; ##########################################
